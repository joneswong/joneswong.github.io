---
layout: post
title:  "A Note on Understanding and Simplifying One-shot Architecture Search"
date:   2020-07-31 20:30:06 -0600
categories: automl
---

Today, we studied the ICML18 paper [Understanding and Simplifying One-shot Architecture Search](http://proceedings.mlr.press/v80/bender18a.html). This paper empirically discusses the long-term concerns of one-shot NAS:
- By weight-sharing, does the performance of a stand-alone model correlates with that of its corresponding one-shot model?
- If it is correlated, what does the training of one-shot model actually learn?
- Which tricks for training the one-shot model are helpful for improving the correlations?

## Methodology
The search space in use is similar with what DARTS has proposed. In training the one-shot model, the most critic point is to
> To make sure that the one-shot model accuracies for specific architectures correlate well with stand-alone model accuracies

Useful tricks include:
- Path dropout: All operators are used in the training course, which are likely to co-adapt. An intuitive idea is to randomly dropout some pathes in the forward phase, so that the model is robust to such changes. Specifically, the dropout rate is 0 at the beginning and increase with a linear schedule.

- Batch normalization (BN): This has been proved helpful for stabilizing model training. However, as each batch the considered (retained) operators are different, statistics of the BN layer should be calculated on the fly. Besides, each mini-batch is further partitioned into several ghost batches where each ghost batch dropouts pathes in one way but different from others.

- Since only retained pathes contribute to the training loss, we should pose $L_2$ regularization upon them but the whole trainable parameters.

Random search, RL, and evolutional algorithms could be used for choose candidate architectures.
For evaluation, the top architectures selected by the one-shot model are trained from scratch and evaluate on test set.

## Empirical Results
From Fig. 4, we see that the dropout rate (a hyperparameter)
> must be tuned carefully in order to achieve good correlations between one-shot and stand-alone model accuracies

Only when the dropout rate is appropriately configured, the one-shot model accuracies of candidate architectures correlate with the stand-alone models' accuracies.

From Fig. 5, we surpricingly see that they are strongly correlated, but the accuracies of one-shot models lie in a much broader range (i.e., $[0.3, 0.9]$) but the accuracies of stand-alone models gathered around $0.9$.

From Table 1, we see that this method outperforms many other NAS methods, but the one-shot model favors large models. Although large stand-alone models achieve better accuracies correspondinly, what does the one-shot model learn and why does it prefer large models?

The authors' hypothesis is that:
> the one-shot model learns which operations in the network are most useful, and comes to rely on these operations when they are available

They first use an extremely small dropout rate to train an one-shot model and calcualte the symmetric KL divergence between the prediction (i.e., a distribution over the classes) of this model and the prediction of candidate architectures, on sampled training instances.
From Fig. 6, we observed a strong correlation between such KL divergence and the validation accuracy for those candidate architectures.
> This means that the closer a candidate architecture’s predictions are to those of our reference architectures (where most of the operations in the one-shot model are turned on), the higher its quality typically is during standalone training. Weight sharing implicitly forces the one-shot
model to identify and focus on the operations that are most
useful for generating good predictions