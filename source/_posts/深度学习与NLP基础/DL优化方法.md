---
title: DL优化方法
date: 2019-12-07
author: 长腿咚咚咚
toc: true
mathjax: true
categories: 深度学习与NLP基础
tags:
	- DL优化方法
---



[TOC]



## 1 防止欠拟合、过拟合的方法

- **欠拟合：**模型没有充分学习到数据中的特征信息，无法很好的拟合数据，在训练集和测试集上效果都不好。
- **过拟合：**模型过于复杂，将噪声数据的特征也学习到模型中了，泛化能力下降，导致模型的训练误差远小于测试误差。

![2019-8-18_21-7-0](DL%E4%BC%98%E5%8C%96%E6%96%B9%E6%B3%95/2019-8-18_21-7-0.png)

### 1.1 降低欠拟合的风险

1. 添加新特征
   避免特征不足or特征与样本标签的相关性不强时无法拟合
2. 增加模型复杂度
   提高模型的学习能力
3. 减少正则化系数

### 1.2 降低过拟合风险

1. 从数据入手，获得更多的训练数据

2. 降低模型复杂度

3. 正则化
   在模型损失函数中添加惩罚项，使学出的模型参数值较小，避免权值过大带来的过拟合风险。

4. 集成学习方法（Dropout、交叉检验等）
   降低单一模型的过拟合风险

5. 训练的调优：Early stoping

   当验证集的误差连续 `n`次递增时停止训练。



## 2. 深度学习技巧

### 2.1 Dropout

* Dropout的思想是：在每次训练过程中随机地忽略一些神经元。
* 为什么Dropout能解决过拟合
  整个dropout过程就相当于： 对很多个不同的神经网络取平均，相当于Bagging的一种近似

* dropout是怎么实现的

  * 在训练时，每个神经单元以概率pp被保留(Dropout丢弃率为1−p)；
  * 在预测阶段（测试阶段），每个神经单元都是存在的，权重参数w要乘以p，输出是：pw
  * 为了保持同样的输出期望值并使下一层也得到同样的结果

  ```
  import numpy as np
  p = 0.5  # 神经元激活概率
  def train_step(X):
      """ X contains the data """
      # 三层神经网络前向传播为例
      H1 = np.maximum(0, np.dot(W1, X) + b1)
      U1 = np.random.rand(*H1.shape) < p   # first dropout mask
      H1 *= U1 # drop!
      H2 = np.maximum(0, np.dot(W2, H1) + b2)
      U2 = np.random.rand(*H2.shape) < p # second dropout mask
      H2 *= U2 # drop!
      out = np.dot(W3, H2) + b3
  def predict(X):
      # ensembled forward pass
      H1 = np.maximum(0, np.dot(W1, X) + b1) * p    # NOTE: scale the activations
      H2 = np.maximum(0, np.dot(W2, H1) + b2) * p   # NOTE: scale the activations
      out = np.dot(W3, H2) + b3
  ```

  

### 2.2 正则化

#### L1正则化

L1正则化项：向量的1范数
$$
正则化项： \Omega(\theta) = ||w||_1 =  \sum_i |w_i| \\
目标函数： \tilde{J}(w;X,y) = \alpha ||w||_1  + J(w;X,y)  \\
梯度： \nabla_w \tilde{J}(w;X,y) = \alpha sign(w) + \nabla_w J(w;X,y) \\
$$
L1 正则化使得权重值可能被减少到0。 因此，L1对于压缩模型很有用。



#### L2正则化

L2正则化项：向量的2范数（元素平方和相加再开放
$$
正则化项： \Omega(\theta) = \frac{1}{2} ||w||_2^2  = \frac{1}{2}w^Tw \\
目标函数： \tilde{J}(w;X,y) = \frac{\alpha}{2}w^Tw  + J(w;X,y)  \\
梯度： \nabla_w \tilde{J}(w;X,y) = \alpha w + \nabla_w J(w;X,y) \\
梯度更新 ： w \leftarrow (1- \epsilon \alpha) w - \epsilon \nabla_w J(w;X,y)
$$
L2正则化又称权重衰减。因为其导致权重**趋向于0**（但不全是0）。



