---
title: nlp中的预训练
date: 2019-12-14
author: 长腿咚咚咚
toc: true
mathjax: true
categories: nlp基础
tags:
	- nlp中的预训练
---



[TOC]



### 1 图像领域的预训练

* 图像领域（CNN系列模型）预训练的步骤：
  1. 先用某个训练集合、在任务A上进行预训练，并保存参数；
  2. 采取相同的网络结构处理任务B，并在浅层结构中加载A任务学习好的参数，其它高层结构参数仍然随机初始化
  3. 之后我们用B任务的训练数据来训练网络，有两种做法：
     * 浅层加载的参数在训练B不更新，称为Frozen;
     * 浅层加载的参数在训练B更新，称为Fine-Tuning

* 为什么有效：
  * 不同层级的CNN学到不同层级的特征，浅层特征的通用性更广，高层特征与任务更相关

### 2 word2vec & glove

NLP领域的预训练最开始依靠的是word embedding，其中最知名的是word2vec，后来的glove在word2vec上进行改进。

word2vec就是训练了一个语言模型，其副产物（得到的参数矩阵）可以作为词的分布式表示

glove的改进：损失函数不同，引入了词语的共现信息（构建了共现矩阵）

* word embedding的缺陷：多义词

### 3 ELMO：解决多义词问题

​		ELMO是“Embedding from Language Models”的简称

​		论文：《Deep contextualized word representation》

​		原理本质：**先预训练一个word embedding**；下游任务的实际使用时，**使用上下文单词的语义去调整**（参数是不可训练的）

​		结构：双层双向的LSTM进行编码，语言模型作为训练任务

​		缺点：LSTM特征抽取能力弱，双向特征融合的方式——拼接的方式融合特征能力较弱



### 6 GPT：

​		GPT是“Generative Pre-Training”的简称

​		两个大改进：特征抽取器使用了**transformer**；用于下游任务时，GPT**参数可训练**

​		只使用上文预测下一个单词，抛开了下文（只是单向）

### 7 Bert

​		GPT中的单向改为双向

​		NLP四大任务：序列标注、文本分类、文本关系推断（两个输入）、文本生成

​		普适性强：适用于以上任务

BERT的关键：**Transformer**作为特征抽取器，**双向**语言模型，预训练是一个**多任务的训练**，更巨大的语料

如何改造双向语言模型：


1. 随机挖掉15%单词，预测被挖掉的单词。被挖掉的单词，80%被替换为MASK，10%替换为另一个单词，10%原地不动
2. 句子关系预测，真正相连的句子和随机组合的两个句子



BERT的输入：三个embedding：位置、单词、句子



### References

https://zhuanlan.zhihu.com/p/49271699
