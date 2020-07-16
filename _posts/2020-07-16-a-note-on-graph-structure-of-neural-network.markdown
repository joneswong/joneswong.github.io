---
layout: post
title:  "A Note on Graph Structure of Neural Network"
date:   2020-07-16 19:30:06 -0600
categories: automl
---

Today, we studied the ICML20 paper [Graph Structure of Neural Network](https://icml.cc/Conferences/2020/ScheduleMultitrack?event=5795) which attempts to answer the questions:
> - Is there a systematic link between the network structure and its predictive performance?
> - What are structural signatures of well-performing neural networks?
> - How do such structural signatures generalize across tasks and datasets?
> - Is there an efficient way to check whether a given neural network is promising or not?

Considering the limitations of computational graph (e.g., has to be DAG), the authors define **relational graph** for better representing a network.
**Relational graph** express the forward procedure of networks via **message exchange**.
Taking fully-connected network for instance:
> we can represent one input channel and one output channel together as a single node, and an edge in the relational graph represents the message exchange between the two nodes (Figure 1(a)).
> a message exchange is defined by a message function, whose input is a node’s feature (scalar, vector, or even tensor) and output is a message, and an aggregation function, whose input is a set of messages and output is the updated node feature.
> One neural network layer corresponds to one round of message exchange over a relational graph, and to obtain deep networks, we perform message exchange over the same graph for several rounds.

Formally, **message exchange** is defined by the Equation 1 and Table 1 presents how to express diverse networks by **relational graph**.

Once we represent a network by a **relational graph**, we can measure each graph by its:
- average path length: average shortest path between any pair of nodes.
- clustering coefficient: the proportion of edges between the nodes within a given node's neighborhood (denumerator is the total number of their possible edges), averaged over all nodes.

The search space of neural architectures is characterized by the two dimensions in this work.
From Figure 3, we know that traditional graph generation algorithms only cover a small region of the design space.
In contrast, this work proposes **WS-flex** which covers a large region of the design space.

To ensure the differences on accuracy (or error rate) come from the differences of graph structure, rather than the model capacity, the authors first compute the FLOPS (number of multiply-adds) of their baseline---a complete relational graph and use them as the reference complexity for each other network with different graph structure.

The authors conducted extensive experiments from which we can have the following conclusions (may be not that reliable IMO):
- for both fully-connected and convolutional neural networks, for both datasets, their are *sweet spots* on the two dimensional design space (see Fig 4 (a)(c)(f)) which are the graph structures with significantly better performance.
- the accuracy (or error rate) is approximately a second-order polynomial curve along with each of the measure (see Fig 4 (b)(d)).
- the performances of different graph structures when translated into different networks over different datasets are consistent (see Fig 4 (e)).
- the *sweet spots* can be quickly identified, say that, instead of evaluating tremendous graph structures (here 3942), we are allowed to subsample the design space (e.g., more than 1000), and the correlation is satisfactory.
- good relational graphs perform well even at the early training epoch (here around 3 epochs), which indicates that there may be no need to carefully train each network as usual.
- the *sweet spots* conincide with biological neural systems to some extent.

An interesting point mentioned is:
> instead of exhaustively searching over all the possible connectivity patterns, certain graph generators and graph measures could define a smooth space where the search cost could be significantly reduced.

The authors also show that the graph structure derived from learned network weights (i.e., parameters) approches the best structure along their training course.
I am wondering whether it is possible to set up the relationship between network weights and the measure of the derived graph structure, so that, once we have known the *sweet spot* of the design space, we can add some auxilliary objective to update the network weights aiming to approach that region.