#### 关于正则化的一些问题

* 为什么只对权重做惩罚，不包含偏置:
  精确拟合偏置所需的数据通常比拟合权重少得多，因此偏置拟合程度较好。
  正则化偏置参数可能会导致明显的欠拟合。

* **L1正则化与L2正则化的异同**
  相同点：限制参数的规模，使模型偏好于权值较小的目标函数，防止过拟合。
  不同点：

  1. L1 正则化可以产生**稀疏权值矩阵**，可以用于特征选择；L2 趋向于生成参数值很小的矩阵
  2. L1 适用于特征之间有关联的情况； L2 适用于特征之间没有关联的情况

* 为什么L1正则化可以产生稀疏解，而L2不会？
  
  产生稀疏解，其实就是目标函数求导后 w=0时得到目标函数的极小值
  
  形式化一下分别加了L1和L2正则化的目标函数：
  $$
  \begin{gathered}
  J_{L 1}(w)=L(w)+\lambda|w| \\
  J_{L 2}(w)=L(w)+\lambda w^{2}
  \end{gathered}
  $$
  假设在w=0处，其L(w)的导数为d0，那么L2正则化的导数在w=0处和以前一样，没啥变化，不会导致稀疏解

$$
\begin{gathered}
\left.\frac{\partial J_{L 2}(w)}{\partial w}\right|_{w=0}=d_{0}+2 \times \lambda \times w=d_{0} 
\end{gathered}
$$

​		但是L1正则化的导数不一样，在w=0的两边取极限，发现两边的导数可能异号，导数先小于0后大于0，所以w=0处出现极小值，导致稀疏解：


$$
\begin{gathered}
\left.\frac{\partial J_{L 1}(w)}{\partial w}\right|_{w=0^{-}}=d_{0}-\lambda \\
\left.\frac{\partial J_{L 1}(w)}{\partial w}\right|_{w=0^{+}}=d_{0}+\lambda
\end{gathered}
$$

* 为什么L1和L2正则化可以防止过拟合
  L1 & L2 正则化会使模型偏好于更小的权值，更小的权值意味着更低的模型复杂度，有助于提高模型的泛化能力。





## 3 梯度下降方法

### 2.1 基础梯度下降法

* 标准梯度下降
  $$
  \theta = \theta + \eta \cdot  \nabla_\theta J(\theta)
  $$

* mini-batch 梯度下降
  $$
  \theta = \theta + \eta \cdot  \nabla_\theta J(\theta; x^{(i:i+n)}; y^{(i:i+n)})
  $$
  
* 随机梯度下降
  $$
  \theta = \theta + \eta \cdot  \nabla_\theta J(\theta; x^{(i)}; y^{(i)}) \,\,\, \eta 为学习率
  $$
  
* 优缺点：
  1. 标准梯度下降需要计算所有样本梯度，更新速度慢；梯度方向一致且更新次数少，容易陷入局部最小值
  2. 随机梯度下降是mini-batch的特殊，不能达到最优解，不容易陷入局部最小

### 2.2 动量梯度下降法

* 梯度的指数加权平均值$v_t$:
  $$
  v_t = \gamma v_{t-1} + \eta \nabla_\theta J(\theta)  \\ \theta = \theta - v_t \\
  \gamma: \text{加权系数，常取值为0.9}
  $$

* 使用指数加权平均值来更新梯度：本质是通过增加动量来减少随机，增加梯度的稳定性。

* 其中的指数加权平均值本质上考虑了前几次的梯度，不至于在”下坡“过程中因为遇到暂时的”上坡“而改变方向

### 2.3  自适应学习率优化算法

* Adagrad
* Adadelta
* RMSprop
* Adam

ps：看到这里脑壳痛，先放放









## 4 归一化方法

### 4.1 ICS问题

