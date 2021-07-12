---
title: 预训练模型具体工作survey
date: 2020-05-23 23:02:00
author: 长腿咚咚咚
toc: true
mathjax: true
top: true
categories: NLP进阶
tags:
	- 预训练模型具体工作survey
---



## 1 预训练模型综述

### 1.1 历史进展

![image-20201102103611033](%E9%A2%84%E8%AE%AD%E7%BB%83%E6%A8%A1%E5%9E%8B%E5%85%B7%E4%BD%93%E5%B7%A5%E4%BD%9Csurvey/image-20201102103611033.png)



**第一代自然语言预训练模型：词向量模型**
I 典型代表：CBOW, Skip-gram, Glove, Fasttext
I 词向量表示是固定，不会随着上下文的改变而变化  



**第二代自然语言预训练模型：预训练语言模型**
I 典型代表：ELMo, BERT, GPT
I Pre-training-then-fine-tuning已经成为NLP研究新范式
I 将在pre-training阶段学习到的语言表示迁移到下游任务  



### 1.2 发展方向

1. **大力出奇迹：大模型带来使用方式的变化**  

   预训练词向量：
   I 将NLP带入神经网络时代
   I 普通提高了NLP各项任务的水平
   预训练语言模型（ELMo、BERT、GPT）：
   I 采用预训练+微调模式，大大简化NLP模型设计
   I 刷新各项NLP任务的性能
   预训练语言模型（GPT-3）：
   I 超大模型，无需微调，直接Zero-shot方式既可以完成各项NLP任务  

2. 更小巧：压缩与加速
   I 各种预训练模型被各大公司竞相提出
   I 先做大阶段:“大算力+大模型+大数据+创意任务”探索能力边界
   I 再做小阶段: 在各种下游任务上形成生产力（对话/阅读理解/搜索等）  

3. 更优秀：功能更多、性能更高、训练更快
   更好的模型结构、更难的训练任务、更多的功能  

4. 更聪明：外部知识融入  

5. 更能干：跨界出圈 （语音、图像等）





### 1.3 训练任务

