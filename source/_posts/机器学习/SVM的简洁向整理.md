---
title: SVM
date: 2019-11-28
author: 长腿咚咚咚
toc: true
mathjax: true
top: true
categories: 机器学习
tags:
	- SVM
---

# SVM

[TOC]

## 1 整体概览

* 什么是SVM
  SVM 是一种二分类模型。它的基本思想是在特征空间中寻找间隔最大的分离超平面使数据二分类

- 线性可分支持向量机——硬间隔支持向量机
- 线性支持向量机——软间隔支持向量机
- 非线性支持向量机——核技巧

## 2 线性可分支持向量机

### 2.1 函数间隔与几何间隔

- 分离超平面：$w^{*} \cdot x + b^{*}=0$
- 点到超平面的距离：正的一侧：$\gamma_{i}=\frac{1}{\|w\|}(w \cdot x_i +b) $      负的一侧：$\gamma_{i}=-\frac{1}{\|w\|}(w \cdot x_i +b) $      
- 由于$w \cdot x+b$与类标记y的符号是否一致能代表分类正确，定义：
  - 超平面对于**样本点**$(x_i,y_i)$的**函数间隔**：$\hat{\gamma}_{i}=y_{i}\left(w \cdot x_{i}+b\right)$
  - 超平面对于**数据集**的函数间隔：$\hat{\gamma}=\min  \hat{\gamma}_{i}$
  - 超平面对于**样本点**$(x_i,y_i)$的**几何间隔**：$\gamma_{i}=y_i\frac{1}{\|w\|}\left(w \cdot x_{i}+b\right)$
  - 超平面对于**数据集**的几何间隔：$\gamma=min{ \gamma_{i}}$

### 2.2 间隔最大化

- 最大间隔分离超平面可以表示为约束优化问题：

  - 最大间隔求分离超平面

  $$
  \max _{w, b}\gamma
  $$

  $$
  \text { s.t. } \quad y_{i}\left(\frac{w}{\|w\|} \cdot x_{i}+\frac{b}{\|w\|}\right) \geqslant \gamma, \quad i=1,2, \cdots, N
  $$

  - 将几何间隔改为函数间隔：
    $$
    \max _{w, b} \frac{\hat{\gamma}}{\|w\|}
    $$

    $$
    \text { s.t. } \quad y_{i}\left(w \cdot x_{i}+b\right) \geqslant \hat{\gamma}, \quad i=1,2, \cdots, N
    $$

  - **将函数间隔固定为1，最大化改为最小化（得到凸二次规划问题）**
    $$
    \min _{w, b} \frac{1}{2}\|w\|^{2}
    $$

$$
\text { s.t. } \quad y_{i}\left(w \cdot x_{i}+b\right)\geqslant 1, \quad i=1,2, \cdots, N
$$



### 2.3 学习的对偶算法

- 拉格朗日对偶问题

  - 转化为最小约束的问题：
    $$
    \min _{w, b} \frac{1}{2}\|w\|^{2}
    $$
    
    $$
    \text { s.t. } \quad y_{i}\left(w \cdot x_{i}+b\right) \geqslant 1, \quad i=1,2, \cdots, N
    $$

  - **拉格朗日乘子法（带约束的问题转为无约束的问题）**，这也是**损失函数**
    在这里，对于系数$\alpha$，非支持向量的系数为0，支持向量的系数大于0
    
  - $$
    L(w, b, \alpha)=\frac{1}{2}\|w\|^{2}-\sum_{i=1}^{N} \alpha_{i} y_{i}\left(w \cdot x_{i}+b\right)+\sum_{i=1}^{N} \alpha_{i} 
    $$

