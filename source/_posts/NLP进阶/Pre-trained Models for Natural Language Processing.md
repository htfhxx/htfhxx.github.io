---
title: 《Pre-trained Models for Natural Language Processing》
date: 2020-04-27
author: 长腿咚咚咚
toc: true
mathjax: true
categories: NLP进阶
tags:
	- 《Pre-trained Models for Natural Language Processing》
---



[TOC]

语言表示学习及其研究进展。

如何使预训练模型的知识适应下游任务

预训练模型未来研究的一些潜在方向







## 1 论文概览

此篇论文主要有如下贡献：

1. 对NLP相关的预训练模型，进行了全面的介绍。 
   包括背景知识，模型结构，预训练的任务，一些扩展。
2. 从四个角度对现有预训练模型进行系统分类
   1）word表示的类型；  2）预训练模型的结构；  3）预训练任务类型；  4）特定类型的方案或输入的扩展
3.  收集了大量资源。
   包括开源实现，可视化工具，语料库和论文清单。
4. 对预训练模型的整体讨论。 
   讨论并分析现有预训练模型的局限性，建议了可能的未来研究方向。



## 2 背景知识

### 2.1 语言表示学习

一个好的语言表示应该表达通用的先验，它不是特定于任务的，但是对于解决各种任务可能很有用。因此，一个好的表示法应该捕获隐藏在文本数据中的隐含语言规则和常识知识，例如词汇含义，句法结构，语义角色，甚至是语用学。

分布式表示的核心思想是通过低维实值向量来描述一段文本的含义。向量的每个维度没有相应的意义，而整体则代表一个具体的概念。 

word embeddings（词嵌入）有两种：非上下文嵌入和上下文嵌入。 它们之间的区别在于，单词的嵌入是否会根据其出现的上下文而动态变化。

<img src="Pre-trained%20Models%20for%20Natural%20Language%20Processing/1585818716729.png" alt="1585818716729" style="zoom:50%;" />

非上下文嵌入有两个主要限制。
第一个问题是嵌入是静态的。 单词的嵌入与上下文无关，始终是固定的。 因此，这些非上下文嵌入无法建模多义词。 第二个问题是未登录词（out-of-vocabulary）问题。



### 2.2 为什么需要预训练

大力出奇迹需要巨大的超多参数的模型，需要更大的数据集来完全训练模型参数并防止过度拟合。

由于注释成本极其昂贵，因此对于大多数NLP任务而言，构建大规模的标记数据集是一项巨大的挑战。

相反，大规模的未标记语料库相对容易构建。 为了利用巨大的未标记文本数据，我们可以首先从它们中学习良好的表示形式，然后将这些表示形式用于其他任务。 最近的研究表明，借助从大型无注释语料库中的预训练模型提取的表示形式，可以在许多NLP任务上显着提高性能。

预训练的特长总结如下：

1. 可以学习通用的语言表示形式，并帮助完成下游任务；

2. 更好的模型初始化，可以带来更好的泛化性能并加快收敛速度；

3. 可以将预训练视为一种正则化，以避免对小数据过度拟合。



### 2.3 预训练模型的发展变化

第一代预训练模型旨在学习良好的word embeddings.

下游任务不再需要这些预训练模型模型本身，因此对于计算效率来说它们通常很浅，例如Skip-Gram和GloVe。 

这些经过预训练的嵌入可以捕获单词的语义，但它们不受上下文限制，无法捕获更高层次的文本概念，例如句法结构，语义角色，回指等。



第二代预训练模型关注于学习上下文的word embeddings. 例如CoVe, ELMo, OpenAIGPT and BERT.

不同于第一代预训练模型，第二代预训练模型在下游任务中仍然需要这些模型结构本身来表示特定上下文中的单词。此外，不像早期的word2vec只有skip-gram和cbow两种简单的任务，各种预训练任务也被提出使预训练模型能更好的学习。



## 3 预训练模型的分类

不同的预训练模型，主要取决于上下文encoders、预训练的任务类型，当然还有其对应的目的

### 3.1 神经上下文Encoder类型

大多数神经上下文编码器可分为三类：卷积模型，序列模型和基于图的模型。

一种更直接的方法是使用全连接的图对每两个单词的关系进行建模，然后让模型自己学习结构。 通常，连接权重是通过self-attention机制动态计算的，该机制会隐式指示单词之间的联系，并且因此可以建模长距离的依赖。一种成功的实现：Transformer。

可惜transformer由于其结构复杂且庞大，需要巨大的语料来训练，否则容易过拟合。这也是为什么要预训练的原因。

<img src="Pre-trained%20Models%20for%20Natural%20Language%20Processing/1585819538116.png" alt="1585819538116" style="zoom: 50%;" />



### 3.2 预训练任务

预训练任务可以概括为三类：监督学习，无监督学习和自监督学习。

其中监督学习就是从标注数据中学习从input到output的映射函数；无监督学习是在无标注数据中找到一些固有的知识，例如聚类、密度或潜在的表述等；自监督学习，是去预测输入的任意部分，accoding to输入的其他部分，例如masked语言模型。

![1587989122954](Pre-trained%20Models%20for%20Natural%20Language%20Processing/1587989122954.png)

#### 3.2.1 语言建模任务(Language Modeling —— LM)

语言建模任务是最常用的。

