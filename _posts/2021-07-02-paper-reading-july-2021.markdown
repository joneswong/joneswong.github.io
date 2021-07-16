---
layout: post
title:  "Paper Reading (July, 2021)"
date:   2021-07-02 17:24:02 -0700
categories: research
---

## On "From Local Structures to Size Generalization in Graph Neural Netorks"
Local structure depends on graph size, e.g., in $$G(n, p)$$, the expected degree is $$np$$, hence fixing $$p$$ and increasing $$n$$ changes the local structure of the graph.

To characterize the local structure by graph d-pattern: def. 4.1. and fig. 2.
d-pattern is defined in a recursive way and is motivated by WL test (and the analysis in GIN), e.g., 1-pattern represents the node's degree, and a 2-pattern of a node is a pair: (its degree, the set of its neighbor's degrees).

To connect GNN with d-patterns:
On one side, GNN can be programed to output any value on any d-pattern independently. (more precise version is Thr. 4.3)
Conversely, Thr. 4.2: d-layer GNN (with a node-level output additionally) will output the same results for nodes with the same d-patterns.
Combining them, we can independently control the (output) values of d-layer GNNs on the set of d-patterns and these values completely determine the GNN's output.

The relation between size generalization and d-pattern discrepancy:
Thr. 5.1
$$P_1, P_2$$ be finitely supported distributions of graphs
$$P_1^d, P_2^d$$ the distribution of d-patterns over $$P_1, P_2$$ respectively
assume that any graph in $$P_2$$ contains a node with a d-pattern in $$P_2^d \setminus P_1^d$$
then any regression task solvable by a d-depth GNN, there exists a (d+3)-depth GNN that
- perfectly solves the task on $$P_1$$
- predicts with an *arbitrary* error on $$P_2$$
The main idea is to leverage the unseen d-patterns from $$P_2^d$$ to change the output on graphs from $$P_2$$
Then Thr.5.2
The main idea is pick a set of d-patterns that has small probability, i.e., $$\epsilon$$ on $$P_1^d$$ but large probability in $$P_2^d$$.

To improve size generalization:
consdier domain adaptation setting where we have access to labeled samples from the source domain but the target domain is unlabeled
SSL: propose node-regression pretext task, that is, to regression the histogram of each feature field where the histogram is aggregated at the focused node from its d-depth tree (d-hop neighborhood)
