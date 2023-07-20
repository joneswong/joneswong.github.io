---
layout: post
title:  "A Note on Chap 2.4 2.5 of 'Group Equivariant Deep Learning'"
date:   2023-07-18 20:24:02 -0700
categories: daily
---

## Summary of Core-ideas

A neural network layer is essentially an operator that transforms an input signal into another signal (might be over another domain). 
When an operator is said to be equivariant to a group $$G$$, any group element acting on the input signal leads to an predictable change in the output signal.

## Concepts

*What is equivariance?*

Given

- An operator $$\Phi: \mathcal{X}\rightarrow\mathcal{Y}$$
- a group $$G$$ has representations $$\rho^{\mathcal{X}}$$ and $$\rho^{\mathcal{Y}}$$ on $$\mathcal{X}$$ and $$\mathcal{Y}$$, respectively

$$\Phi$$ is equivariant to $$G$$ if: $$\forall g\in G, \rho^{\mathcal{Y}}(g)\circ\Phi=\Phi\circ\rho^{\mathcal{X}}(g)$$

*What is a Haar measure on a group $$G$$?*

- (left Haar measure) $$\forall \tilde{g}\in G, \text{d}(\tilde{g}g) = \text{d}g$$
- (right Haar measure) $$\forall \tilde{g}\in G, \text{d}g\tilde{g} = \text{d}g$$

*What is unimodular group?*

When the left and right Haar meansures coincide, the underlying group $$G$$ is said to be a unimodular group.

## Examples and Interpretations

*How to prove that (eg2.10) the correlation operator is equivariant to the translation group?*

The correlation operator is defined as $$(k \star f)(x) = (\mathcal{T}_{x}k, f)_{\mathbb{L}_{2}(\Re^d)}$$.

As the translation group has left-regular representation for the space $$\mathbb{L}_{2}(\Re^d)$$, we have $$\forall g\in\Re^d, (\rho(g)\circ f)(\tilde{x}) = f(g^{-1}\otimes \tilde{x}) = f(\tilde{x} - g)$$. Thus, the above equation becomes $$\int_{\tilde{x}} k(\tilde{x}-x) f(\tilde{x}-g)\text{d}\tilde{x}$$. Similarly, $$(\rho(g)\circ (k\star f))(x) = (k\star f)(x-g)$$. Thus, the above equation becomes $$\int_{\tilde{x}} k(\tilde{x}-(x-g)) f(\tilde{x})\text{d}\tilde{x}$$.

Let $$x' = \tilde{x}-g$$ and apply substitution. We get $$\int_{\tilde{x}} k(\tilde{x}-x) f(\tilde{x}-g)\text{d}\tilde{x} = \int_{x'} k(x' - x + g) f(x')\text{d}x'$$, which is equivalent to apply the group representation on the output signal.

*Any useful result from Haar measure?*

(e.g., Lemma 2.3) Given $$k, f \in \mathbb{L}_{2}(G)$$, $$\mathcal{L}_g$$ the left-regular representation of $$g\in G$$ on $$\mathbb{L}_{2}(G)$$, and Haar measure $$\text{d}g$$. Then we have $$(\mathcal{L}_{g}k, f)_{\mathbb{L}_{2}(G)} = (k, \mathcal{L}_{g^{-1}}f)_{\mathbb{L}_{2}(G)}$$

As the textbook shows, $$LHS = \int_{G} [\mathcal{L}_{g}k](\tilde{g}) f(\tilde{g})\text{d}\tilde{g} = \int_{G} k(g^{-1}\tilde{g}) f(\tilde{g})\text{d}\tilde{g} $$.

We can make substitution: $$\tilde{g}=gg'$$. Then the equation becomes $$\int_{gG} k(g') f(gg')\text{d}gg' $$, which, due to the measure is a Haar measure, can be written as $$\int_{G} k(g') f(gg')\text{d}g' = \int_{G} k(g') [\mathcal{L}_{g^{-1}}f](g')\text{d}g' = RHS$$.

*How to interpret the notations of Haar measure's invariance?*

Suppose, in the above example, $$G$$ is translations on $$\Re$$, $$g=-1$$, and the integral interval is $$[a, b]$$. Then the variable substitution $$\tilde{g}=gg' = 1 + g'$$ should let $$gg'=1+g'$$ change along with $$[a, b]$$, which, in turn, lets $$g'$$ change along with $$gG=[a, b] -1 = [a-1, b-1]$$. Thus, Haar measure means a "uniform measure", that is to say, different intervals have the same outer measure (length), i.e., $$\|[a, b]\|=\|[a-1, b-1]\|$$.

*Any concrete example of the above equation?*

(e.g., EX 2.6) Because $$G=SE(d)$$, we have $$\tilde{x}=g\odot x' = R_{g}x' + x_{g}$$, and $$\text{d}\tilde{x}=\text{d}(R_{g} x' + x_g) = \text{d}x'$$ is a Haar measure.