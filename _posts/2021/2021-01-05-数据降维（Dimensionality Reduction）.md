---
layout: post
category: Dimensionality Reduction
title: 【机器学习】数据降维
tagline: by Kanglei Zhou
tags: 
  - Machine learning
published: true

---

# 数据降维（Dimensionality Reduction）

![机器学习主要问题](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page20_Image1.jpg)

## 数据维数

 维数 (又称维度)

- 数学中：独立参数的数目
- 物理中：独立时空坐标的数目

![$x-y-z$ 维度](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page21_Image1.jpg)

> 例	“点是0维、直线是1维、平面是2维、体是3维”

- 点基于点是 0 维：在点上定位一个点，不需要参数
- 点基于直线是 1 维：在直线上定位一个点，需要 1 个参数
- 点基于平面是 2 维：在平面上定位一个点，需要 2 个参数
- 点基于体是 3 维：在体上定位一个点，需要 3 个参数

![$64×64$ 像素数字 "3"](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page22_Image1.jpg)

又如，$64×64$ 像素数字放入 $100×100$ 像素白板

![$100×100$ 像素白板](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page22_Image2.jpg)

实际上中，可以通过如下操作增加维数：

- 水平 / 垂直的平移变化
- 旋转变化
- 尺度变化
- 形状变化（不同人写作的习惯）
- 光照变换
- ……

这些变换的因素是**潜变量**（Latent Variables），潜变量在。

## 数据降维

- 为什么要降维？

- 在原始的高维空间中，包含冗余信息和噪声信息，会在实际应用中引入误差，影响准确率；而降维可以提取数据内部的本质结构，减少冗余信息和噪声信息造成的误差，提高应用中的精度。

- 一个简单的例子

  - 噪声：选择一个方向投影过滤噪声

  ![含有噪声的数据](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page24_Image4.jpg)

  - 冗余

  ![低冗余数据](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page24_Image2.jpg)

  ![高冗余数据](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page24_Image3.jpg)

- 降维

  - 利用某种映射将原高维度空间的数据点投影到低维度的空间：

  $$
  f:X\to Y
  $$

  ![高冗余数据](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page24_Image1.jpg)

## 降维方法

- 主成分分析（Principal Component Analysis）
- 等距映射（Isometric Mapping）
- 局部线性嵌入（Locally Linear Embedding）
- ……

![数据降维](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page25_Image1.jpg)

## 主成分分析

### 概述

主成分分析(Principal Component Analysis, PCA)，将原有众多具有一定相关性的指标重新组合成一组少量互相无关的综合指标。

- K. Pearson (1901年论文) 针对非随机变量

- H. Hotelling (1933年论文) 推广到随机向量

![二维降维](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page26_Image1.jpg)

> - 使得降维后样本的**方差尽可能大**
> - 使得降维后数据的**均方误差尽可能小**

### 算法原理

>  **例**	将 $D$ 维数据集 $\{x_n\},n=1,2,\cdots,N$ 降为 $M$ 维，其中 $M < D$

#### 特殊情况

最大方差思想：**使用较少的数据维度保留住较多的原数据特性**

首先考虑 $M = 1$，定义这个空间的投影方向为 $D$ 维向量 $u_1$，出于方便且不失一般性，令 $u_1^Tu_1 = 1$，即 $u_1$ 为单位向量。则

- 每个数据点 $x_n$ 在新空间中表示为标量 $u_1^Tx_n$
- 样本均值在新空间中表示为 $u_1^T \bar{x}$，其中

$$
\bar{x} = \frac{1}{N}\sum_{n=1}^{N}x_n
$$

- 投影后样本方差尽可能大，表示为 

$$
\max S' = \max_{u_1\in\mathbb{R}^M} \frac{1}{N} \sum_{n=1}^{N}\left( u_1^Tx_n - u_1^T\bar{x} \right)^2 = \max_{u_1\in\mathbb{R}^M} u_1^TSu_1
$$

