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

## On "Learning to Pre-train Graph Neural Networks"
Background: the dataset and the loss function considered in pre-train and fine-tune steps are different. The authors aim at fill this gap.

Idea: instead of directly minimizing the loss of pre-train task, this paper utilizes MAML to make the pre-trained GNN model useful for further adaptation. Specifically, in the pre-train step, they sample a support and a query set (where each node pair corresponds to an instance) from each graph to construct a task.

## On "Prevalence of Neural Collapse during the terminal phase of deep learning training"
Background: when we train a DNN for classification tasks, it is conventional to continue the training course, even though the training error has reached zero (i.e., 0-1 loss). This is named Terminal Phase of Training (TPT).

Key notations: there are $$C$$ classes, each of which has same number of training instances. The last activation (i.e., what is fed into the output layer) of the $$i$$-th instance of class $$c$$ is denoted as $$\mathbf{h}_{i,c}$$. The global-mean and class-mean is defined by averaging the corresponding activations and denoted by $$\mu_{G}$$ and $$\mu_c,c=1,\ldots,C$$ respectively. Furthermore, the total covariance, between-class covariance, and within-class covariance are defined accordingly and denoted by $$\Sigma_T$$, $$\Sigma_B$$, and $$\Sigma_W$$ respectively. The output layer (i.e., the classifier) is parameterized by $$W$$ where the $$c$$-th row is denoted by $$w_c$$.

Observations: during the TPT, there is a pervasive inductive bias called Neural Collapse which consists of:
- (NC1): Variability Collapse: $$\Sigma_W\rightarrow 0$$, i.e., activations of each class collapse into the same point.
- (NC2): Convergence to Simplex ETF: $$ \lvert \|\mu_c - \mu_G \|_2 - \|\mu_{c'} - \mu_G \|_2 \rvert \rightarrow 0,\forall c, c'$$, i.e., equal length; $$<\tilde{\mu_c}, \tilde{\mu_{c'}}>\rightarrow \frac{C}{C-1}\delta_{c=c'} - \frac{1}{C-1}$$, i.e., equal angle between any pair of class-mean, where $$\tilde{\mu}$$ is the $$\mu$$ normalized into a unit vector.
- (NC3): Convergence to self-duality: $$\|\frac{W^{T}}{\|W\|_F} - \frac{\dot{M}}{\|\dot{M}\|_F} \|_{F} \rightarrow 0$$, i.e., each $$\mu_c$$ and $$w_c$$ lie in the same direction and differ by only a scaling factor, where $$\dot{M}=[\mu_1-\mu_G,\ldots,\mu_C-\mu_G]$$.
- (NC4): Simplification to neraest class classification (NCC): comparing $$h$$ with $$w_c,c=1,\ldots,C$$ is equivalent to comparing with $$\mu_c,c=1,\ldots,C$$.

These are inspired by empirical observatioins presented in the Fig. 2-7.

Points: there is implicit inductive bias of architecture, SGD, and TPT that causes NC. With mean squared error or Cross-Entropy error, NC1-2 imply NC3-4.

Comments: to me, the magic is that why DNN use all except for the last layer (i.e., the classifier) to achieve such a perfect feature engineering, that is, the activations of instances belonging to the same class collapse into one vector. Once NC1-2 are observed, as the implication shows, NC3-4 come naturally.

## On "STRATEGIES FOR PRE-TRAINING GRAPH NEURAL NETWORKS"
Background: how to copy the success of pre-training for GNN like that of NLP and CV models.

Idea: pre-training of GNN should rely on both node-level and graph-level tasks, so that the GNNs capture both local and global patterns.

Methods: at node-level, there are two kinds of tasks:
- context prediction
- attribute masking

They define context of a node $$v\in G$$ as the subgraph containing nodes away from $$v$$ by $$[r_1, r_2], r_1 < K, r_2 > K$$ hops, where $$K$$ is the number of layers of the GNN to be pre-trained. Then a auxiliary context GNN is applied to encode the nodes in the context. The average of all the node embeddings corresponding to nodes that are belong to the overlapping of the context and the $$K$$-hop neighborhood of $$v$$ is used to represent the context of $$v$$. The context prediction task is a binary classification task of whether a particular neighborhood and a particular context graph belong to the same node. They use negative sampling to guide the training of considered GNN as well as the context GNN (see Eq. 3.1).
For the attribute masking task, a certain attribute in the input node features, e.g., atom type, is replaced by a pre-defined specific masking symbol, and a linear classifier is fed with the node embedding to predict the masked attribute.
At graph-level, they pre-train the GNN with multiple graph property prediction tasks. The authors conjecture that predicting the similarity between graphs is also a good task, but the quadratic complexity and the lack of a general similarity forbid this idea.
The whole pipeline proceeds in the order:
1. pre-train by node-level tasks and attain the GNN;
2. pre-train the GNN with multiple graph-level tasks;
3. fine-tune the GNN on a specific downstream task.

Observations:
- Using expressive GNN model is crucial to fully utilize pre-training
- If we only use a bunch of graph-level tasks for pre-training, some of them may be unrelated to the downstream task of interest and thus cause negative transfer. The proposed two-level, from-local-to-global pre-train work better.
