---
title: 可控文本生成
date: 2018-12-28 14:22:00
author: 长腿咚咚咚
toc: true
mathjax: true
top: true
categories: NLP进阶
tags:
	- 可控文本生成
---



## 可控文本生成survey

## 1. 问题描述

**文本生成**是一个较宽泛的概念，广义上只要输出是自然语言文本的各类问题都属于这个范畴，包括从文本、数据、图像等生成文本。

目前我们侧重于text -> text。data -> text 虽然也较为常用，在一些端到端的生成方法中，基本也都是处理成text进行输入。

<img src="%E5%8F%AF%E6%8E%A7%E6%96%87%E6%9C%AC%E7%94%9F%E6%88%90survey/1616136959580.png" alt="1616136959580" style="zoom: 67%;" />

但是只有text->text是不够的，在真实的应用场景中，文本都是受限或者可控的。

在实际应用中，除text->text之外，还有**约束可控的文本生成**，是在传统NLG的基础上，通过加入受限的条件，比如主题信息、情感倾向、句子长短等，从而让输出的文本带有受限的属性。   

![1616137024841](%E5%8F%AF%E6%8E%A7%E6%96%87%E6%9C%AC%E7%94%9F%E6%88%90survey/1616137024841.png)

例如最常见的，控制句子长短、控制要生成的文本中包含一些关键词等等。例如上图 1.在对话场景中生成不同感情的回复；2.通过主题写文章，本质上是通过关键字来控制生成文本。

本次分享侧重于这种 受限文本生成 或 约束可控文本生成。



## 2. 任务及方法分类

接下来是我基于目前见过的文章和初步的调研，对目前可控文本生成任务的一个分类和总结，后续会随着进行扩充和改进。

### **2.1 无约束的文本生成**

 text ->text：本质上是不需要在模型和数据层面添加额外的约束

例如对联，摘要，以输入为开头的故事等等，这类很大程度上依赖数据 和 模型拟合的程度

<img src="%E5%8F%AF%E6%8E%A7%E6%96%87%E6%9C%AC%E7%94%9F%E6%88%90survey/1616137286754.png" alt="1616137286754" style="zoom: 67%;" />

### 2.2 **主题限定**     

 topic(+text) ->text：根据主题生成文本  

单纯的topic 到 text，本质上也是text->text。所以我觉得更应该看做在text ->text的基础上加入topic的限制，这样能更好的进行拓展。

![1616137359406](%E5%8F%AF%E6%8E%A7%E6%96%87%E6%9C%AC%E7%94%9F%E6%88%90survey/1616137359406.png)



![1616137354498](%E5%8F%AF%E6%8E%A7%E6%96%87%E6%9C%AC%E7%94%9F%E6%88%90survey/1616137354498.png)

### 2.3 **形式or风格限定的文本生成**  

  text + keywords（形式or风格限制）-> text

#### 2.3.1 文本形式限定

1. **关键词限定： 必须包含原词**
   下图，textrank 关键词提取。通过知识图谱或者类似知网的语言知识库，扩充词汇。
   ![1616137467028](%E5%8F%AF%E6%8E%A7%E6%96%87%E6%9C%AC%E7%94%9F%E6%88%90survey/1616137467028.png)
2. **感叹句疑问句 or 是否第一人称 or 文本长度**
   ![1616137481590](%E5%8F%AF%E6%8E%A7%E6%96%87%E6%9C%AC%E7%94%9F%E6%88%90survey/1616137481590.png)

#### 2.3.2 文本风格限定

1. **generic or specific**
   问题生成的任务，给定一段文本，生成这段文本中包含答案的问题。生成generic or specific的问题。

   ![1616137531732](%E5%8F%AF%E6%8E%A7%E6%96%87%E6%9C%AC%E7%94%9F%E6%88%90survey/1616137531732.png)
   适用于很多种产品，还是只针对这一个产品。
   明确的直率的 or 含蓄的委婉的：
   ![1616137538778](%E5%8F%AF%E6%8E%A7%E6%96%87%E6%9C%AC%E7%94%9F%E6%88%90survey/1616137538778.png)

2. **positive or negative**
   早期在情感控制生成这方面的研究，只局限于有限的分类，比如上面的五类，甚至是positive negative两类，近年细粒度的逐渐增多。

   ![1616137568531](%E5%8F%AF%E6%8E%A7%E6%96%87%E6%9C%AC%E7%94%9F%E6%88%90survey/1616137568531.png)

   先标注了一个情感分类器，标注每句对话，构造平行语料进行训练。为整首诗和诗里的每一句都进行细粒度的情感标记，提出了一个VAE模型。
   ![1616137573113](%E5%8F%AF%E6%8E%A7%E6%96%87%E6%9C%AC%E7%94%9F%E6%88%90survey/1616137573113.png)

   