其中，原样本方差为
$$
S = \frac{1}{N} \sum_{n=1}^{N}\left(x_n - \bar{x} \right)^2
$$
该约束优化问题可表述为
$$
\begin{equation*}
\begin{aligned}
\max_{u_1\in\mathbb{R}^M} ~~& u_1^TSu_1 \\
\mathrm{s.t.} ~~& u_1^Tu_1 = 1
\end{aligned}
\end{equation*}
$$
利用拉格朗日乘子法，构造 Lagrange 函数
$$
\mathcal{L}(u_1) = u_1^TSu_1 + \lambda_1(1-u_1^Tu_1)
$$
对 $u_1$ 求偏导并令偏导数为 0，有
$$
\frac{\partial \mathcal{L}}{\partial u_1} = 2 S u_1 - 2 \lambda_1 u_1 = 0
$$
即
$$
S u_1 = {\color{red}\lambda_1} u_1
$$
结论：$u_1$ 是 $S$ 的特征向量。进一步，有
$$
\max_{u_1 \in \mathbb{R}^M} u_1^TSu_1 = \lambda_1
$$
即 $u_1$ 是 $S$ 最大特征值对应的特征向量时，方差取到最大值，称 $u_1$ 为第一主成分。

#### 一般情况

- 最小均方误差思想：**使原数据与降维后的数据 (在原空间中的重建) 的误差最小**

考虑更一般性的情况（$M>1$）, 新空间中数据方差最大的最佳投影方向由协方差矩阵 $S$ 的 $M$ 个特征向量定义, 其分别对应 $M$ 个最大的特征值 $\lambda_1,\lambda_2,\cdots,\lambda_M$。

- 首先获得方差最大的1维，生成该维的补空间；
- 继续在补空间中获得方差最大的1维，生成新的补空间；
- 依次循环下去得到M维的空间

定义一组**单位正交**的 $D$ 维基向量 $\{u_i\},i=1,2,\cdots,D$，满足
$$
u_i^Tu_j = \delta_{ij} \to 0, u_i^Tu_i = 1
$$
由于基是完全的，每个数据点可以表示为基向量的线性组合
$$
x_n = \sum_{i  =1}^{D} \alpha_{ni}u_i
$$
相当于进行了坐标转换
$$
\begin{aligned}
\{x_{n1},x_{n2},\cdots,x_{nD}\} &\mathop{\longrightarrow}^{\{u_i\}} \{\alpha_{n1},\alpha_{n2},\cdots,\alpha_{nD}\}
\\
&~~\Downarrow \\
\alpha_{nj} &= x_n^Tu_j
\end{aligned}
$$
则
$$
x_n = \sum_{i=1}^D\left( x_n^T u_i \right)u_i
$$
在 $M$ 维变量（$M<D$）生成的空间中对其进行表示
$$
\tilde{x}_n = \sum_{i=1}^M {\color{red}z_{ni}}u_i + \sum_{i = M+1}^D {\color{magenta}b_i} u_i
$$
目标为最小化失真率
$$
\min ~J = \min~\frac{1}{N} \sum_{n=1}^N\| x_n - \tilde{x}_n \|^2
$$
导数置为 0 得
$$
\begin{aligned}
z_{nj} &= x_n^T u_j, j = 1,2,\cdots,M \\
b_j &= \bar{x}^T u_j,j=M+1,\cdot,\cdots,D
\end{aligned}
$$
则有
$$
\begin{aligned}
x_n - \tilde{x}_n &= \sum_{i=M+1}^D\{ (x_n-\bar{x})^T u_i \} u_i

\end{aligned}
$$
即
$$
\begin{aligned}

J &= \frac{1}{N}\sum_{n=1}^N \sum_{M+1}^D \left(x_n^Tu_i - \bar{x}^Tu_i\right)^2 \\
&=\sum_{i=M+1}^Du_i^TSu_i

