---
title: NLP之文本表示
date: 2019-12-14
author: 长腿咚咚咚
toc: true
mathjax: true
categories: 深度学习与NLP基础
tags:
	- NLP之文本表示
---

### 1 词袋

词袋模型(Bag-of-words model)是将一段文本（比如一个句子或是一个文档）用一个“装着这些词的袋子”来表示，这种表示方式不考虑文法以及词的顺序。**「而在用词袋模型时，文档的向量表示直接将各词的词频向量表示加和」**。

缺点：词与词之间是没有顺序关系



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



word2vec 的优点：考虑了词与词之间的顺序，引入了上下文的信息



glove的改进：损失函数不同，引入了词语的共现信息（构建了共现矩阵）

* word embedding的缺陷：多义词



### 3 [ELMo](https://www.aclweb.org/anthology/N18-1202.pdf)

ELMO全称为：Embedding from Language Models。论文：《Deep contextualized word representation》

![图片](NLP%E4%B9%8B%E6%96%87%E6%9C%AC%E8%A1%A8%E7%A4%BA/640)

结构：两层双向LSTM，上下层的LSTM之间有residual connection ，加强了梯度的传播

训练：训练一个双向语言模型，前向语言模型和后向语言模型的概率对数和 为优化目标

下游：token embedding、双层LSTM的输出，三者分配权重，作为最后的embedding用于下游任务（实验证明每一层LSTM得到的embedding蕴含信息不一样，多种融合更丰富）



优点：基于的假设是词向量不应该是固定的，语言模型的作用可以利用上下文区分多义词。

缺点：LSTM特征抽取能力弱，双向特征融合的方式——拼接的方式融合特征能力较弱，参数固定



### 4 GPT：

GPT是“Generative Pre-Training”的简称

两个大改进：特征抽取器使用了**transformer**；用于下游任务时，GPT**参数可训练**

只使用上文预测下一个单词，抛开了下文（只是单向）



### 5 BERT

结构：Transformer的Encoder 12/24层，Hidden size 768/1024，自注意力的头12/16，总参数量100M/300M

预训练任务：


1. MaskLM: 随机挖掉15%单词，预测被挖掉的单词。被挖掉的单词，80%被替换为MASK，10%替换为另一个单词，10%原地不动
2. 句子关系预测，真正相连的句子和随机组合的两个句子

BERT的输入：三个embedding：位置、单词、句子

优点：Transformer提取特征能力强，

缺点：参数太多，模型太大，少量数据训练容易过拟合。



### References

https://zhuanlan.zhihu.com/p/49271699