---
layout: post
title:  "Several PyTorch-related mistakes"
date:   2022-03-15 20:24:02 -0700
categories: daily Python PyTorch
---

## When is deepcopy and when is shallow copy?
What do you expect?
~~~
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

for v in m.parameters():
    v[0] = 123.0
print(m.state_dict())
print(a)

m.load_state_dict(a)
print(m.state_dict())
print(a)

for k in a:
    a[k][1] = 456.0
print(m.state_dict())
print(a)
~~~
{: .language-python}

The output is as follow:
```
OrderedDict([('fc.weight', tensor([[-0.2181,  0.4883],
        [-0.3931,  0.3060]])), ('fc.bias', tensor([ 0.2015, -0.1971]))])
OrderedDict([('fc.weight', tensor([[-0.2181,  0.4883],
        [-0.3931,  0.3060]])), ('fc.bias', tensor([ 0.2015, -0.1971]))])
OrderedDict([('fc.weight', tensor([[123.0000, 123.0000],
        [ -0.3931,   0.3060]])), ('fc.bias', tensor([123.0000,  -0.1971]))])
OrderedDict([('fc.weight', tensor([[123.0000, 123.0000],
        [ -0.3931,   0.3060]])), ('fc.bias', tensor([123.0000,  -0.1971]))])
OrderedDict([('fc.weight', tensor([[123.0000, 123.0000],
        [ -0.3931,   0.3060]])), ('fc.bias', tensor([123.0000,  -0.1971]))])
OrderedDict([('fc.weight', tensor([[123.0000, 123.0000],
        [ -0.3931,   0.3060]])), ('fc.bias', tensor([123.0000,  -0.1971]))])
OrderedDict([('fc.weight', tensor([[123., 123.],
        [456., 456.]])), ('fc.bias', tensor([123., 456.]))])
OrderedDict([('fc.weight', tensor([[123., 123.],
        [456., 456.]])), ('fc.bias', tensor([123., 456.]))])
```

Roughly speaking, `state_dict()` method returns a deepcopy of the module's tensors, while `load_state_dict()` makes the module refers to the given tensors in the dict (i.e., shallow copy the tensors).
