---
layout: post
title:  "A Note on Chap 2.3 of 'Group Equivariant Deep Learning'"
date:   2023-05-25 20:24:02 -0700
categories: daily
---

## Summary of Core-ideas

It is often helpful to study a homogeneous space of a group.

We can start with a group and a subgroup to build the corresponding quotient space, which is homogeneous to the original group.
Conversely, we can start with a space and use the stabilizer of some element of this space to build the subgroup that will lead to a quotient space, identifying (i.e., there exists an isomorphism between the two spaces) this space.



## Concepts

*When will we say a group action is transitive?*

A group action $\odot:G\times\mathcal{X}\rightarrow\mathcal{X}$ is transitive if $\forall x, \tilde{x}\in\mathcal{X},\exists g\in G$, s.t. $\tilde{x}=g\odot x$.

*When will a space be called homogeneous space for the group acting on it?*

- the group is a Lie group
- the group action is transitive

*What is a semidirect-product?*

Basically, it is a group.

It is constructed from two groups $N$ and $H$ with a group action $\odot: H\times N\rightarrow N$. We denote the semidirect-product by $N\rtimes H$.

The group product and inverse is defined as:

$(n,h)\cdot(\tilde{n},\tilde{h}) = (n\cdot (h\odot\tilde{n}), h\cdot\tilde{h})$

$(n,h)^{-1}=(h^{-1}\odot n^{-1}, h^{-1})$

Note that, for simplicity, we use the same symbol $\cdot$ for these two groups' respective group product.

*What is a coset?*

For a group $G$ and a subgroup of it $H$, $gH=\{g\cdot h | h\in H\}$ for any group element $g\in G$ is a coset.

*What is quotient space?*

Given a group $G$ and its subgroup $H$, the quotient space $G/H$ denotes the collection of distinct cosets $\{gH | g\in G\}$.

Thus, elements in $G/H$ are equivalence classes, where any $g\neq\tilde{g}$ are in the same class if $\exists h\in H$ s.t., $g=\tilde{g}\cdot h$.

*What is a stabilizer?*

Suppose there is a group action $\odot: G\times \mathcal{X}\rightarrow \mathcal{X}$, then the stabilizer (subgroup) of $G$ w.r.t. $x_0 \in \mathcal{X}$ is defined as:
$Stab_{G}(x_0)=\{g|g\odot x_0 = x_0 \}$

*What is affine group?*

Groups constructed by $\Re^{d}\rtimes H$ for a certain $H\subseteq\text{GL}(\Re^{d})$, where $\text{GL}(\Re^d)$ denotes the general linear transformations acting on $\Re^d$. $\text{GL}(\Re^d)$ consists of invertible matrices acting on $\Re^d$, and $H$ is commonly a subgroup of $\text{GL}(\Re^d)$.


## Examples and Interpretations

*Any example of semi-product?*

Consider $SE(d)=(\Re^{d},+)\rtimes SO(d)$. Then the group product and inverse element can be expressed as:

$(x, R)\cdot(\tilde{x},\tilde{R})=(x+R\tilde{x}, R\tilde{R})$

$(x, R)^{-1}=(-R^{-1}x, R^{-1})$

where $x\in\Re^{d}$ and $R$ is the corresponding rotation matrix of a specific angle.

*Are cosets always groups?*

No, a coset may not be a group.

*Any example of quotient space?*

Suppose $G=\{0, 1, \ldots, 7\}$, and the group product is defined as $g\cdot\tilde{g}=(g+\tilde{g})\text{ mod }8$.

Suppose $H=\{0, 4\}$, then the quotient space $G / H$ consists of $\{0, 4\}, \{1, 5\}, \{2, 6\}, \{3, 7\}$.

*What's the relationship between $G$ and its quotient space $G/H$?*

Define the group action $i \odot gH$ to be $\{i\cdot j | j\in gH\}$.

Then, for any $gH$ and $\tilde{g}H$, $\exists i\in G$ s.t., $i\odot gH = \tilde{g}H$, because, to let $i\cdot g\cdot h = \tilde{g}\cdot h$, we could just let $i=\tilde{g} \cdot g^{-1}$.

Thus, $G/H$ is a homogeneous space of $G$.

*How to interpret affine space from the perspective of quotient space?*

Taking SE(2)=$\Re^2 \rtimes S^1$ as an example. The group product, according to the definition of semidirect product, is $g_1 \cdot g_2 =(x_1, \theta_1) \cdot (x_2, \theta_2) = (x_1 + R_{\theta_1}x_2, \theta_1 + \theta_2)$. Obviously, $\{(0, \theta)|\theta \in S^1 \}$ is a subgroup, where $(0, 0)$ is the group product identity, and closure is preserved. Then each equivalence class is in the form of $\{ (x, \theta)|\theta \in S^1 \}$.

*Why can the representation of a semidirect product be decomposed into the function composition of their respective representations?*