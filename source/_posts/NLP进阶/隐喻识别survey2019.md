---
title: 隐喻识别survey2019
date: 2019-04-30 21:37:00
author: 长腿咚咚咚
toc: true
mathjax: true
categories: NLP进阶
tags:
	- 隐喻识别survey2019
---

[TOC]

## 1. 隐喻（Metaphor）

隐喻不仅是人类语言中常用的一种修辞手法，更代表着人类对世界的认识和理解方式。

目前查到的最贴切的一个解释：隐喻就是人类利用在某一个领域的理解和经历，去解释另一个领域的认知行为。

隐喻识别属于隐喻计算的一个子任务。隐喻计算可以分为隐喻识别、隐喻理解和隐喻生成三个子任务。

隐喻理解的子任务举例：CCL2018 中文隐喻的情感分析(隐喻句子的情感分类)




## 2. 隐喻识别的意义
自然语言处理研究领域中,隐喻是一个不可回避的问题.

一些研究表明,**中文和英文的语料中存在着大量隐喻表达**.      因此,机器翻译、文本处理、信息检索等若局限于获取字面意义,而无法理解隐喻等“言外之意”是远远不够的.

若能在这些领域中引入隐喻计算系统,则有希望进一步提高机器翻译质量,改善搜索引擎反馈,提高人工智能系统的水平等等.


## 3. 隐喻的分类（根据语言表达）
### 3.1 名词性隐喻
用一个名词去形容另一个名词

由指称词“是”“像”“如”等连接： 知识就是力量。    人生如梦
用标点符号“,” “—”等连接：  书 ,人类进步的阶梯


### 3.2 动词性隐喻
动作（谓语）的实施者（主语）或者承受者（宾语）不符合人类对该动作的正常的搭配认知。

例如： <狗，吃，良心>


### 3.3 形容词性隐喻
表现为某个名词拥有了本身所不具备的（形容词）属性，即某个实体本身的属性与形容词所描述的没有关联。

如：愤怒的大海

### 3.4 副词性隐喻
动作因为某种属性被描述成另一种具备类似属性的动作

如：他们的头点的跟小鸡啄米似的。（用小鸡啄米形容点头快）

### 3.5 常规隐喻 - 死喻
用太多成为常规搭配

如：尖锐的声音

## 4. 隐喻识别的方法

### 对句子的分类 or 序列标注任务识别隐喻词汇

### 基于知识库 or 语义规则的方法
常见的知识库有：Hownet（董振东、董强的知网）、Wordnet（英语词汇数据库）、同义词词林、vernet等

知网：以汉语和英语的词语所代表的概念为描述对象，以揭示概念与概念之间以及概念所具有的属性之间的关系为基本内容的常识知识库。 （http://www.keenage.com/zhiwang/c_zhiwang.html）
描述了上下位、同义、反义等各种关系。


偏语言学的角度，进行依存分析抽取文本语义关系，建立语言模型
- 与人工定义的语义规则是否匹配 --> 识别动词的常规表达、转喻、隐喻和异常表达;
- 名词间的上下位关系来识别名词性隐喻;       (语言学概念--概括性较强的单词叫做特定性较强的单词的上位词)
- 词语在语料库中的共现频率来识别动词和形容词性隐喻;
- 使用诗歌作为语料进行隐喻识别
- .......

最高精度为75%  --  2018 ACL Word Embedding and WordNet Based Metaphor Identification and

### 基于统计的方法
主要思想是利用大规模的语料库进行统计和分析，再对具体的隐喻问题进行分类和识别。

隐喻识别被看作一个分类问题

使用概念的不同特征训练隐喻识别的分类器 (概念的属性特征和语义特征)

例如：
- 使用属性特征为概念进行语义建模进而训练可解释的隐喻识别分类器
- 对数据库中的概念进行属性特征抽取,使用回归方法进行隐喻识别
- ......


### 神经网络的方法

2018 EMNLP Neural Metaphor Detection in Context  使用Bi-LSTM 识别隐喻词汇和句子分类

最高精度 2018 EMNLP 

句子分类：MOH-X 79.1%
序列标注：MOH-X 75.6%




## 5 隐喻数据
1. CCL2018  5000条人工标注隐喻数据用于评测(已过期，暂时未找到)
2. 隐喻语料资源/知识库(英文)(http://www.jos.org.cn/html/2015/1/4669.htm#top)

Master Metaphor List    基于源域和目标域映射的在线知识库

Sense-Frame       词例化隐喻知识库

MetaBank、ATT-Meta、Metalude、Hamburg Metaphor Database、ItalWordNet等隐喻知识库

3. 目前最大的隐喻数据集（链接已过期）

Introducing the LCC Metaphor Datasets
https://pdfs.semanticscholar.org/edf9/b7367660d7f8255633717bf050f000a7c8fd.pdf




4. Mohammad等人(2016)开发的数据集。(http://saifmohammad.com/WebPages/metaphor.html)


该数据集包含1230个普通文本和409个隐喻句子

2018 ACL Word Embedding and WordNet Based Metaphor Identification and Interpretation:  75%的精度


5. TroFi and MOH\MOH-X（已获得）https://github.com/htfhxx/metaphor-in-context

句子的分类

TroFi (Birke and Sarkar, 2006) 

MOH (Mohammad et al., 2016)

MOH-X refers to a subset of MOH dataset used in previous work (Shutova et al., 2016)


6. VUA
隐喻词的检测（序列标注）
VUA (Steen et al., 2010)
verb + sentence + verb_index + sentence_lable

7. 动词的隐喻分类数据集
https://github.com/EducationalTestingService/metaphor

2016 ACL Semantic classifications for detection of verb metaphors


8. 中文情感词汇本体库

词语+情感信息（情感分类，强度，极性等）

2018 ACL Construction of a Chinese Corpus for the Analysis of the Emotionality of Metaphorical Expressions  大工-林鸿飞 

9. 其他

隐喻的新颖分数

2018 AAAI Exploring the Terrain of Metaphor Novelty: A Regression-Based Approach for Automatically Scoring Metaphors

对隐喻的新颖性进行打分

对隐喻新颖性起作用的语言和概念特征



