I Baseline: LM(GPT,ELMo), MLM(BERT), NSP(BERT)
I Whole Word Masking (BERT)、SpanBERT (Joshi et al. 2019)
I RTD (Replaced Token Prediction): Electra(Clark 2020)
I SOP (Sentence Order Prediction): ALBERT(Lan et al. 2020)
I DAE (Denoising Autoencoder (DAE): BART(Mike et al. 2019)
I Multi-task Learning: MT-DNN(Liu et al. 2019)
I Generator and Discriminator: Electra(Clark 2020)  



* LM(GPT,ELMo)
* Masked Language Modeling (MLM)  (BERT)
  * Sequence-to-Sequence MLM (Seq2Seq MLM)  
  * Enhanced Masked Language Modeling (E-MLM)  
    * RoBERTa：动态mask而不是静态
    * UniLM：将mask任务拓展成三种LM任务，单向双向和seq2seqMLM
    * spanBERT：替代MLM为Random Contiguous Words Masking and Span Boundary Objective 
    * structBERT：Span Order Recovery task   
* Permuted Language Modeling (PLM)  
  * 取代MLM，对输入序列随机排列，置换序列中mask一些token去预测，根据的是这些token的原始位置和其他token
* Denoising Autoencoder (DAE)  
* cc：破坏文本并恢复：mask/token delete/文本跨度mask预测缺失数量/句子打乱排列预测/随机选择一个token并旋转，预测序列开始位置
* Contrastive Learning (CTL)  对比学习
  * 对比学习假设观察到的某些文本对在语义上比随机采样的文本更相似。
  * Deep InfoMax  预测正常ngram比打乱的概率大
  * Replaced Token Detection (RTD)  预测一个token是否被代替
  * Next Sentence Prediction (NSP)  
  * Sentence Order Prediction (SOP)  



## 2 具体工作

### 自回归型PLM

**ELMO**

双层双向的LSTM语言模型，特征集成应用到下游任务

**GPT / GPT2**

单向transformer，自回归语言模型。GPT2训练规模更大参数更大，模型结构微调。

自回归型PLM





### 自编码PLM

**BERT**

Masked LM 获取上下文表征，NSP进行句子级建模。

缺点：生成任务效果差，无法进行文档级别的任务



### 引入外部知识

**ERNIE1.0（Baidu） 引入知识（实体）**

在预训练阶段，预先识别出实体，采用3中mask预测任务：随机subword mask，Phrase-Level Masking，Entity-Level Masking

**ERNIE（THU）引入知识（实体对齐-知识嵌入）**

在有实体输入的位置，将实体向量和文本表示通过**非线性变换进行融合**，以融合词汇、句法和知识信息。引入预测实体任务。



### 改进Mask策略

**Bert-wwm 整词mask**

原生BERT模型：按照subword维度进行mask，然后进行预测；局部的语言信号，缺乏全局建模的能力。

BERT WWM(Google)：按照whole word维度进行mask，然后进行预测；

**ERNIE1.0（Baidu） 引入知识（实体）**

在预训练阶段，预先识别出实体，采用3中mask预测任务：随机subword mask，Phrase-Level Masking，Entity-Level Masking

**SpanBERT**

采取随机mask一段空间长度，预测span中被mask的部分。





### 引入多任务学习

**MTDNN（下游任务引入多任务学习）**

单句分类、句对相似度、句对分类、句对排序  多任务一同finetune

**ERNIE2.0 （Baidu）引入知识（实体）**

ERNIE 2.0 是在预训练引入多任务学习，构建多个层次的任务全面捕捉训练语料中的词法、结构、语义的潜在知识。词法层面、结构层面、语义层面。3大类任务的若干个子任务一起用于训练。

构建**增量学习（**后续可以不断引入更多的任务**）**模型，通过多任务学习**持续更新预训练模型**，这种**连续交替**的学习范式**不会使模型忘记之前学到的语言知识**。





### 精细调参

**RoBERTa(FaceBook)**

丢弃NSP，效果更好；动态改变mask策略，把数据复制10份，然后统一进行随机mask；

对学习率的峰值和warm-up更新步数作出调整；在更长的序列上训练： 不对序列进行截短，使用全长度序列；

![1617252934206](%E9%A2%84%E8%AE%AD%E7%BB%83%E6%A8%A1%E5%9E%8B%E5%85%B7%E4%BD%93%E5%B7%A5%E4%BD%9Csurvey/1617252934206.png)

### 大改进

**XLNET**

针对自回归PLM无法对上下文进行表征。具体怎么做才能让这个模型：看上去仍然是从左向右的输入和预测模式，但是其实内部已经引入了当前单词的下文信息呢？

引入排列语言模型：原LM的过程中引入打乱的序列，这样在预测当前token时可以看到一部分下文信息。





**ELECTRA**

把生成式的Masked language model(MLM)预训练任务改成了判别式的Replaced token detection(RTD)任务，判断当前token是否被语言模型替换过，**序列标注任务**。

随机替换太简单，那么就用一个MLM的G-BERT来对输入句子进行更改（MLM任务，输入的是mask的句子），然后去判断（序列标注）。

两者一同训练，但梯度不传递（离散的中间结果）

![1618908926156](%E9%A2%84%E8%AE%AD%E7%BB%83%E6%A8%A1%E5%9E%8B%E5%85%B7%E4%BD%93%E5%B7%A5%E4%BD%9Csurvey/1618908926156.png)



**Transformer-XL 文档级的建模**

如果序列长度超过固定长度，处理起来就比较麻烦。一种处理方式，就是将文本划分为多个segments。

这存在两个问题，1）因为segments之间独立训练，所以不同的token之间，最长的依赖关系，就取决于segment的长度；2）出于效率的考虑，在划分segments的时候，不考虑句子的自然边界，而是根据固定的长度来划分序列，导致分割出来的segments在语义上是不完整的。

在Trm的基础上，Trm-XL提出了一个改进，在对当前segment进行处理的时候，缓存并利用上一个segment中所有layer的隐向量序列，而且上一个segment的所有隐向量序列只参与前向计算，不再进行反向传播，这就是所谓的segment-level Recurrence。









## Reference

https://arxiv.org/abs/2003.08271

pre-trained-langauge-models-research-advances-and-prospectives 刘群

https://zhuanlan.zhihu.com/p/346964839

https://zhuanlan.zhihu.com/p/84159401

https://zhuanlan.zhihu.com/p/76912493

https://zhuanlan.zhihu.com/p/71759544

https://zhuanlan.zhihu.com/p/89763176

https://zhuanlan.zhihu.com/p/70257427











