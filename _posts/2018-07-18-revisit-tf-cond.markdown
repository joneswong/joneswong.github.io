---
layout: post
title:  "Revisit TF cond"
date:   2018-07-18 22:24:02 -0700
categories: daily TF
---
We usually use `tf.cond` op to express a conditional branching. However, its semantics are not that intuitive. Let's see some toy examples:

~~~
def study_cond():
    a = tf.get_variable(name='a', shape=(), initializer=tf.constant_initializer(.0))
    op0 = tf.assign_add(a, 1.0)
    op1 = tf.assign_add(a, -1.0)
    p = tf.placeholder(dtype=tf.bool, shape=(), name='p')
    where_op = tf.where(p, op0, op1)
    b = tf.placeholder(shape=(), dtype=tf.float32)
    cond_op0 = tf.cond(p, lambda: op0, lambda: op1)
    cond_op1 = tf.cond(p, lambda: tf.assign_add(a, b), lambda: tf.assign_add(a, -1.0))

    with tf.Session() as sess:
        sess.run(tf.global_variables_initializer())
        for i in range(1024):
            sess.run(where_op, feed_dict={p: False})
        print(a.eval(sess))# a is .0
        for i in range(1024):
            sess.run(cond_op0, feed_dict={p: False})
        print(a.eval(sess))# a is .0
        for i in range(1024):
            sess.run(cond_op1, feed_dict={p: False, b: 1.0})
        print(a.eval(sess))# a is -1024
~~~
{: .language-python}

As you can see, unless the assignment ops are defined inside the callables, they will be executed no matter in which branch they are called.
If you are supriced, official document of TF explains this as the nature of Dataflow programming model and your expectation is a so-called lazier semantics.
This [thread](https://stackoverflow.com/questions/37063952/confused-by-the-behavior-of-tf-cond) gives a vivid explanation:

> Because execution in a TensorFlow graph flows forward through the graph, all operations that you refer to in either branch must execute before the conditional is evaluated.

Note that in our example, if we run the `cond_op1` op without feeding a value for placeholder `b`, the program will crash.

