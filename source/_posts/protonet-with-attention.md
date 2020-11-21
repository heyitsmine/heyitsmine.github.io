---
title: 在prototypical network中加入注意力机制
date: 2020-11-09 21:30:43
categories:
- deep learning
tags:
- few-shot
- prototypical network
---

# prototypical network 原型网络

# Attention

## Context attention

在prototypical network中，每个类别的原型由支持集中该类别所有样本的向量表示取均值而得。然而，在实际中每个类别的含义很丰富，在不同的样本中含义存在一定差别。因此，直接对向量表示取平均值获得的原型不够准确。

基于支持集中不同样本重要性不同的事实，提出上下文注意力机制，向与原型更相关的样本分配更高的权重。计算样本向量表示$\bold{S}$之间的矩阵乘积再除以$\sqrt{d_w}$以表示$\bold{S}$中样本之间的相关性，再对其使用$softmax$获得每个样本的权重。最终的$\bold{S}_{new}$由权重乘以样本向量表示获得：

$\bold{S}_{new}=CATT(\bold{S})=softmax(\frac{ss^T}{\sqrt{d_w}})\bold{S}$

## Local Matching and Aggregation

## Instance Matching and Aggregation

## Class Matching

## Instance-level Attention

## Feature-level Attention

[用词] prototype, vector

