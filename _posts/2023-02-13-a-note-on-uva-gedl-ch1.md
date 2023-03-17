---
layout: post
title:  "A Note on Chap1 of 'Group Equivariant Deep Learning'"
date:   2023-02-12 20:24:02 -0700
categories: daily
---

## Summary of Core-ideas

Convolution --- Cross-correlation --- Template matching, i.e., inner product at different "positions"

"at different positions" --- "between kernel transformed by different group elements and the input signal"

CNN (positions mean different translation) --- R-CNN (positions mean different roto-translation)

## Concepts

*What's convolution?*

For $$k,f\in\mathbb{L}_{2}(X)$$,

$$
(k\ast f)(x)=\int_{X} k(x-\tilde{x})f(\tilde{x})d\tilde{x}
$$

Two functions (a kernel and a signal) are transformed into another function (signal).

*What's cross-correlation?*

For $$k,f\in\mathbb{L}_{2}(X)$$,

$$
(k\ast f)(x)=\int_{X} k(\tilde{x}-x)f(\tilde{x})d\tilde{x}
$$

For multi-channel signals, we just sum up the results of all channels in calculating convolution and cross-correlation.

*What is translation and roto-translation operators?*

Let $$k\in\mathbb{L}_{2}(\Re^d)$$, then, for each $$x\in\Re^d$$,

$$
[\mathcal{T}_{x}k](\tilde{x})=k(\tilde{x}-x)
$$

We say $$\mathcal{T}$$ parameterized by $$x\in\Re^d$$ is a translation operator.

Let $$g=(x,\theta)$$ where $$x$$ is a translation vector, and $$\theta$$ is the rotation angle,

$$
[\mathcal{L}_{g}k](\tilde{x})=k(R_{\theta}^{-1}(\tilde{x}-x))
$$

We say $$\mathcal{L}$$ parameterized by $$g$$ is a roto-translation operator. Particularly, there $$R_{\theta}$$ is a matrix for executing the rotation action and its inverse means rotating with the opposite angle (see later chapters for more details).

Other necessary definitions in this chapter include the *inner product and norm* of $$\mathbb{L}_{2}(X)$$.

## Interpretations

*Is convolution and cross-correlation the same stuff?*

Yes, they are related via kernel reflection, namely, letting $$k(x)=k'(-x),\forall x\in X$$, then $$k\ast f=k'\star f$$.

*What is cross-correlation doing?*

It makes "template matching".
  - Intuitively, inner product is a similarity measure;
  - Cross-correlation measures the inner product between a kernel and a signal, where the kernel is transformed by an operator: $$(k\star f)(x)=(\mathcal{T}_{x}k,f)_{\mathbb{L}_{2}(\Re^d)}$$.

Recall that convolution is the "same" as cross-correlation, so CNN is making template matching.

*What's the source of CNN's power?*

- If the desired pattern is translated in $$\Re^2$$, it will still be recognized by the translated kernel $$k$$.
- Weight-sharing: no matter where the pattern is located in $$\Re^2$$, the kernel $$k$$ applied to recognize it is parameterized by the same suite of parameters.

*What's the motivation of G-CNN?*

It is still making template matching with inner product but the kernel is roto-translation lifted (not just translated), that is to say, making *group correlation* (here *roto-translation lifting correlation*):

$$
(k\star_{\text{SE(2)}}f)(x,\theta)=(\mathcal{L}_{g}k,f)_{\mathbb{L}_{2}(\Re^2)}=\int_{\Re^2}k(R_{\theta}^{-1}(\tilde{x}-x))f(\tilde{x})d\tilde{x}
$$

where the two functions (kernel $$k$$ and signal $$f$$) are transformed into a higher dimensional function (signal or say feature map) with input $$(x,\theta)$$. Here "SE" is short for special euclidean motion group.

In this way, those two points regarded as CNN's power are further generalized, namely, from translation to roto-translation.