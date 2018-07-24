---
layout: post
title:  "TF variable sharing"
date:   2018-07-24 23:24:02 -0700
categories: daily TF
---
`tf.make_template()` has been widely used in many open source projects, e.g., Sonnet, TensorForce, etc., for variable sharing. It's signature is as follow:

~~~
make_template(
    name_,
    func_,
    create_scope_now_=False,
    unique_name_=None,
    custom_getter_=None,
    **kwargs
)
~~~
{: .language-python}

It wraps an arbitrary function `func_`, so that, in the wrapped function, any TF variable defined by `tf.get_variable()` will be created only once (at the construction of this template if `create_scope_now` is `True`, or at the first call of this template). In the following calls of this template, these TF variables will be automatically reused. Let's see some toy examples:

~~~
def study_template():
    def tf_fn(x):
        w = tf.get_variable(name='w', shape=(), dtype=tf.float32, initializer=tf.random_uniform_initializer(minval=.0, maxval=1.0))
        return w*x+.0
    fn = tf.make_template(name_='fn', func_=tf_fn)
    x = tf.placeholder(name='x', shape=(), dtype=tf.float32)
    with tf.variable_scope('abc') as sess:
        y1 = fn(x)
    with tf.variable_scope('def') as sess:
        y2 = fn(x)

    # abc/fn/w:0
    for var in tf.get_collection(tf.GraphKeys.GLOBAL_VARIABLES):
        print(var.name)

    with tf.Session() as sess:
        sess.run(tf.global_variables_initializer())
        y1_val, y2_val = sess.run([y1, y2], feed_dict={x:0.5})

        # identical
        print(y1_val)
        print(y2_val)
~~~
{: .language-python}

What's the mechanism? Let's first revisit the most commonly used variable sharing mechanism: `tf.variable_scope()` which is usually used as *with block*. In Python, *with block* such as `with A() as a:` calls the `__enter__()` method of class `A` and `a` is its returned value. When leave this block, the `__exit()__` method is called. The signature of `tf.variable_scope`'s constructor is as follow:

~~~
__init__(
    name_or_scope,
    default_name=None,
    values=None,
    initializer=None,
    regularizer=None,
    caching_device=None,
    partitioner=None,
    custom_getter=None,
    reuse=None,
    dtype=None,
    use_resource=None,
    constraint=None,
    auxiliary_name_scope=True
)
~~~
{: .language-python}

In addition to the required argument `name_or_scope`, the most frequently used argument is `reuse` which takes value from:

- `True` indicates reuse mode (do NOT create variable) in this scope and subscopes.
- `None`/`False` means that this scope will inherit the reuse behavior of its parent scope
- `tf.AUTO_REUSE` indicates variables will be created if they do NOT exist, or just returned otherwise.

Roughly speaking, in the `enter_scope_uncached()` method (the `__enter()__` method just call it), the returned value is made by:

~~~
cope = _pure_variable_scope(
    self._name_or_scope,
    reuse=self._reuse,
    initializer=self._initializer,
    regularizer=self._regularizer,
    caching_device=self._caching_device,
    partitioner=self._partitioner,
    custom_getter=self._custom_getter,
    old_name_scope=old_name_scope,
    dtype=self._dtype,
    use_resource=self._use_resource,
    constraint=self._constraint)
try:
    entered_pure_variable_scope = pure_variable_scope.__enter__()
except:
    pure_variable_scope.__exit__(*sys.exc_info())
    raise
self._cached_pure_variable_scope = pure_variable_scope
return entered_pure_variable_scope
~~~
{: .language-python}

where `self._name_or_scope` and `self._reuse` are still the arguments passed into the `variable_scope` class. So, let's see what does `_pure_variable_scope` class return in its `__enter__()` method.

First, in the constructor of `_pure_variable_scope` class, there is a line `self._var_scope_store = get_variable_scope_store()`. This method is as follow:

~~~
def get_variable_scope_store():
  """Returns the variable scope store for current thread."""
  scope_store = ops.get_collection(_VARSCOPESTORE_KEY)

  if not scope_store:
    scope_store = _VariableScopeStore()
    ops.add_to_collection(_VARSCOPESTORE_KEY, scope_store)
  else:
    scope_store = scope_store[0]

  return scope_store
