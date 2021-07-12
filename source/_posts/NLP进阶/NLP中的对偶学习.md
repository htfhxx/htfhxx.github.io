---
title: NLP中的对偶学习
date: 2020-07-07
author: 长腿咚咚咚
toc: true
mathjax: true
categories: NLP进阶
tags:
	- NLP中的对偶学习
---



## 1 介绍

### 1.1 2016-NIPS-Dual Learning for Machine Translation

神经机器翻译（NMT）的训练需要数以千万计的双语句子对。为了解决此培训数据瓶颈，开发了一种双重学习机制，该机制可使NMT系统通过双重学习自动从未标记的数据中学习。 

这种机制是受以下观察启发的：任何机器翻译任务都有双重任务，例如，英语到法语翻译（主要）与法语到英语翻译（双重）； 即使没有人工标记，原始任务和双重任务也可以形成一个闭环，并生成信息反馈信号来训练翻译模型。 在双重学习机制中，我们使用一个代理来代表主要任务的模型，并使用另一个代理来代表双重任务的模型，然后要求他们通过强化学习过程互相教导。



### 1.2 2017-ICML-Dual supervised learning  

许多有监督学习任务以双重形式出现，是对偶的。例如，英语到法语的翻译与法语到英语的翻译，语音识别与文本到语音的转换，图像分类与图像的生成。

由于其模型之间的概率相关性，两个双重任务之间具有内在联系。Dual supervised learning  则同时训练两个双重任务的模型，并明确利用它们之间的概率相关性来规范训练过程。 



概率的对偶性

<img src="NLP中的对偶学习\image-20200707163830395.png" alt="image-20200707163830395" style="zoom:67%;" />

任务目标

<img src="NLP中的对偶学习\image-20200707163912656.png" alt="image-20200707163912656" style="zoom:67%;" />



将概率对偶性的约束转化为下面的正则项

![image-20200707164221757](NLP中的对偶学习\image-20200707164221757.png)

算法流程

<img src="NLP中的对偶学习\image-20200707164338168.png" alt="image-20200707164338168" style="zoom:67%;" />





## 2 NLG &NLU

### 2.1 2019-INLG-Semi-Supervised Neural Text Generation by Joint Learning of Natural Language Generation and Natural Language Understanding Models  

提出了一种半监督式深度学习方案，该方案可以从未注释的数据和可用的注释数据中学习。 

它使用NLG和自然语言理解（NLU）序列到序列模型，这些模型可以共同学习以弥补标注数据的不足。 

<img src="NLP中的对偶学习\image-20200707175315658.png" alt="image-20200707175315658" style="zoom:67%;" />



### 2.2 2019-ACL-Dual Supervised Learning for Natural Language Understanding and Generation

自然语言理解（NLU）和自然语言生成（NLG）都是NLP和对话领域中的关键研究主题。 自然语言理解是从给定的话语中提取核心语义，而自然语言的生成则相反，其目的是根据给定的语义构造相应的句子。 

这种双重关系尚未在文献中进行研究。 本文提出了一种在双重监督学习基础上自然语言理解和生成的新颖学习框架，为利用双重性提供了一种方法。 





提出了一种基于双重监督学习的自然语言理解和生成的新型训练框架，该框架首先利用了NLU和NLG之间的对偶性，并将其引入学习目标中作为正则化项。

![image-20200713180452504](NLP%E4%B8%AD%E7%9A%84%E5%AF%B9%E5%81%B6%E5%AD%A6%E4%B9%A0/image-20200713180452504.png)

文章还解释了如何对$p(x)$和$p(y)$建模，最后表明，带上正则项的模型比两任务迭代训练的好一点。这篇是个短文，实验量不大。



### 2.3 2020-ACL-Towards Unsupervised Language Understanding and Generation by Joint Dual Learning  

在模块化对话系统中，自然语言理解（NLU）和自然语言生成（NLG）是两个关键组成部分，其中NLU从给定的文本中提取语义，而NLG则根据输入的语义表示构造相应的自然语言句子。

先前的工作（Su et al。，2019）是首次尝试利用NLU和NLG之间的双重性通过双重监督学习框架来提高绩效。 但是，先前的工作仍然以监督的方式学习了这两个组成部分。 

取而代之的是，本文引入了一种通用的学习框架来有效地利用这种对偶性，并提供了将有监督和无监督学习算法结合在一起以联合方式训练语言理解和生成模型的灵活性。 