* 对偶问题求解 —— 求$\min_{w,b} L(w,b,a)$  - 原始问题的对偶问题极大极小问题，后面两部分始终小于0

  * 对w、b求偏导：
    $$
    \begin{array}{l}{\nabla_{w} L(w, b, \alpha)=w-\sum_{i=1}^{N} \alpha_{i} y_{i} x_{i}=0} \\ {\nabla_{b} L(w, b, \alpha)=\sum_{i=1}^{N} \alpha_{i} y_{i}=0}\end{array}
    $$
    得到：
    $$
    \begin{array}{l}{w=\sum_{i=1}^{N} \alpha_{i} y_{i} x_{i}} \\ {\sum_{i=1}^{N} \alpha_{i} y_{i}=0}\end{array}
    $$

  * 将其带入拉格朗日函数
    $$
    \begin{aligned} L(w, b, \alpha) &=\frac{1}{2} \sum_{i=1}^{N} \sum_{j=1}^{N} \alpha_{i} \alpha_{j} y_{i} y_{j}\left(x_{i} \cdot x_{j}\right)-\sum_{i=1}^{N} \alpha_{i} y_{i}\left(\left(\sum_{j=1}^{N} \alpha_{j} y_{j} x_{j}\right) \cdot x_{i}+b\right)+\sum_{i=1}^{N} \alpha_{i} \\ &=-\frac{1}{2} \sum_{i=1}^{N} \sum_{j=1}^{N} \alpha_{i} \alpha_{j} y_{i} y_{j}\left(x_{i} \cdot x_{j}\right)+\sum_{i=1}^{N} \alpha_{i} \end{aligned}
    $$

* 对偶问题求解 —— 求$\min_{w,b} L(w,b,a)$对a的极大

  * **对偶问题为**
    $$
    \begin{array}{ll}{\max _{\alpha}} & {-\frac{1}{2} \sum_{i=1}^{N} \sum_{j=1}^{N} \alpha_{i} \alpha_{j} y_{i} y_{j}\left(x_{i} \cdot x_{j}\right)+\sum_{i=1}^{N} \alpha_{i}} \\ {\text { s.t. }} & {\sum_{i=1}^{N} \alpha_{i} y_{i}=0} \\ {} & {\alpha_{i} \geqslant 0, \quad i=1,2, \cdots, N}\end{array}
    $$

  * **极大转换成极小**
    $$
    \begin{array}{cl}{\min _{\alpha}} & {\frac{1}{2} \sum_{i=1}^{N} \sum_{j=1}^{N} \alpha_{i} \alpha_{j} y_{i} y_{j}\left(x_{i} \cdot x_{j}\right)-\sum_{i=1}^{N} \alpha_{i}} \\ {\text { s.t. }} & {\sum_{i=1}^{N} \alpha_{i} y_{i}=0} \\ {} & {\alpha_{i} \geqslant 0, \quad i=1,2, \cdots, N}\end{array}
    $$

  * 假设有解：$\mathrm{a}^{*}=\left(\alpha_{1}^{*}, \alpha_{2}^{*}, \ldots, \alpha_{N}^{*}\right)^{\mathrm{T}}$，则最优化的解为：
    $$
    \begin{array}{c}{w^{*}=\sum_{i=1}^{N} \alpha_{i}^{*} y_{i} x_{i}} \\ {b^{*}=y_{j}-\sum_{i=1}^{N} \alpha_{i}^{*} y_{i}\left(x_{i} \cdot x_{j}\right)}\end{array}
    $$

  * 并可以证明满足KKT的条件。

  * 补充：极小问题满足二次规划问题，通过代入数据集后求解a从而求得w和b

## 3 线性(不可分)支持向量机与软间隔

### 3.1 引入软间隔后的问题

* 引入松弛变量后的约束
  $$
  y_{i}\left(w \cdot x_{i}+b\right) \geqslant 1-\xi_{i}
  $$

* 引入松弛变量后的目标函数
  $$
  \frac{1}{2}\|w\|^{2}+C \sum_{i=1}^{N} \xi_{i}
  $$

* 线性不可分的学习问题变为如下凸二次规划问题：
  $$
  \begin{array}{cl}{\min _{w, b, \xi}} & {\frac{1}{2}\|w\|^{2}+C \sum_{i=1}^{N} \xi_{i}} \\ {\text { s.t. }} & {y_{i}\left(w \cdot x_{i}+b\right) \geqslant 1-\xi_{i}, \quad i=1,2, \cdots, N} \\ {} & {\xi_{i} \geqslant 0, \quad i=1,2, \cdots, N}\end{array}
  $$

### 3.2 对偶问题与求解

