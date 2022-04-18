---
title: 'paper reading: A gentle introduction to graph neural networks'
author: daydreamatnight
declare: true
date: 2022-04-14 13:58:38
tags:
  - paper reading
  - deep learning
---

> This is a series of paper reading notes, hopefully, to push me to read paper casually and to leave some record of what I've learned.

> This is a tech blog written by google research team in 2021 that introducing the graph neural network. GNN has gradually become popular in the last 4 years. Personally, I think the graph structure looks similar to the CFD mesh, and there are works  focusing on simulating physics via GNN.

<!-- more -->

Paper link: [A gentle introduction to graph neural networks](https://staging.distill.pub/2021/gnn-intro/?ref=https://githubhelp.com)

Useful link: https://www.bilibili.com/video/BV1iT4y1d7zP

## Notes

*Because this blog has introduced GNN in detail and explained well with the interactive diagrams, it almost leaves me no need for extra notes. As a result, a throughout reading of the [original blog](https://staging.distill.pub/2021/gnn-intro/?ref=https://githubhelp.com) is recommended. And as a result this notes are only in pieces.*

<img src=" GNN interative archtecture.png" alt="GNN interative archtecture" style="zoom:50%;" />

>### Graph-level task
>
>In a graph-level task, our goal is to predict the property of an entire graph. For example, for a molecule represented as a graph, we might want to predict what the molecule smells like, or whether it will bind to a receptor implicated in a disease.
>
><img src=" GNN graph level task.png" alt="GNN graph level task" style="zoom:50%;" />

The example is actually a simple task example, and the loops can be detected with ordinary algorithm such as: [Fast and Slow Pointer: Floyd's Cycle Detection Algorithm](https://codeburst.io/fast-and-slow-pointer-floyds-cycle-detection-algorithm-9c7a8693f491). But with more complicated task, GNN can be useful. And here is the related Leetcode question: [No.141: Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/description/).



> <img src=" GNN graph message passing.png" alt="GNN graph message passing" style="zoom:50%;" />
>
> This is **reminiscent of standard convolution**: in essence, message passing and convolution are operations to aggregate and process the information of an element’s neighbors in order to update the element’s value. In graphs, the element is a node, and in images, the element is a pixel. **However**, the number of neighboring nodes in a graph can be variable, unlike in an image where each pixel has a set number of neighboring elements.

When introducing the message passing method, this blog makes an analogy with the convolution. And in addition to the however part mentioned in this blog, another difference is that in convolution, each element's value is weighted differently while in the aggregation method they are all the same. 

Interestingly, similar weights can be achieved with Graph Attention Networks, which is introduced in later section.

> Another way of communicating information between graph attributes is via attention. For example, when we consider the sum-aggregation of a node and its 1-degree neighboring nodes we could also consider using a weighted sum.
>
> A common scoring function is the inner product and nodes are often transformed before scoring into query and key vectors via a linear map to increase the expressivity of the scoring mechanism.

## Reviews

Writing: the whole article is well coherent and fluent, building the knowledge of GNN step by step, from a highly simplified model to the real GNN. The beautiful interactive figures make the article easy to read and digestible. Yet lacking mathematics and codes is both pros and cons. Unfortunately, *tarting today Distill will be taking a one year hiatus, which may be extended indefinitely.* 

Graph neural network: a graph is a powerful tool so that all kinds of data can be described as a graph. But this power leads to a huge difficulty in optimisation. One reason is the sparsity, the dynamic structure makes it difficult the train on CPU or GPU. Another is that GNN is very sensitive to hyper-parameters, just like the experiment section of this blog shows. As such, it is an active research area yet rarely deployed in industry.
