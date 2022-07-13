---
layout: post
title:  "Several PyTorch-related mistakes"
date:   2022-03-15 20:24:02 -0700
categories: daily Python PyTorch
---

## When is deep copy and when is shallow copy?
What do you expect?
~~~
import copy

import torch


class LogisticRegression(torch.nn.Module):
    def __init__(self, in_channels, class_num):
        super(LogisticRegression, self).__init__()
        self.fc = torch.nn.Linear(in_channels, class_num)

    def forward(self, x):
        return self.fc(x)

m = LogisticRegression(2, 2)
a = m.state_dict()
print(m.state_dict())
print(a)
print("\n\n")


for v in m.parameters():
    #v[0] = 123.0
    #v[0].data = torch.Tensor([123.0])
    v.data -= 0.5 * v.data
print("state_dict() is shallow copy:")
print(m.state_dict())
print(a)
print("\n\n")

# whether to comment this line matters!!!!!!
a = copy.deepcopy(a)
m.load_state_dict(a)

for k in a:
    #a[k][1] = 456.0
    #a[k][1].data = torch.Tensor([456.0])
    a[k].data -= 0.5 * a[k].data
print("load_state_dict() is ? copy:")
print(m.state_dict())
print(a)
print("\n\n")


for param in m.parameters():
    #param[0] = 999.0
    #param[0].data = torch.Tensor([999.0])
    param.data -= 0.5 * param.data
print("load_state_dict() is ? copy:")
print(m.state_dict())
print(a)
~~~
{: .language-python}

The output is as follow:
```
OrderedDict([('fc.weight', tensor([[-0.0472, -0.4500],
        [-0.3051, -0.3033]])), ('fc.bias', tensor([-0.1162, -0.4246]))])
OrderedDict([('fc.weight', tensor([[-0.0472, -0.4500],
        [-0.3051, -0.3033]])), ('fc.bias', tensor([-0.1162, -0.4246]))])



state_dict() is shallow copy:
OrderedDict([('fc.weight', tensor([[-0.0236, -0.2250],
        [-0.1525, -0.1517]])), ('fc.bias', tensor([-0.0581, -0.2123]))])
OrderedDict([('fc.weight', tensor([[-0.0236, -0.2250],
        [-0.1525, -0.1517]])), ('fc.bias', tensor([-0.0581, -0.2123]))])



load_state_dict() is ? copy:
OrderedDict([('fc.weight', tensor([[-0.0236, -0.2250],
        [-0.1525, -0.1517]])), ('fc.bias', tensor([-0.0581, -0.2123]))])
OrderedDict([('fc.weight', tensor([[-0.0118, -0.1125],
        [-0.0763, -0.0758]])), ('fc.bias', tensor([-0.0291, -0.1062]))])



load_state_dict() is ? copy:
OrderedDict([('fc.weight', tensor([[-0.0118, -0.1125],
        [-0.0763, -0.0758]])), ('fc.bias', tensor([-0.0291, -0.1062]))])
OrderedDict([('fc.weight', tensor([[-0.0118, -0.1125],
        [-0.0763, -0.0758]])), ('fc.bias', tensor([-0.0291, -0.1062]))])
```

or as follow with `a = copy.deepcopy(a)` commented out:
```
OrderedDict([('fc.weight', tensor([[ 0.1331,  0.0882],
        [ 0.2721, -0.1324]])), ('fc.bias', tensor([-0.5216,  0.4661]))])
OrderedDict([('fc.weight', tensor([[ 0.1331,  0.0882],
        [ 0.2721, -0.1324]])), ('fc.bias', tensor([-0.5216,  0.4661]))])



state_dict() is shallow copy:
OrderedDict([('fc.weight', tensor([[ 0.0665,  0.0441],
        [ 0.1360, -0.0662]])), ('fc.bias', tensor([-0.2608,  0.2330]))])
OrderedDict([('fc.weight', tensor([[ 0.0665,  0.0441],
        [ 0.1360, -0.0662]])), ('fc.bias', tensor([-0.2608,  0.2330]))])



load_state_dict() is ? copy:
OrderedDict([('fc.weight', tensor([[ 0.0333,  0.0220],
        [ 0.0680, -0.0331]])), ('fc.bias', tensor([-0.1304,  0.1165]))])
OrderedDict([('fc.weight', tensor([[ 0.0333,  0.0220],
        [ 0.0680, -0.0331]])), ('fc.bias', tensor([-0.1304,  0.1165]))])



load_state_dict() is ? copy:
OrderedDict([('fc.weight', tensor([[ 0.0166,  0.0110],
        [ 0.0340, -0.0165]])), ('fc.bias', tensor([-0.0652,  0.0583]))])
OrderedDict([('fc.weight', tensor([[ 0.0166,  0.0110],
        [ 0.0340, -0.0165]])), ('fc.bias', tensor([-0.0652,  0.0583]))])
```

What makes the difference? The implementation of `load_state_dict()` can be roughly understood as copy each corresponding tensor:
```
param_x.copy_(tensor_x)
```
which is different from the ordinary shallow copy
```python
param_x = tensor_x
```
that leads to `id(param_x) == id(tensor_x)`; and is also different from  the so-called deep copy
```python
param_x = copy.deepcopy(tensor_x)
```
for general Python object, e.g., list. Specifically, deep copy (a list) means allocating another memory space to hold the same values as the right-value (i.e., the source list).
In contrast, Tensor object has an attribute that records the memory address of stored values, and `copy_()` just passes the addresses of left-value and right-value to CUDA API to copy the stored values.
Thus, without the command `a = copy.deepcopy(a)`, the tensors in `a` have the same addresses as the parameters in `m`, since `a` is acquired by `state_dict()`. By `_copy()`, parameters of `m` and tensors in `a` point to the same memory space, making any changes in `a`'s entries observable from `m`'s entries.