* 拉格朗日函数为
  $$
  L(w, b, \xi, \alpha, \mu) \equiv \frac{1}{2}\|w\|^{2}+C \sum_{i=1}^{N} \xi_{i}-\sum_{i=1}^{N} \alpha_{i}\left(y_{i}\left(w \cdot x_{i}+b\right)-1+\xi_{i}\right)-\sum_{i=1}^{N} \mu_{i} \xi_{i}
  $$

* 对偶问题求解 —— 求$\min_{w,b,\xi} L(w, b, \xi, \alpha, \mu)$

  * 对各变量的偏导数
    $$
    \begin{array}{l}{\nabla_{w} L(w, b, \xi, \alpha, \mu)=w-\sum_{i=1}^{N} \alpha_{i} y_{i} x_{i}=0} \\ {\nabla_{b} L(w, b, \xi, \alpha, \mu)=-\sum_{i=1}^{N} \alpha_{i} y_{i}=0} \\ {\nabla_{\xi_{i}} L(w, b, \xi, \alpha, \mu)=C-\alpha_{i}-\mu_{i}=0}\end{array}
    $$

  * 偏导数为0的结果
    $$
    \begin{array}{c}{w=\sum_{i=1}^{N} \alpha_{i} y_{i} x_{i}} \\ {\sum_{i=1}^{N} \alpha_{i} y_{i}=0} \\ {C-\alpha_{i}-\mu_{i}=0}\end{array}
    $$

* 对偶问题求解 —— 对$\min_{w,b,\xi} L(w, b, \xi, \alpha, \mu)$求a的极大

$$
\begin{array}{ll}{\max _{\alpha}} & {-\frac{1}{2} \sum_{i=1}^{N} \sum_{j=1}^{N} \alpha_{i} \alpha_{j} y_{i} y_{j}\left(x_{i} \cdot x_{j}\right)+\sum_{i=1}^{N} \alpha_{i}} \\ {\text { s.t. }} & {\sum_{i=1}^{N} \alpha_{i} y_{i}=0} \\ {} & {C-\alpha_{i}-\mu_{i}=0} \\ {} & {\alpha_{i} \geqslant 0} \\ {} & {\mu_{i} \geqslant 0, \quad i=1,2, \cdots, N}\end{array}
$$

* 利用等式约束消去u，最终的结果

$$
\begin{array}{c}{w^{*}=\sum_{i=1}^{N} \alpha_{i}^{*} y_{i} x_{i}} \\ {b^{*}=y_{j}-\sum_{i=1}^{N} y_{i} \alpha_{i}^{*}\left(x_{i} \cdot x_{j}\right)}\end{array}
$$

* 并可以证明满足KKT的条件。

## 4 非线性支持向量机与核函数

### 4.1 核函数

* 非线性带来高维转换（高维空间更容易线性可分）
* 对偶表示带来内积
* 核函数的定义

$$
\phi(x): \mathcal{X} \rightarrow \mathcal{H}
$$

$$
K(x, z)=\phi(x) \cdot \phi(z)
$$

### 4.2 正定核

* 定义映射并构成向量空间S
* 在S上定义内积，构成内积空间
* 将S完备化构成希尔伯特空间





### 4.3 常用的核函数

* 多项式核函数
* 高斯核函数
* sigmoid核函数



## 小结

　SVM算法的主要优点有：

1. 解决高维特征的分类问题和回归问题很有效，在特征维度大于样本数时依然有很好的效果。
2. 仅仅使用一部分支持向量来做超平面的决策，无需依赖全部数据。
3. 有大量的核函数可以使用，从而可以很灵活的来解决各种非线性的分类回归问题。
4. 样本量不是海量数据的时候，分类准确率高，泛化能力强。

　

SVM算法的主要缺点有：

1. 如果特征维度远远大于样本数，则SVM表现一般。
2. SVM在样本量非常大，核函数映射维度非常高时，计算量过大，不太适合使用。
3. 非线性问题的核函数的选择没有通用标准，难以选择一个合适的核函数。
4. SVM对缺失数据敏感。





## reference：

```
《统计学习方法》
《机器学习》——西瓜书
Bilibili白板推导
https://www.cnblogs.com/pinard/p/6113120.html
```