3. **修辞（比喻、拟人）限制等**
   少有的现代诗歌的文章,通过对某几句进行控制，生成带有比喻和拟人的句子。任务是输入上下文，主题、每一句的标签（可以标记也可以预测），生成下一句诗。训练了一个句子分类器，为训练集的句子打上标签，将比喻和拟人的标签应用于修改后的模型中进行训练

<img src="%E5%8F%AF%E6%8E%A7%E6%96%87%E6%9C%AC%E7%94%9F%E6%88%90survey/1616137764169.png" alt="1616137764169" style="zoom:67%;" />

4. 文本简化（含义、长短、词句）
   任务是生成不同简化等级的句子。引入了词级别的loss，根据词频敲定一个词级别的loss。
   <img src="%E5%8F%AF%E6%8E%A7%E6%96%87%E6%9C%AC%E7%94%9F%E6%88%90survey/1616137797585.png" alt="1616137797585" style="zoom:50%;" />



## 3. 任务及方法分类

**模型方面**

* GAN
* seq2seq
* VAE
* RL
* 预训练生成模型，例如GPT2、CTRL等



文本控制方面

* 通过前向特征输入并且构造平行语料来对最终的文本进行控制
* 通过后向的调节或后处理的方式
  * 后向调节中，loss的设计
  * 后处理模式则直接对生成的结果进行beam_search采样和判断



分别举例：

2018 AAAI Emotional Chatting Machine: Emotional Conversation Generation with Internal and External Memory

通过训练了一个情感分类器，将对话内容打标签，数据驱动进行训练。模型结构加入了内部记忆和外部记忆模块，有助于情感文本的

![1616137915393](%E5%8F%AF%E6%8E%A7%E6%96%87%E6%9C%AC%E7%94%9F%E6%88%90survey/1616137915393.png)





2019 ACL Controllable Text Simplification with Lexical Constraint Loss

引入了词级别的loss，根据词频敲定一个词级别的loss，Grade体现在文档l上

![1616137945458](%E5%8F%AF%E6%8E%A7%E6%96%87%E6%9C%AC%E7%94%9F%E6%88%90survey/1616137945458.png)

![1616137974103](%E5%8F%AF%E6%8E%A7%E6%96%87%E6%9C%AC%E7%94%9F%E6%88%90survey/1616137974103.png)



•华为诺亚方舟 2019 GPT-based Generation for Classical Chinese Poetry

经过一个大力出奇迹的过程

![1616137988231](%E5%8F%AF%E6%8E%A7%E6%96%87%E6%9C%AC%E7%94%9F%E6%88%90survey/1616137988231.png)





## Reference

①Nishihara D, Kajiwara T, Arase Y. Controllable text simplification with lexical constraint loss[C]//Proceedings of the 57th Annual Meeting of the Association for Computational Linguistics: Student Research Workshop. 2019: 260-266.

②Keskar N S, McCann B, Varshney L R, et al. Ctrl: A conditional transformer language model for controllable generation[J]. arXiv preprint arXiv:1909.05858, 2019.

③Chen M, Tang Q, Wiseman S, et al. Controllable Paraphrase Generation with a Syntactic Exemplar[J]. arXiv preprint arXiv:1906.00565, 2019.

④Oraby S, Harrison V, Ebrahimi A, et al. Curate and generate: A corpus and method for joint control of semantics and style in neural nlg[J]. arXiv preprint arXiv:1906.01334, 2019.

⑤Cao Y T, Rao S, Daumé III H. Controlling the Specificity of Clarification Question Generation[C]//Proceedings of the 2019 Workshop on Widening NLP. 2019: 53-56.

⑥Liu Z, Fu Z, Cao J, et al. Rhetorically controlled encoder-decoder for modern chinese poetry generation[C]//Proceedings of the 57th Annual Meeting of the Association for Computational Linguistics. 2019: 1992-2001.

⑦Ke P, Guan J, Huang M, et al. Generating informative responses with controlled sentence function[C]//Proceedings of the 56th Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers). 2018: 1499-1508.

⑧Feng X, Liu M, Liu J, et al. Topic-to-Essay Generation with Neural Networks[C]//IJCAI. 2018: 4078-4084.

⑨Song Z, Zheng X, Liu L, et al. Generating Responses with a Specific Emotion in Dialog[C]//Proceedings of the 57th Annual Meeting of the Association for Computational Linguistics. 2019: 3685-3695.

⑩Zhou H, Huang M, Zhang T, et al. Emotional chatting machine: Emotional conversation generation with internal and external memory[C]//Thirty-Second AAAI Conference on Artificial Intelligence. 2018.

⑪Liao Y, Wang Y, Liu Q, et al. GPT-based Generation for Classical Chinese Poetry[J]. arXiv preprint arXiv:1907.00151, 2019.

⑫Radford A, Wu J, Child R, et al. Language models are unsupervised multitask learners[J]. OpenAI Blog, 2019, 1(8): 9.