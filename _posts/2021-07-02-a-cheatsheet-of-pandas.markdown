---
layout: post
title:  "My Pandas Cheatsheet!"
date:   2021-07-02 08:24:02 -0700
categories: tools
---
For the ease of extracting and analyzing our experimental results, some basic operations upon pandas dataframe are recorded here. We assume a dataframe where each row corresponds to one kind of configuration, and each column corresponds to either one dimension of hyper-parameter or one kind of metrics.

load a csv file:
`df = pd.read_csv("agg/test_best.csv")`

select the row whose value at a given column is the smallest in the dataframe:
`df.loc[df['mae'].argmin()]`

groupby on a column and aggregate the rows by a given operator:
`df.groupby(['aggr'], as_index=False)['mae'].mean()`

in case rows that take a certain value on a column should be removed at first:
`df.loc[df['aggr'] != 'add'].groupby(['l_mp'], as_index=False)['mae'].mean()`
