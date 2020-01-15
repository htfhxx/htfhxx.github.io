---
title: DL优化方法
date: 2019-12-07
author: 长腿咚咚咚
toc: true
mathjax: true
categories: deep-learning
tags:
	- DL优化方法
---



[TOC]



## 1 防止欠拟合、过拟合的方法

- **欠拟合：**模型⽆法得到较低的训练误差。
- **过拟合：**是模型的训练误差远小于它在测试数据集上的误差。

<img src="DL%E4%BC%98%E5%8C%96%E6%96%B9%E6%B3%95/2019-8-18_21-7-0.png" style="zoom:50%;" />

### 1.1 增大训练集规模

### 1.2 Dropout

* Dropout的思想是：在每次训练过程中随机地忽略一些神经元。
* 为什么Dropout能解决过拟合
  整个dropout过程就相当于： 对很多个不同的神经网络取平均，相当于Bagging的一种近似

### 1.3 正则化

在模型损失函数中添加惩罚项，使学出的模型参数值较小，是应对过拟合的常⽤⼿段。

#### 1.3.1 L1正则化

L1正则化项：向量的1范数
$$
正则化项： \Omega(\theta) = ||w||_1 =  \sum_i |w_i| \\
目标函数： \tilde{J}(w;X,y) = \alpha ||w||_1  + J(w;X,y)  \\
梯度： \nabla_w \tilde{J}(w;X,y) = \alpha sign(w) + \nabla_w J(w;X,y) \\
$$
L1 正则化使得权重值可能被减少到0。 因此，L1对于压缩模型很有用。



#### 1.3.2 L2正则化

L2正则化项：向量的2范数（元素平方和相加再开放
$$
正则化项： \Omega(\theta) = \frac{1}{2} ||w||_2^2  = \frac{1}{2}w^Tw \\
目标函数： \tilde{J}(w;X,y) = \frac{\alpha}{2}w^Tw  + J(w;X,y)  \\
梯度： \nabla_w \tilde{J}(w;X,y) = \alpha w + \nabla_w J(w;X,y) \\
梯度更新 ： w \leftarrow (1- \epsilon \alpha) w - \epsilon \nabla_w J(w;X,y)
$$
L2正则化又称权重衰减。因为其导致权重**趋向于0**（但不全是0）。



#### 1.3.2 关于正则化的一些问题

* 只对权重做惩罚，不包含偏置
  精确拟合偏置所需的数据通常比拟合权重少得多，因此偏置拟合程度较好。
  且正则化偏置参数可能会导致明显的欠拟合。

* L1正则化与L2正则化的异同
  相同点：限制参数的规模，使模型偏好于权值较小的目标函数，防止过拟合。
  不同点：

  1. L1 正则化可以产生**稀疏权值矩阵**，可以用于特征选择；L2 趋向于生成参数值很小的矩阵
  2. L1 适用于特征之间有关联的情况； L2 适用于特征之间没有关联的情况

* 为什么L1正则化可以产生稀疏值，而L2不会？
  

  ![L2](DL%E4%BC%98%E5%8C%96%E6%96%B9%E6%B3%95/L2.png)![L1](DL%E4%BC%98%E5%8C%96%E6%96%B9%E6%B3%95/L1.png)

  如下图所示：加入的正则化惩罚项可以看成是对原损失函数的约束，尽可能使惩罚项最小。

  彩色的圈是原损失函数的梯度线，在同一个圈上的梯度相同。
  可以得知，选择L1惩罚项时，容易出现很多的“尖”，在这些“尖”上L1惩罚项最小，导致了部分参数为0

* 为什么L1和L2正则化可以防止过拟合
  L1 & L2 正则化会使模型偏好于更小的权值，更小的权值意味着更低的模型复杂度，有助于提高模型的泛化能力。

### 1.4 Early stoping

提前终止指的是**当验证集的误差连续 `n`次递增时停止训练**。



## 2 梯度下降方法

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









## 3 归一化方法

### 3.1 普通的数据归一化方法

* Min-max 归一化：
  $$
  x^* = \frac{x -min } {max - min}
  $$

* Zero-mean 归一化：均值为0，标准差为1的标准正态分布
  $$
  x^* = \frac{x- \mu }{\sigma}
  $$

### 3.2 Batch Normalization

假设一个 batch 为 m 个输入 $B = \{x_{1}, \cdots, x_m\}$ , Batch Normalization在这 m 个数据之间做归一化， 并学习参数 $\gamma , \beta$：
$$
\mu_B \leftarrow \frac{1}{m} \sum_{i=1}^{m} x_i    \\
\sigma_B^2 \leftarrow \frac{1}{m} \sum_{i=1}^{m} (x_i - \mu_B)^2  \\
\hat{x}_i  \leftarrow \frac{x_i - \mu_B}{\sqrt{\sigma_B^2 + \epsilon}} \\
y_i \leftarrow \gamma \hat{x}_i + \beta \equiv BN_{\gamma, \beta}{(x_i)}
$$

### 3.3 Layer Normalization

不同于 BN， 其在层内进行 Normalization。即直接对隐层单元的输出做 Normalization。
$$
u^l = \frac{1}{H} \sum_{i=1}^H a_i^l \\
\sigma^l = \sqrt{\frac{1}{H}\sum_{i=1}^H(a_i^l - u^l)^2} \\
\hat{a}_i^l  \leftarrow \frac{a_i^l - \mu^l}{\sqrt{\sigma_l^2 + \epsilon}} \\
y_i^l \leftarrow \gamma \, \hat{a}_i^l + \beta \equiv LN_{\gamma, \beta}{(a_i^l)}
$$





reference：

```
《统计学习方法》
https://github.com/NLP-LOVE/ML-NLP
https://github.com/htfhxx/NLPer-Interview
```










