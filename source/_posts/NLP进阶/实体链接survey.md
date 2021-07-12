---
title: 实体链接survey
date: 2021-04-21
author: 长腿咚咚咚
toc: true
mathjax: true
categories: NLP进阶
tags:
	- 实体链接survey
---



## 1 实体链接

### 1.1 任务定义

实体链接，即 命名实体识别+命名实体歧义消除（到知识库）。



例子：

![1618990310893](%E5%AE%9E%E4%BD%93%E9%93%BE%E6%8E%A5survey/1618990310893.png)



形式化定义：

Entity Recognition (ER) and Entity Disambiguation (ED)

<img src="%E5%AE%9E%E4%BD%93%E9%93%BE%E6%8E%A5survey/1618991552050.png" alt="1618991552050" style="zoom:50%;" />

<img src="%E5%AE%9E%E4%BD%93%E9%93%BE%E6%8E%A5survey/1618991563611.png" alt="1618991563611" style="zoom:50%;" />

### 1.2 相关数据集

AIDA CoNLL-YAGO数据集

#### TAC KBP英语实体链接综合和评估数据2010



### 1.3 评价指标

实体相关度的评价

nDCG@1 、nDCG@5 、 nDCG@10 、 MAP 

仅实体消歧的评价

- Micro-Precision: Fraction of correctly disambiguated named entities in the full corpus.
- Macro-Precision: Fraction of correctly disambiguated named entities, averaged by document.

end-to-end 的评价

- Gerbil Micro-F1 - strong matching: micro InKB F1 score for correctly linked and disambiguated mentions in the full corpus as computed using the Gerbil platform. InKB means only mentions with valid KB entities are used for evaluation.
- Gerbil Macro-F1 - strong matching: macro InKB F1 score for correctly linked and disambiguated mentions in the full corpus as computed using the Gerbil platform. InKB means only mentions with valid KB entities are used for evaluation.



### 1.4 开源工具

https://github.com/informagi/REL

https://github.com/facebookresearch/BLINK



## 2. 基础方法

![1618992254711](%E5%AE%9E%E4%BD%93%E9%93%BE%E6%8E%A5survey/1618992254711.png)

EL主要包括两个步骤：

1. 实体识别，从普通文本中提取出实体mention
2. 实体消歧，为给定提及预测相应的实体。
   实体消歧又分为两个步骤：
   1. 候选实体的生成、可能实体的产生
   2. 实体排序，涉及 上下文/mention-候选实体 相似性分数的计算



### 2.1 候选实体生成

这一步是根据mention给出一个模棱两可的实体列表

<img src="%E5%AE%9E%E4%BD%93%E9%93%BE%E6%8E%A5survey/1618992439845.png" alt="1618992439845" style="zoom:50%;" />

surface form matching ：大致是根据n-gram、levenshtein distance等进行模糊匹配

dictionary lookup：构建附加别名的词典（例如使用wiki生成别名字典等）

prior probability ：预先计算好每个mention链接到entity的超链接数（例如wiki百科词条上的链接）

![1618992626717](%E5%AE%9E%E4%BD%93%E9%93%BE%E6%8E%A5survey/1618992626717.png)



### 2.2 Context-Mention Encoding

<img src="%E5%AE%9E%E4%BD%93%E9%93%BE%E6%8E%A5survey/1618992776331.png" alt="1618992776331" style="zoom:50%;" />

本质上是各种句子级的编码方式：RNN、RNN+Attention、BERT

不同点在于可能涉及到：用各种方法表示mention向量、让mention之间交互得到全局语义





### 3.1.3. Entity Encoding 

<img src="%E5%AE%9E%E4%BD%93%E9%93%BE%E6%8E%A5survey/1618992821368.png" alt="1618992821368" style="zoom:50%;" />

一般情况下，类似于词级别的分布式表示

根据数据源，实体表示模型可能有很大的不同



### 3.1.3. Entity Ranking

<img src="%E5%AE%9E%E4%BD%93%E9%93%BE%E6%8E%A5survey/1618993429290.png" alt="1618993429290" style="zoom:50%;" />



<img src="%E5%AE%9E%E4%BD%93%E9%93%BE%E6%8E%A5survey/1618992920721.png" alt="1618992920721" style="zoom:67%;" />

另外，某些mention不存在知识库中的对应实体，需要有方法判断是否不存在：设立阈值、候选实体中额外加一个NIL实体、分类时对是否存在进行额外分类等



## 3 改进方法

### 3.1 NR+ND联合建模



### 3.2 Global Context Architectures 

对每个mention的候选实体进行关联强度的计算

弊端是组合数量规模大、关联强度的标注比较困难

![1618994086807](%E5%AE%9E%E4%BD%93%E9%93%BE%E6%8E%A5survey/1618994086807.png)



### 3.3 Domain-Independent Architectures 

主要是克服标注数据的问题

半监督、无监督、远程监督、zero-shot的方法



### 3.4 Cross-lingual Architectures 



## 4 跨空间的实体关联？

比如又链接到维基百科又链接到百度百科？

近几年的工作都是从单个实体库里链接实体。因为所有数据集标的都是WikiPedia。

有zero-shot实体链接，是研究怎么泛化到一个新的实体库的。但是没有那种，可以测试多个实体库的测试数据？

这种实体库都是那种大而全的吗？有没有那种分类的？基本都是wikidata那种通用实体库



## Reference

《Neural Entity Linking: A Survey of Models based on Deep Learning》

