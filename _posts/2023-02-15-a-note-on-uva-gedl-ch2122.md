---
layout: post
title:  "A Note on Chap 2.1 2.2 of 'Group Equivariant Deep Learning'"
date:   2023-02-15 20:24:02 -0700
categories: daily
---

## Concepts

*What's vector space?*

A vector space on a field $$F$$ is a set $$V$$ with two binary operators that satisfy:

- For vector addition:
  - Associativity: $$\forall u,v,w\in V, (u+v)+w=u+(v+w)$$
  - Commutavility: $$\forall u,v\in V, u+v=v+u$$
  - Identity: $$\exists 0\in V$$ so that $$\forall u\in V, u+0=u$$
  - Inverse: $$\forall u\in V, \exists -v \in V$$ so that $$v + (-v) = 0$$
- For scalar multiplication:
  - Compatibility between scalar multiplication and field multiplication: a(bv)=(ab)v
  - Identity: $$1v=v$$
  - Distributivity (w.r.t. vector addition): $$a(u+v)=au+av$$
  - Distributivity (w.r.t. field addition): $$(a+b)v=av+bv$$

*What's group?*

$$(G, \cdot)$$, where $$G$$ is a set, and $$\cdot$$ is the group product (i.e., a binary operator), so that

  - Closure: $$\forall g,h\in G, g\cdot h \in G$$
  - Identity: $$\exists e\in G$$, s.t., $$e\cdot g=g,\forall g\in G$$
  - Inverse: $$\forall g\in G,\exists g^{-1}$$, s.t., $$g^{-1}\cdot g = g\cdot g^{-1} = e$$
  - Associativity: $$\forall g,h,i\in G, (g\cdot h)\cdot i = g\cdot(h\cdot i)$$

*What's Lie group? (not that formal)*

It is continuous group that is also a differentiable manifold, where continuous group means $$G$$ is infinite, and its group operator is continuous.

*What's subgroup?*

When we say $$(H,\cdot)$$ is a subgroup of $$(G,\cdot)$$, $$H\sube G$$ should preserve the closure property of $$\cdot$$.

*What's group homomorphism?*

Given two gruops $$(G,\ast)$$ and $$(H,\cdot)$$, a group homomorphism from the former to the latter is a function $$f$$ that satisfies:

$$
\forall g,\tilde{g}\in G,(h=f(g)\text{ and }\tilde{h}=f(\tilde{g}))\rightarrow h\cdot\tilde{h} = f(g\ast\tilde{g})
$$

*What's group action?*

Given a group $$(G,\cdot)$$, a group action on a space $$X$$ is a binary operator $$\odot$$ from $$G\times X$$ to $$X$$, so that:

$$
\forall g,\tilde{g}\in G,x\in X, g\odot(\tilde{g}\odot x) = (g\cdot\tilde{g})\odot x
$$

*What's representation?*

Given a group $$(G,\cdot)$$, a representation parameterized by $$g\in G$$ is a *linear and invertible* function $$\rho(g)$$ mapping from a *vector space* $$V$$ to itself, so that:

$$
\forall g,\tilde{g}\in G,v\in V, \rho(g)\rho(\tilde{g})v = \rho(g\cdot\tilde{g})v
$$

*What's matrix representation?*

When the dimension of $$V$$, denoted by $$d$$ is finite, it is equivalent to consider $$\Re^{d}$$. Then any linear transformation can be expressed by $$d\times d$$ matrix. So a matrix representation $$D(g)$$ is a $$d\times d$$ matrix that respects the properties a representation should have and acts on $$v\in \Re^{d}$$ by matrix-vector multiplication. It is often said $$D(g)\in\text{GL}(d,\Re)$$, where "GL" is short for general linear group.

*What's left-regular representation?*

Suppose the vector space $$V$$ consists of functions in $$\mathbb{L}_2(X)$$, and a group $$G$$ has group action on $$X$$ denoted by $$\odot$$. Then a left-regular representation $$\mathcal{L}_{g}$$ of $$G$$ acting on $$\mathbb{L}_2(X)$$ is representation that satisfies:

$$
\forall g\in G,\forall f\in V, \forall x\in X,[\mathcal{L}_{g}f](x)=f(g^{-1}\odot x)
$$

## Examples and Interpretations

*Why do we need group product?*

Recall that, in chap1, we interpret cross-correlation as the inner product between the kernel (transformed by a group) and the signal. Taking input at different positions by the cross-correlation means transforming the kernel by different group members. Thus, group product can be interpreted as the composition of two transformations, and the inverse of $$g$$ means the transformation cancelling out $$g$$'s effect.

Examples include translation group $$G=(\Re^d,+)$$ and rotation group $$\text{SO}(2)=([0,2\pi),+_{\text{mod }2\pi})$$, which is often parameterized as:

$$
R_{\theta}=\left[\begin{array}{cc}
cos\theta & -sin\theta \\
sin\theta & cos\theta \\
\end{array}\right],
$$
and group product corresponds to matrix multiplication.

*So why do we need group structure?*

Obviously, the translation group with its group product as "+" and vanilla scalar-vector multiplication forms a vector space. As for SO(2), with its group product as "+" and scalar multiplication (and then mod $$2\pi$$), it also forms a vector space.
At least, their group product is our very familiar arithmatic operations (matrix multiplication between two rotation matrices is commutative).
However, for SE($$d$$), its group product is:

$$
(x,\theta) \cdot (\tilde{x},\tilde{\theta})=(x+R_{\theta}\tilde{x},R_{\theta}R_{\tilde{\theta}}),
$$

which is not commutative, and thus SE($$d$$) cannot be a vector space by regarding group product as vector addition and supplementing the definition of scalar-vector multiplication.

*How to interpret "group action is a group homomorphism"?*

It is a mapping from $$G$$ to $$\{f_g | g\in G\}$$, and the group product of $$\{f_g | g\in G\}$$ is function composition, namely, $$\forall f_g, f_h, f_g \cdot f_h = f_g(f_{h}())$$. Then we need to prove $$f_{g\cdot h}=f_g \cdot f_h$$ (note that $$\cdot$$ at the LHS and RHS are that of respective group), which is correct by the definition of group action.

*What's the essential property that makes a group representation left-regular?*