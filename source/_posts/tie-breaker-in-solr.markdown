---
layout: post
title: "solr中的tie breaker"
date: 2014-10-22 17:14
comments: true
categories: solr
tags: [ solr, lucene, tie ]
---
solr 查询参数中有tie这样的一个参数，下面是它的官方解释：


**The tie (Tie Breaker) Parameter**

```
The tie parameter specifies a float value (which should be something much less than 1) to use as tiebreaker in DisMax queries.
When a term from the user's input is tested against multiple fields, more than one field may match. If so, each field will generate a different score based on how common that word is in that field (for each document relative to all other documents). The tie parameter lets you control how much the final score of the query will be influenced by the scores of the lower scoring fields compared to the highest scoring field.
A value of "0.0" makes the query a pure "disjunction max query": that is, only the maximum scoring subquery contributes to the final score. A value of "1.0" makes the query a pure "disjunction sum query" where it doesn't matter what the maximum scoring sub query is, because the final score will be the sum of the subquery scores. Typically a low value, such as 0.1, is useful.
```
读起来比较令人费解。

简单解释就是：

这个tie参数通常是一个小于1的浮点数，用于defType=disMax的solr查询。当查询命中多个field的时候，最终的score获得多少将由这个tie参数来进行调节。比如命中了field1，field2这2个field。

如果field1.score= 10，field2.score=3。那么 score = 10 + tie * 3.

也就是说，如果tie=1的话，最终的score就相当于多个字段得分总和;如果tie=0,那么最终的score就相当于是命中的field的最高分。

通常情况下呢，官方推荐tie=0.1。


