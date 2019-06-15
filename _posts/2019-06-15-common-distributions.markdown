---
layout: post
title:  "Common Distributions"
date:   2019-06-15 13:24:02 -0700
categories: stat
---

Recently, I am enrolled in an open course about Bayesian Statistics which provides a note on common distributions. I quickly reviewed them and wrote this post for summarization.

# Discrete
## Uniform Distribution
We randomly pick $$1, 2, \ldots, N$$ each with a probability $$\frac{1}{N}$$.
It is easy to show the mean of this distribution is $$E[X]=\frac{\frac{1}{2}(1+N)N}{N}=\frac{N+1}{2}$$.
As for the variance (i.e., $$Var[X]=E[X^2]-E[X]^2$$), the first term needs to solve $$\sum_{x=1}^{N}x^2$$ which seems not that trivial.
Luckily, Khan provides a video showing how to solve such kind of number series with the core idea of Calculus.
For $$f_n=\sum_{x=1}^{N}x^2$$, $$f_{0}=0, f_{1}=1, f_{2}=5, f_{3}=14, f_{4}=30$$, their differences are $$f_{1}-f_{0}=1, f_{2}-f_{1}=4, f_{3}-f_{2}=9, f_{4}-f_{3}=16$$, then their differences between differences are $$3, 5, 7$$. Eventually, the differences are a constant of $$2$$.
Regarding the difference as a derivative, the closed-form solution of $$f_x$$ should be $$f_{x}=Ax^3+Bx^2+Cx+D$$ whose third-order derivative is constant (actually, here we know $$A=\frac{1}{3}$$ directly from the constant $$2$$). 
Solving an linear equation of three individual $$f_{x},x=1,2,3$$ equations leads to the final results $$Var[X]=\frac{N^{2}-1}{12}$$.

## Binomial Distribution
Considering $$Y\sim Binom(n,p)$$, this is just the sum of $$n$$ i.i.d. Bernoulli random variables.
For simplicity, let's consider two RVs $$X, Y$$ and solve the relationship between the mean/variance of their sum and the mean/variance of themselves.
First,
$$
\begin{align*}
E[X+Y]&=\sum_{x}\sum_{y}p(x,y)(x+y)=\sum_{x}x\sum_{y}p(x,y)+\sum_{y}y\sum_{x}p(x,y) \\
&=\sum_{x}xp(x)+\sum_{y}yp(y)=E[X]+E[Y] \\
\end{align*}
$$.

As you can see, other variables in the joint distribution are marginalized without any independence assumption.
By definition:
$$
\begin{align*}
Var[X+Y]&=E[((X+Y)-E[X+Y])^2]=\sum_{x}\sum_{y}p(x,y)((x+y)-E[X+Y])^{2} \\
&=\sum_{x}\sum_{y}p(x,y)((x-E[X])+(y-E[y]))^{2} \\
&=Var[X]+Var[Y]+\sum_{x}\sum_{y}2p(x,y)(x-E[X])(y-E[Y]) \\
\end{align*}
$$

If $$X$$ and $$Y$$ are independent, say that $$p(x,y)=p(x)p(y)$$, the last term can be wrote as $$2\sum_{x}p(x)(x-E[X])\sum_{y}p(y)(y-E[Y])=2\times 0\times 0=0$$.
Thus, $$Var[X+Y]=Var[X]+Var[Y]$$ if $$X$$ and $$Y$$ are independent.
The above analysis can be easily extend to the case of arbitrary number of variables.
According to the above analysis, the mean and variance of $$Y\sim Binom(n,p)$$ is $$np$$ and $$np(1-p)$$.

## Poison Distribution
First, the rate $$\lambda$ of $X\sim Poison(\lambda)=\frac{\lambda^{x}\exp{-\lambda}}{x!}$$ should be the rate of duration in question (see the example in that course material).
The mean and variance can be proved with a commonly used math trick as follow.

$$
\begin{align*}
E[X]&=\sum_{x=0}^{\inf}\frac{x\lambda^{x}\exp(-\lambda)}{x!} \\
&=\sum_{x=0}^{\inf}\frac{x\lambda^{x}\exp(-\lambda)}{x!} \\
&=\lambda\sum_{x=1}^{\inf}\frac{\lambda^{x-1}\exp(-\lambda)}{(x-1)!} \\
&=\lambda\sum_{x=0}^{\inf}\frac{\lambda^{x}\exp(-\lambda)}{x!}=\lambda \\
\end{align*}
$$

The last step is valid since the summation is exactly the pdf of the Poison distribution and thus sum up to 1.
The variance is induced in a similar way.

$$
\begin{align*}
E[X^2]&=\sum_{x=0}^{\inf}\frac{x^{2}\lambda^{x}\exp(-\lambda)}{x!} \\
&=\lambda\sum_{x=1}^{\inf}\frac{x\lambda^{x-1}\exp(-\lambda)}{(x-1)!} \\
&=\lambda\sum_{x=0}^{\inf}\frac{(x+1)\lambda^{x}\exp(-\lambda)}{(x)!} \\
&=\lambda^2+\lambda \\
\end{align*}
$$

Based on above proofs, both the mean and the variance of Poison distribution equal $$\lambda$$.

(Continuing...)
