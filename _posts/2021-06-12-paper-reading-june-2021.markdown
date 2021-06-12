---
layout: post
title:  "Paper Reading (June, 2021)"
date:   2021-06-12 17:24:02 -0700
categories: research
---

For the sake of looking up empirical/theoretical observations/conclusions conveniently, I decide to regularly write down some notes here about the papers studied.

## On "An Investigation of Why Overparameterization Exacerbates Spurious Correlations"
Background: each instance $$x$$ is associated with a label $$y$$ and, in addition, a spurious label $$a$$ (see Fig. 2). The instances could be partitioned into different groups $$g=(y, a)$$, and we are focusing on minimize the *worst-group* error. By ERM, no matter whehter the model is over-parameterized or under-parameterized, it is poor. Thus, the standard strategy is to reweight the training instances by $$\frac{1}{\hat{p}_g}$$ where $$\hat{p}_g$$ represents the proportion of its group.

Observations: in the over-parameterization regime, although the model can achieve zero training error, its worst group error becomes worse. The model uses the patterns that cannot generalize well to correctly classify the training instances of the minority group. This paper studied the reason behind such a behavior.

Points: (To be studied) 1. properties of dataset that make this trend. 2. inductive bias towards memorizing the fewer points.
