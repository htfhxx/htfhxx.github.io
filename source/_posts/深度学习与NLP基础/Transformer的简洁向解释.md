---
title: Transformer的简洁向解释
date: 2019-11-28
author: 长腿咚咚咚
toc: true
mathjax: true
top: true
categories: 深度学习与NLP基础
tags:
	- Transformer的简洁向解释
---



[TOC]

<img src="Transformer%E7%9A%84%E7%AE%80%E6%B4%81%E5%90%91%E8%A7%A3%E9%87%8A/1574773800747.png" alt="1574773800747" style="zoom:67%;" />

## 1 主要结构

​	    **1. 输入：Input Embedding——$(x_1,x_2,x_3,...,x_n)$ 进行Positional Encoding，投入Encoder；**

​		**2. Encoder：编码处理后输出——$(z_1,z_2,z_3,...,z_n)$，并将其作为Decoder的输入；**

​		**3. Decoder：进行解码处理；**

​		**4. 输出：最终对Decoder的输出进行处理最终的概率$(y_1,y_2,y_3,...,y_m)$**

​		**注意！**Encoder和Decoder都是并行计算的N（论文取N=6）个相同结构的堆叠。

## 2 Encoder

### 2.1 Encoder整体

<img src="Transformer%E7%9A%84%E7%AE%80%E6%B4%81%E5%90%91%E8%A7%A3%E9%87%8A/1574779976637.png" alt="1574779976637" style="zoom:33%;" />

​		Encoder部分有两个sub-layer，**Multi-Head Attention**和**Feed Forward**。上图的Add代表残差网络，由图可知在每个sub-layer都加入了残差项。其中的Norm是Layer Normalization。

​		Multi-Head Attention是本论文的核心，主要是self Attention；Feed Forward是一个简单的全连接前馈神经网络。

### 2.2 Encoder的输入
​		Encoder端每个大模块接收的输入是不一样的，第一个大模块(最底下的那个)接收的输入是输入序列的embedding，其余大模块接收的是其前一个大模块的输出，最后一个模块的输出作为整个Encoder端的输出。  

### 2.3 Multi-Head Attention

<img src="Transformer%E7%9A%84%E7%AE%80%E6%B4%81%E5%90%91%E8%A7%A3%E9%87%8A/1574775307731.png" alt="1574775307731" style="zoom: 33%;" />

​		

​		对于self-attention来讲，Q(Query), K(Key), V(Value)三个矩阵均来自同一输入。单个Multi-Head Attention层的输入进行处理得到QKV，通过线性变换输入到Scaled Dot-Product Attention，得到多组结果进行concat并加权后，作为Encoding后的结果。（QKV是什么见下一小节）

​		整个过程的公式：
$$
\begin{aligned} \text { MultiHead }(Q, K, V) &\left.=\text { Concat (head }_{1}, \ldots, \text { head }_{\mathrm{h}}\right) W^{O} \\ \text { where head }_{\mathrm{i}} &=\text { Attention }\left(Q W_{i}^{Q}, K W_{i}^{K}, V W_{i}^{V}\right) \end{aligned}
$$
​		

### 2.4 Scaled Dot-Product Attention

<img src="Transformer%E7%9A%84%E7%AE%80%E6%B4%81%E5%90%91%E8%A7%A3%E9%87%8A/1574777556699.png" alt="1574777556699" style="zoom:33%;" />

​		Scaled Dot-Product Attention是Transformer较为重要的核心部分，整个过程：
$$
\text { Attention }(Q, K, V)=\operatorname{softmax}\left(\frac{Q K^{T}}{\sqrt{d_{k}}}\right) V
$$

1. Q和K计算相似度，此处使用点积的方法；
2. 防止结果过大，除以${\sqrt{d_{k}}}$，其中${d_{k}}$是K的维度；（内积太大的话softmax后就非0即1了,不够“soft”了）（为什么除以${\sqrt{d_{k}}}$: 让qk的方差与dk无关，从${d_{k}}$变为1，参考：https://blog.csdn.net/qq_37430422/article/details/105042303）
3. 经过一个Mask操作； Q，K长度是不定时，进行补齐操作，将补齐的数据设置为负无穷
4. 进行softmax归一化得到Q和K的Attention；
5. Attention与V相乘，得到self-Attention的结果。

​		这里的QKV论文里没有详细展开，现有的博客文章很少提到，本文引用的第二篇博客中有段话写的很好。Attention机制中，将Source中看做是由一系列的(Key,Value)对构成，此时给定某元素Query，计算Query和各个Key的相关性，得到每个Key对应Value的权重系数，然后对Value进行加权求和，即得到了最终的Attention数值。所以本质上Attention机制是对Source中元素的Value值进行加权求和，而Query和Key用来计算对应Value的权重系数。

<img src="Transformer%E7%9A%84%E7%AE%80%E6%B4%81%E5%90%91%E8%A7%A3%E9%87%8A/att7.png" alt="image" style="zoom:33%;" />

​		而对于本文的self-Attention来说，key、value和query都是其本身，也就是上一层的输出，作为下一层的输入。

## 3 Decoder

​		Decoder部分有三个sub-layer，**Masked Multi-head Attention**、**Encoder-Decoder Attention**和**Feed Forward**。上图的Add和Norm与Encoder部分一致，分别代表残差网络和Layer Normalization。

