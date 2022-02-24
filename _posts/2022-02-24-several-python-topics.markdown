---
layout: post
title:  "Python stuffs"
date:   2022-02-24 20:24:02 -0700
categories: daily Python
---

## Creation of object and SINGLETON
Conceptually, we use `__new__` when you need to control the creation of a new instance and use `__init__` when you need to control initialization of a new instance.
~~~
# Python program to
# demonstrate __new__

# don't forget the object specified as base
class A(object):
    def __new__(cls):
        print("Creating instance")
        print(cls)
        result = super(A, cls).__new__(cls)
        print(result)
        return result

    def __init__(self):
        print("Init is called")

obj = A()
print(obj)
~~~
{: .language-python}

```
Creating instance
<class '__main__.A'>
<__main__.A object at 0x7f67cadd2748>
Init is called
<__main__.A object at 0x7f67cadd2748>
```

~~~
class Logger(object):
    def __new__(cls, *args, **kwargs):
        if not hasattr(cls, '_logger'):
            cls._logger = super(Logger, cls
                    ).__new__(cls, *args, **kwargs)
        return cls._logger
~~~
{: .language-python}

## 