独立同分布（independent and identically distributed），独立同分布的数据可以简化常规机器学习模型的训练、提高模型的能力。

内部协变量偏移（Internel Covariate Shift, ICS）问题

深度神经网络的训练过程中，每一层的参数更新会导致上层的输入数据分布发生变化，上层参数需要不断适应新的输入数据分布，会降低学习速度。



### 4.2 Batch Normalization

动机：针对单个神经元，利用网络训练时一个 mini-batch 的数据来计算该神经元 $x_i$ 的均值和方差，进行归一化。



假设一个 batch 为 m 个输入 $B = \{x_{1}, \cdots, x_m\}$ , Batch Normalization在这 m 个数据之间做归一化， 并学习参数 $\gamma , \beta$：
$$
\mu_B \leftarrow \frac{1}{m} \sum_{i=1}^{m} x_i    \\
\sigma_B^2 \leftarrow \frac{1}{m} \sum_{i=1}^{m} (x_i - \mu_B)^2  \\
\hat{x}_i  \leftarrow \frac{x_i - \mu_B}{\sqrt{\sigma_B^2 + \epsilon}} \\
y_i \leftarrow \gamma \hat{x}_i + \beta \equiv BN_{\gamma, \beta}{(x_i)}
$$

对Batch Normalization的优点进行总结，大概有以下几个方面：

- 让模型更稳定，可以采用更高的学习率，提高模型训练的速度，并且可以有效避免梯度的消失和爆炸
- 起到正则化的作用，类似dropout的功能
- 不用太在意参数的初始化

Batch Normalization 在推断时如何设置方差和期望：

* 不用验证样本的均值和方差，而是利用训练集统计的均值和方差，统计方式一般包括移动平均和全局平均。

有哪些问题

* 依赖 batch_size
* 无法处理序列化数据
* 推断时无法使用

### 4.3 Layer Normalization

不同于 BN， 其在层内进行 Normalization。即直接对隐层单元的输出做 Normalization。
$$
u^l = \frac{1}{H} \sum_{i=1}^H a_i^l \\
\sigma^l = \sqrt{\frac{1}{H}\sum_{i=1}^H(a_i^l - u^l)^2} \\
\hat{a}_i^l  \leftarrow \frac{a_i^l - \mu^l}{\sqrt{\sigma_l^2 + \epsilon}} \\
y_i^l \leftarrow \gamma \, \hat{a}_i^l + \beta \equiv LN_{\gamma, \beta}{(a_i^l)}
$$

**与Batch Normalization 区别**

Batch 顾名思义是对一个batch进行操作。假设我们有 10行 3列 的数据，即我们的batchsize = 10，每一行数据有三个特征，假设这三个特征是【身高、体重、年龄】。那么BN是针对每一列（特征）进行缩放，例如算出【身高】的均值与方差，再对身高这一列的10个数据进行缩放。体重和年龄同理。这是一种“列缩放”。

而layer方向相反，它针对的是每一行进行缩放。即只看一笔数据，算出这笔所有特征的均值与方差再缩放。这是一种“行缩放”。

**BN是取不同样本的同一个通道的特征做归一化；LN取的是同一个样本的不同通道做归一化**

```
一个batch的数据：[[1,2],[3,4],[5,6]]
Batch Normalization:
    index 1: 1+3+5 = 9
    index 2： 2+4+6 = 12
    [[1/9,2/12],[3/9,4/12],[5/9,6/12]]
Layer Normalization:
    [[1/3,2/3],[3/7,4/7],[5/11,6/11]]

```











## reference：

```
《统计学习方法》
https://github.com/NLP-LOVE/ML-NLP
https://github.com/km1994/NLP-Interview-Notes/
https://zhuanlan.zhihu.com/p/54530247
https://zhuanlan.zhihu.com/p/74516930
https://blog.csdn.net/b876144622/article/details/81276818
https://www.zhihu.com/question/37096933
https://www.cnblogs.com/zingp/p/11631913.html
https://www.cnblogs.com/mengxiangtiankongfenwailan/p/9989065.html
```