​		decoder相对encoder，有两个不同的地方，一是第一级的Masked Multi-head，二是第二级中Multi-Head Attention的输入。

<img src="Transformer%E7%9A%84%E7%AE%80%E6%B4%81%E5%90%91%E8%A7%A3%E9%87%8A/1574780008243.png" alt="1574780008243" style="zoom:33%;" />



​	

### 3.1 Masked Multi-head Attention

​		Masked Multi-head是decoder的第一级decoder，其key, query, value均来自前一层decoder的输出。

​		**但其加入了Mask操作，因为翻译过程我们当前还并不知道下一个输出哪个词语。**

​		训练过程中，因为在实现中无法每次动态的输入，就一次性把目标序列通通输入第一个大模块中，然后在Multi-head Attention中对序列进行mask即可。

​		在测试过程中，先生成第一个位置的输出，第二次预测时，再将其加入输入序列，以此类推直至预测结束。

### 3.2 Encoder-Decoder attention Attention

​		Decoder中第二级decoder是Multi-Head Attention，论文中也称为"encoder-decoder attention"。它不仅接受来自前一级的输出，还接收encoder的输出。

​		它的**query来自于之前一级的decoder层的输出**，但其**key和value来自于encoder的输出**，这使得decoder的每一个位置都可以attend到输入序列的每一个位置。

​		总结一下，k和v的来源总是相同的，Q在encoder及第一级decoder中与K，V来源相同，在encoder-decoder attention layer中与K,V来源不同。

## 4 其他细节

### 4.1 Position-wise Feed-Forward Networks

​		除了Attention子层之外，其他子层都包含了一个全连接的前馈网络。包括两个线性变换，中间有一个ReLU：
$$
\mathrm{FFN}(x)=\max \left(0, x W_{1}+b_{1}\right) W_{2}+b_{2}
$$

### 4.2 Positional Encoding

​		模型结构本质上是忽视了数据的序列信息，因此为了充分利用序列信息，将tokens的相对和绝对位置编码进输入Embedding。

​		论文中使用sine and cosine的方式计算出来（称其效果与训练出来的比较接近），且结果与embedding维度相同。然后将Positional Encoding与embedding 进行加和。

$$
\begin{array}{c}{P E_{(p o s, 2 i)}=\sin \left(\text {pos} / 10000^{2 i / d_{\text {madel }}}\right)} \\ {P E_{(\text {pos}, 2 i+1)}=\cos \left(\text {pos} / 10000^{2 i / d_{\text {madit }}}\right)}\end{array}
$$

## 5 常见问题总结

- Transformer的结构是什么样的？
  Transformer本身还是一个典型的encoder-decoder模型：
  1. Encoder由N(原论文中**N=6**)个相同的大模块堆叠而成，其中每个大模块又由**两个sub-layer**构成，这两个子模块分别为Multi-Head Attention模块，以及一个前馈神经网络模块；
  2. Decoder端同样由N个相同的大模块堆叠而成，其中每个大模块则由**三个sub-layer**构成，这三个子模块分别为Masked Multi-Head Attention模块，本质也是Multi-Head Attention的encoder-decoder attention模块，以及一个前馈神经网络模块；  
  3. 在所有sub-layer中，都附加了残差网络和layer normalization。
- Multi-Head Attention的具体结构？
  Multi-Head Attention的结果是QKV进行不同的变换后，再进行self-Attention后进行concat得到的。
- self-Attention具体内容？
  Q和K计算相似度，此处使用点积的方法；防止结果过大，除以${\sqrt{d_{k}}}$，其中${\sqrt{d_{k}}}$是K的维度；进行softmax归一化得到Q和K的Attention；Attention与V相乘，得到self-Attention的结果。
- Self-Attention为什么work？
  self-attention的特点在于**无视词(token)之间的距离直接计算依赖关系，从而能够学习到序列的内部结构**
- Transformer相比于RNN/LSTM，有什么优势？为什么？
  RNN由于序列依赖的原因，并行计算能力比较差；Transformer的特征提取能力比RNN系列强。
- Transformer与Seq2Seq相比？
  seq2seq最大的问题在于**将Encoder端的所有信息压缩到一个固定长度的向量中**，并将其进行解码。在输入信息较长时，会损失大量的信息。而且这样整体交给Decoder进行编码，Decoder很难关注到真正要关注的信息。
- Transformer是如何加入词序信息的？
  模型结构本质上是忽视了数据的序列信息，因此为了充分利用序列信息，将tokens的相对和绝对位置编码进输入Embedding。论文中使用sine and cosine的方式计算出来，且结果与embedding维度相同。然后将Positional Encoding与embedding 进行加和。
- 为什么transformer块使用LayerNorm而不是BatchNorm？
  从LayerNorm的优点来看，它对于batch大小是健壮的，并且在样本级别而不是batch级别工作得更好。
- 

## 6 参考文章

https://arxiv.org/abs/1706.03762

https://zhuanlan.zhihu.com/p/47063917

[从Encoder-Decoder(Seq2Seq)理解Attention的本质](https://www.cnblogs.com/huangyc/p/10409626.html)

https://www.nowcoder.com/discuss/258321?type=0&order=0&pos=19&page=1

