---
title: 实体对齐survey
date: 2021-04-21
author: 长腿咚咚咚
toc: true
mathjax: true
categories: NLP进阶
tags:
	- 实体对齐survey
---



## 1 实体对齐

### 1.1 任务定义

实体对齐(Entity alignment) ，也称为实体匹配（Entity Matching），就是找到两个知识图谱中相同的等价实体。0



实体对齐任务进行定义：

* 用 $G=(E,R,A,T_R,T_A)$表示知识图谱，其中$E,R,A$表示其中的实体、关系和属性。 $T_R$是关系三元组 E−−R−−E 。$T_A$ 是属性三元组 E−−A−−V。
* 已知有知识图谱 G1 和 G2，目的是找到两个图谱中的等价实体集合$ S(ei1,ei2)$。



用处：

不同的知识库，收集知识的侧重点不同。知识融合的目的就是将不同知识图谱对实体的描述进行整合，从而获得实体的完整描述。



任务难点：

1. 计算复杂度（候选实体数量规模）
2. 数据质量的影响（需要算法对不同的数据情况制定合适的算法）
3. 算法的设计（如何结合实体信息对实体进行表示）



### 1.2 相关知识库与数据集

**知识库**

* FreeBase

  Freebase是个类似wikipedia的[创作共享](https://baike.baidu.com/item/创作共享/7570522)类网站，所有内容都由用户添加，采用创意共用许可证，可以自由引用。两者之间最大的不同在于，Freebase中的条目都采用[结构化数据](https://baike.baidu.com/item/结构化数据/5910594)的形式，而wikipedia不是。

* DBPedia： DBpedia 是一个很特殊的语义网应用范例，它从[维基百科](https://baike.baidu.com/item/维基百科/106382)(Wikipedia)的词条里撷取出结构化的资料，以强化维基百科的搜寻功能，并将其他资料集连结至维基百科

* 维基百科本体知识库

* Omega

* YAGO





* DBP15K（http://ws.nju.edu.cn/jape/）
  3种跨语言的实体对齐语料

![img](%E5%AE%9E%E4%BD%93%E5%AF%B9%E9%BD%90survey/dbp15k.PNG)



* DWY100K （https://github.com/nju-websoft/BootEA）

  2种大规模的数据集

![1619412168183](%E5%AE%9E%E4%BD%93%E5%AF%B9%E9%BD%90survey/1619412168183.png)





* DFB datasets （https://github.com/thunlp/IEAJKE）
   extracted from Freebase

  ![1619412850542](%E5%AE%9E%E4%BD%93%E5%AF%B9%E9%BD%90survey/1619412850542.png)

  

* Wk3l60k （《Co-training Embeddings of Knowledge Graphs and Entity Descriptions for Cross-lingual Entity Alignment》）

extracted from the subset of DBpedia

用于半监督跨语言学习

![1619413546413](%E5%AE%9E%E4%BD%93%E5%AF%B9%E9%BD%90survey/1619413546413.png)

![1619413081309](%E5%AE%9E%E4%BD%93%E5%AF%B9%E9%BD%90survey/1619413081309.png)





### 1.3 评价指标

Hits@N, 表示排名前 k 的候选中正确对齐的实体的比例。

MRR ： 是一个国际上通用的对[搜索算法](https://baike.baidu.com/item/搜索算法)进行评价的机制，即第一个结果匹配，分数为1，第二个匹配分数为0.5，第n个匹配分数为1/n，如果没有匹配的句子分数为0。最终的分数为所有得分之和



## 2. 模型or方法survey

有几个技术体系：

* TransE系列

简单来说，TransE就是讲知识图谱中的实体和关系看成两个Matrix。实体矩阵结构为 ![[公式]](%E5%AE%9E%E4%BD%93%E5%AF%B9%E9%BD%90survey/equation-1619082537319.svg) ，其中n表示实体数量，d表示每个实体向量的维度，矩阵中的每一行代表了一个实体的词向量；而关系矩阵结构为 ![[公式]](%E5%AE%9E%E4%BD%93%E5%AF%B9%E9%BD%90survey/equation.svg) ，其中r代表关系数量，d表示每个关系向量的维度。TransE训练后模型的理想状态是，从实体矩阵和关系矩阵中各自抽取一个向量，进行L1或者L2运算，得到的结果近似于实体矩阵中的另一个实体的向量，从而达到通过词向量表示知识图谱中已存在的三元组 ![[公式]](https://www.zhihu.com/equation?tex=%EF%BC%88h%2C+l%2C+t%EF%BC%89) 的关系。

<img src="%E5%AE%9E%E4%BD%93%E5%AF%B9%E9%BD%90survey/1619082650523.png" alt="1619082650523" style="zoom:50%;" />

* TransE + GCN 系列

联合 知识嵌入模型  和  interaction图模型的结合体



* TransE + 对抗网络系列



* GCN变种



* 融合图结构信息与边信息的多维度方法



* 预训练模型的方法

BERT-INT 《 A BERT-based Interaction Model For Knowledge Graph Alignment》



## Reference

http://pelhans.com/2019/03/18/entity_alignment/

https://www.ijcai.org/proceedings/2018/0556.pdf

https://www.ijcai.org/proceedings/2019/0754.pdf

https://www.ijcai.org/proceedings/2020/0506.pdf

https://www.ijcai.org/proceedings/2019/0448.pdf

https://www.ijcai.org/proceedings/2018/0611.pdf

https://www.ijcai.org/proceedings/2017/0595.pdf

https://www.ijcai.org/proceedings/2019/0733.pdf

https://www.ijcai.org/proceedings/2019/0929.pdf

https://www.ijcai.org/proceedings/2019/0754.pdf

https://www.ijcai.org/proceedings/2018/0556.pdf

















