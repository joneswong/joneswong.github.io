---
layout: post
title:  "Express Combinatorial Operation"
date:   2018-08-23 18:24:02 -0700
categories: daily TF
---
Suppose we have a dense layer consisting of *n* units each applies a sigmoidal function to its input activation. Lets denote the output as a *n*-dimensional vector **a**
Considering that the output takes value from *[0, 1]*, **(1-a)** also takes value from *[0, 1]*.
We want to build a *2^n*-dimensional vector each component is the products of either **a_i** or **(1-a)_i** for **i**=1, ..., *n-1*.

It seems that there is no such a layer in TF high-level APIs. I implemented a toy as follow:

~~~
def combinatorial_op():
    n = 3
    a = tf.constant([0.1, 0.2, 0.3], dtype=tf.float32)
    a = tf.expand_dims(a, -1)
    b = 1.0 - a
    a = tf.concat([a, b], axis=1)

    i = tf.range(n)
    # (1, n) repeated (2**n, 1) times respectively
    # resulting in shape (2**n, n)
    component_idx = tf.tile(tf.expand_dims(i, 0), [2**n, 1])

    # leverage bit operations, e.g., 6=100
    # bit-and & 2, 4, 8 resulting in 0, 0, 1 respectively
    i = tf.range(2**n)
    bits = list()
    for idx in range(n):
        mv = tf.bitwise.bitwise_and(i, 2**idx)
        bits.append(tf.expand_dims(tf.cast(tf.equal(mv, 0), dtype=tf.int32), axis=-1))
    component_choice = tf.concat(bits, axis=1)

    # e.g., idx = tf.constant([[[0,0],[1,1],[2,0]], [[0,0],[1,1],[2,0]]], dtype=tf.int32)
    # select shape (2, 3) tensor of elements, each specified by the innerest [i,j] indices
    index = tf.concat([tf.expand_dims(component_idx, -1), tf.expand_dims(component_choice, -1)], axis=2)
    x = tf.gather_nd(a, index)

    sess = tf.Session()
    sess.run(tf.global_variables_initializer())
    print(a.eval(session=sess))
    print(component_idx.eval(session=sess))
    print(component_choice.eval(session=sess))
    print(sess.run(index))
    print(sess.run(x))
~~~
{: .language-python}
