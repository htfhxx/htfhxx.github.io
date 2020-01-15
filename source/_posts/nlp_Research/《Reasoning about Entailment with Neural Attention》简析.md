---
title: 《Reasoning about Entailment with Neural Attention》简析
date: 2019-12-13 23:02:00
author: 长腿咚咚咚
toc: true
mathjax: true
categories: nlp_Research
tags:
	- 《Reasoning about Entailment with Neural Attention》简析
---

[TOC]

## 1 介绍

ESIM是一个超强的文本蕴含任务模型。

文本蕴含任务就是，给定两个句子，第一个句子是前提——premise，第二个句子是假设——hypothesis，判断两个句子构成的关系——蕴含、不相干、矛盾。本质上是一个句子级的文本分类问题

## 2 网络结构

### 2.1 整体结构

​		模型的输入是premise和hypothesis。

​		论文使用两个LSTM（A）对两个句子进行编码，其中第二个LSTM使用的是第一个LSTM的末状态。

​		然后论文使用了两个Attention对编码后的信息进行处理，一种是Attention（B），用于图中$h_9$对$c_1，c_2,..,c_5$的注意力；一种是Attenion（C），用于图中$h_7,h_8,h_9$分别对$c_1，c_2,..,c_5$的注意力。

​		最后将以上信息进行整合，softmax分类，交叉熵损失。

![1576248174645](%E3%80%8AReasoning%20about%20Entailment%20with%20Neural%20Attention%E3%80%8B%E7%AE%80%E6%9E%90/1576248174645.png)

### 2.2 LSTM



### 2.3 Attention



### 2.4 word-by-word Attention



























