![image-20200713180729134](NLP%E4%B8%AD%E7%9A%84%E5%AF%B9%E5%81%B6%E5%AD%A6%E4%B9%A0/image-20200713180729134.png)



训练算法，分为两部分：Primal Cycle and Dual Cycle  。 前者是让x生成y，生成的y再变回x，后者完全相反。其中，x是semantic representation ， y是 natural language sentences 。

论文提出了一些强化学习的优化目标，包括Explicit Reward  和Implicit Reward。

![image-20200713182645456](NLP%E4%B8%AD%E7%9A%84%E5%AF%B9%E5%81%B6%E5%AD%A6%E4%B9%A0/image-20200713182645456.png)

论文还提出了一些学习的范式，用来联通两个模型一起训练。Straight-Through Estimator，Distribution as Input  。

![image-20200713183152237](NLP%E4%B8%AD%E7%9A%84%E5%AF%B9%E5%81%B6%E5%AD%A6%E4%B9%A0/image-20200713183152237.png) 

反正我打我自己就完事了。











### 2.4 2020-ACL-A Generative Model for Joint Natural Language Understanding and Generation  
也是利用利用NLU和NLG之间的双重性。

提出了一个生成模型，该模型通过共享的潜在变量耦合NLU和NLG。 这种方法使我们能够探索自然语言和形式表示的空间，并促进通过潜在空间的信息共享，最终使NLU和NLG受益。 
还表明通过利用未标记的数据以半监督的方式训练模型可提高模型的性能。

<img src="NLP中的对偶学习\image-20200707180527625.png" alt="image-20200707180527625" style="zoom:67%;" />

NLG 和 NLU本质上是一对 VAE模型。通过x 和 y 可以得到z， 再通过x、z得到y 以及y、z得到x。



**监督学习**情况下的优化目标是：

![image-20200714145953409](NLP%E4%B8%AD%E7%9A%84%E5%AF%B9%E5%81%B6%E5%AD%A6%E4%B9%A0/image-20200714145953409.png)

可以转化为：

<img src="NLP%E4%B8%AD%E7%9A%84%E5%AF%B9%E5%81%B6%E5%AD%A6%E4%B9%A0/image-20200714150020533.png" alt="image-20200714150020533" style="zoom:67%;" />

<img src="NLP%E4%B8%AD%E7%9A%84%E5%AF%B9%E5%81%B6%E5%AD%A6%E4%B9%A0/image-20200714150026339.png" alt="image-20200714150026339" style="zoom:67%;" />



**无监督情况下**的优化目标：

![image-20200714150115230](NLP%E4%B8%AD%E7%9A%84%E5%AF%B9%E5%81%B6%E5%AD%A6%E4%B9%A0/image-20200714150115230.png)

![image-20200714150132883](NLP%E4%B8%AD%E7%9A%84%E5%AF%B9%E5%81%B6%E5%AD%A6%E4%B9%A0/image-20200714150132883.png)

![image-20200714150137802](NLP%E4%B8%AD%E7%9A%84%E5%AF%B9%E5%81%B6%E5%AD%A6%E4%B9%A0/image-20200714150137802.png)

最后的几种训练情况：

监督学习：![image-20200714150401634](NLP%E4%B8%AD%E7%9A%84%E5%AF%B9%E5%81%B6%E5%AD%A6%E4%B9%A0/image-20200714150401634.png)

半监督：![image-20200714150430817](NLP%E4%B8%AD%E7%9A%84%E5%AF%B9%E5%81%B6%E5%AD%A6%E4%B9%A0/image-20200714150430817.png)

无监督：![image-20200714150450655](NLP%E4%B8%AD%E7%9A%84%E5%AF%B9%E5%81%B6%E5%AD%A6%E4%B9%A0/image-20200714150450655.png)

















## Reference

[1] Dual Supervised Learning  

[2] Towards Unsupervised Language Understanding and Generation

by Joint Dual Learning. ACL 2020

[3] Semi-Supervised Neural Text Generation by Joint Learning of Natural

Language Generation and Natural Language Understanding Models ACL 2019

[4] A Generative Model for Joint Natural Language Understanding and Generation ACL 2020

[5] Dual Supervised Learning for Natural Language Understanding and Generation 2019 ACL