\end{aligned}
$$
构造拉格朗日函数，得
$$
\mathcal{L} =\sum_{i=M+1}^Du_i^TSu_i + \sum_{i=M+1}^D\lambda_i (1-u_i^Tu_i)
$$
对 $u_i$ 求偏导，并置为 0 得
$$
S u_i = \lambda_i u_i
$$
$J$ 最小时取 $D-M$ 个最小得特征值。对应的失真度为
$$
J = \sum_{i = M+1}^D \lambda_i
$$

### 算法步骤

- 计算给定样本 $\{x_n\},n=1,2,\cdots,N$ 的均值 $\bar{x}$ 和协方差矩阵 $S$
- 计算 $S$ 的特征向量与特征值
- 将特征值从大到小排列，前 $M$ 个特征值 $\lambda_1,\lambda_2,\cdots,\lambda_M$ 所对应的特征向量 $u_1,u_2,\cdots,u_M$ 构成投影矩阵

![样本点及投影方向 $v_1$ 和 $v_2$](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page33_Image1.jpg)

![不同样本分布的投影方向](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page33_Image2.jpg)

### 应用

![均值图像及不同成分所对应的图像](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page34_Image1.jpg)

特征值分布谱特征值由大到小排列，如下

![特征值排列](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page34_Image2.jpg)

选取前 $M$ 个特征值进行降维，有如下结果

![$M$ 取不通值时的结果](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page35_Image2.jpg)

失真度分布谱随 $M$ 取值由小到大排列，如下：

![失真度分布谱](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page35_Image1.jpg)

### 利用 PCA 处理高维数据

在实际应用中，样本维数可能很高，远大于样本的个数在人脸识别中，1000 张人脸图像，每张图像 $100\times100$ 像素。$D$ 维空间，$N$ 个样本点，$N<D$。$X$ 是 $N \times D$ 维的数据矩阵，其行向量为 $(x_n - \bar{x})^T$，则 $S$ 可以表示为
$$
S = N^{-1} X^T X
$$
其中 $S \in \mathbb{R}^{10000\times10000}$，显然 $D\times D$ 的维度相当大，引起维度灾难。

又
$$
\begin{aligned}
\frac{1}{N} X^TXu_i &= \lambda_i u_i
\\
&\Downarrow
\\
\frac{1}{N} XX^T (Xu_i) &= \lambda_i (Xu_i)
\end{aligned}
$$
令 $v_i = X u_i$，得到
$$
\frac{1}{N} XX^Tv_i = \lambda_iv_i
$$
其中 $XX^T \in \mathbb{R}^{100\times100}$，显然 $N\times N$ 的维度相比 $D\times D$ 要小的多了。

因此，需对 $\frac{1}{N} XX^Tv_i$ 进行特征值分解得 $\lambda_i$ 和 $v_i$，所以
$$
u_i \propto X^T v_i
$$
因为 $u_i$ 为单位向量，所以
$$
u_i = \frac{1}{\|X^Tv_i\|_2}X^Tv_i
$$
这就是所谓的**奇异值分解** (Singular Value Decomposition, SVD)。

## 概率主成分分析

- PCA 的概率表示

隐变量 $z$ 以如下形式产生 $D$ 维观测变量 $x$
$$
x = Wz + \mu + \epsilon
$$
其中，$\mu$ 为均值，$\epsilon$ 为高斯噪声。 $z$ 为 $M$ 维的隐变量，且满足高斯分布
$$
p(z) = \mathcal{N}(z|0,I)
$$
$x$ 以 $z$ 为条件的分布也满足高斯分布
$$
p(x|z) = \mathcal{N}(x|Wz+\mu,\sigma^2I)
$$
![以有向图表示](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page42_Image1.jpg)

- 从隐空间到数据空间的映射，与 PCA 的传统视角相反
- 从数据空间到隐空间的映射，可以由**贝叶斯定理**得到

![1 维隐变量空间 - 2 维数据空间 - 边缘分布的等密度线](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page43_Image1.jpg)