给定一个句子的前 t-1 个word，预测第 t 个word:
$$
p\left(x_{1: t}\right)=\prod_{t=1}^{T} p\left(x_{t} | x_{0: t-1}\right)
$$
$x_0$一般是类似$“\_BOS\_”$这样的字符。

其中，条件概率$p\left(x_{t} | x_{0: t-1}\right)$是通过对上文进行建模，得到的一个词汇的分布。

单向语言模型的一个缺点是每个token的表示仅编码其左边的上文及其本身。更好的文本上下文表示应从两个方向对上下文信息进行编码。 一种改进的解决方案是双向语言模型，由两个单向LM组成。





#### 3.2.2 掩码语言建模任务 (Masked Language Modeling——MLM)

类似于完形填空（Cloze）任务，MLM首先从输入语句中用类似[MASK]的字符掩住一些token，然后训练模型来预测标记所掩盖的标记。 

这种方法将在训练阶段和fine-tuning阶段之间产生不匹配，因为在fine-tuning阶段不会出现[MASK]  token。 为了解决这个问题，模型BERT在80％的时间内使用特殊的[MASK] ，在10％的时间内使用随机token，在10％的时间内使用原始token进行屏蔽。（来自BERT）

将屏蔽的序列馈送到神经编码器，该编码器对应token的隐藏单元进一步馈入softmax分类器，以预测屏蔽的token。BERT中only predict the masked words rather than reconstructing the entire input。或者，可以使用seq2seq的结构，屏蔽序列输入进编码器，解码器以自回归的方式顺序生成masked token。

当然还有很多加强版的MLM任务。





#### 3.2.3 排列语言建模 (Permuted Language Modeling —— PLM)

PLM一定程度上也可以解决 使用了一些类似于[MASK]字符后，导致预训练阶段和fine-truning阶段的差异问题。

从所有可能的排列中随机抽取排列。 然后，将这个随机序列中的某些token选择为目标，然后训练模型以预测这些目标。 此排列不会影响序列的自然位置，而仅定义token预测的顺序。
比如给定句子（w1, w2, w3, w4） 和（1,2,3,4）的一个Permutation，比如说（2,4,3,1），以及要预测的词w3，则根据词w2,w4来预测w3。因为在Permutation中3的前面是2,4。即目标词的context为目标词在Permutation中之前的词。



#### 3.2.4 降噪自编码器（Denoising Autoencoder —— DAE)

借助于DAE的原理，通过对输入序列做一系列的操作来加入“噪声”，然后训练模型去除加入的噪声来恢复原始的输入序列

 有几种破坏文本的方法：

1. 屏蔽token：从输入中随机采样token，并将其替换为[MASK]元素；
2. 删除token：从输入中随机删除token。与token屏蔽不同，该模型需要确定缺失输入的位置；
3. 文本填充：扩充文本并替换为单个[MASK]标记；
4. 打乱句子顺序
5. 文档旋转。选择一个token，将token之后的放入开头。



#### 3.2.5 对照学习假设（Contrastive Learning —— CTL)

一些观察到的文本对在语义上比随机取样的文本更为接近。
$$
\mathbb{E}_{x, y^{+}, y^{-}}\left[-\log \frac{\exp \left(s\left(x, y^{+}\right)\right)}{\exp \left(s\left(x, y^{+}\right)\right)+\exp \left(s\left(x, y^{-}\right)\right)}\right]
$$
CTL背后的想法是“通过比较学习”。 与LM相比，CTL通常具有较少的计算复杂性，因此是PTM的理想替代训练标准。



### 3.3 预训练模型的分类

从以下几个方面对预训练模型进行分类：

1. 表示类型：根据用于下游任务的表示，可以将PTM分为非上下文模型和上下文模型。
2. 网络结构：包括LSTM，Transformer encoder，Transformer decoder和完整的Transformer。  
3. 预训练任务类型
4. 扩展：针对各种场景而设计的预训练模型。包括加入丰富知识的PTM，多语言或特定语言的PTM，多模型PTM，特定领域的PTM和压缩的PTM。 

![1587989390793](Pre-trained%20Models%20for%20Natural%20Language%20Processing/1587989390793.png)

![1587989086246](Pre-trained%20Models%20for%20Natural%20Language%20Processing/1587989086246.png)

## 4 预训练模型的拓展

Knowledge-Enriched PTMs

Multilingual and Language-Specific PTMs

Multi-Modal PTMs

Domain-Specific and Task-Specific PTMs

Model Compression

- **模型剪枝**，即删除不重要的参数
- **权重量化**，即使用少量的位去表示参数
- **参数共享**，即相同类型的层间进行参数共享
- **知识蒸馏**，用知识蒸馏的方式将老师网络中学到的知识转移到容量较小的学生网络中



## 5 Adapting PTMs to Downstream Tasks

尽管PTM可以从大型语料库中获取通用语言知识，但是如何有效地将其知识适应下游任务仍然是关键问题。

### 5.1 迁移学习

<img src="Pre-trained%20Models%20for%20Natural%20Language%20Processing/1587989804104.png" alt="1587989804104" style="zoom:50%;" />

### 5.2 how to transfer?

Choosing appropriate pre-training task, model architecture and corpus

Choosing appropriate layers

To tune or not to tune?

### 5.3 Fine-Tuning Strategies





## Reference

https://arxiv.org/abs/2003.08271

https://blog.csdn.net/Forlogen/article/details/105475933

https://zhuanlan.zhihu.com/p/69989155

https://www.cnblogs.com/shona/p/12546820.html