~~~
 {: .language-python}

where `_VARSCOPESTORE_KEY` collection corresponds to the scopes (organized by so-called `scope_store`) defined in this graph. Thus, in the `__enter__()` method of `_pure_variable_scope` class, 

~~~
self._old = self._var_scope_store.current_scope
self._new_name = (
    self._old.name + "/" + self._name_or_scope if self._old.name
    else self._name_or_scope)
self._reuse = (self._reuse
              or self._old.reuse)  # Re-using is inherited by sub-scopes.
~~~
{: .language-python}

The parent scope's name is used as a prefix and the reuse behavior is inherited if `reuse` was specified as `None`. After the nested scenarios are handled, A `VariableScope` object was instantiated:

~~~
variable_scope_object = VariableScope(
    self._reuse,
    name=self._new_name,
    initializer=self._old.initializer,
    regularizer=self._old.regularizer,
    caching_device=self._old.caching_device,
    partitioner=self._old.partitioner,
    dtype=self._old.dtype,
    use_resource=self._old.use_resource,
    custom_getter=self._old.custom_getter,
    name_scope=name_scope,
    constraint=self._constraint)

self._var_scope_store.open_variable_scope(self._new_name)
self._var_scope_store.current_scope = variable_scope_object
return variable_scope_object
~~~
{: .language-python}

where the constructed variable scope is added to the variable scope store and set as the "current" one. Then, the `VariableScope` object is returned.

In the *with statement*, `sess` is an instance of `VariableScope` class. This class provide a `get_variable()` method where some arguments like `reuse`, if has NOT been specified, will take the value of corresponding class property:

~~~
if reuse is None:
    reuse = self._reuse
~~~
{: .language-python}

Actually, `tf.get_variable()` method is implemented as: `return get_variable_scope().get_variable(...)` where `get_variable_scope()` method is implemented as `return get_variable_scope_store().current_scope`. The story is that in the *with block*, any `tf.get_variable()` call is actually calling the `get_variable()` method of `sess`. This explain why the reuse behavior is controled by the scope and name is concatenated to the prefix (i.e., nested scope names).

When we leave the *with block*, the `__exit__()` method of `variable_scope` class calls the `__exit__()` method of `self._cached_pure_variable_scope`, which close current variable scope and re-set the "old" one (i.e., parent scope) as "current":

~~~
self._var_scope_store.close_variable_subscopes(self._new_name)
self._var_scope_store.current_scope = self._old
~~~
{: .language-python}

I summarized this post for version 1.9. According to this [post](https://blog.csdn.net/u012436149/article/details/73555017) which is based on an out-of-date version, the implementation has been slightly changed while the mechanism is the same.

Finally, for `make_template()` method, it just calls `make_template_internal()` method to return a `Template` object. In the constructor of `Template` class, it initializes `self._variables_created` as `False` and, suppose `create_scope_now` is given as `False`, initializes `self._variable_scope` as `None`. When we call a template to build graph, the behavior is as follow:

~~~
def __call__(self, *args, **kwargs):
    if self._variable_scope:
      # Only reuse variables if they were already created.
      with variable_scope.variable_scope(
          self._variable_scope, reuse=self._variables_created):
        return self._call_func(args, kwargs)
    else:
      # The scope was not created at construction time, so create it here.
      # Subsequent calls should reuse variables.
      with variable_scope.variable_scope(
          self._unique_name, self._name,
          custom_getter=self._custom_getter) as vs:
        self._variable_scope = vs
        return self._call_func(args, kwargs)
~~~
{: .language-python}

where, except for the first time, the `reuse` flag is always `True` as `self._variables_created` has been set as `True` in `_call_func()` method in the first call. As for the prefix, since `self._variable_scope` is assigned a `VariableScope` object after the first call of this tamplate, the first *if branch* will pass such an instance instead of a string to the `name_or_scope` argument of `variable_scope` class. In this case, it will NOT consider the parent scopes' names as prefix which ensures that we just see "abc/fn/w:0" and no "def/fn/w:0".


This is the whole story. TF docs are good, but, without source code, TF is still a black-box to me...
