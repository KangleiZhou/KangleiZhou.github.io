---
layout: post
category: ML
title: 【机器学习】数据降维
tagline: by Kanglei Zhou
tags: 
  - Machine learning
published: true

---

# 数据降维（Dimensionality Reduction）

[TOC]

机器学习领域中所谓的降维就是指采用某种映射方法，将原高维空间中的数据点映射到低维度的空间中。

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

- 降维的好处

  - 直观地好处是维度降低了，**便于计算和可视化**，其更深层次的意义在于**有效信息的提取综合及无用信息的摈弃**。

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

- 降维的本质

  - 学习一个映射函数 $f : x\to y$，其中 $x$ 是原始数据点的表达，目前最多使用向量表达形式。 $y$ 是数据点映射后的低维向量表达，通常 $y$ 的维度小于 $x$ 的维度（当然提高维度也是可以的）。$f$ 可能是**显式的**或**隐式的**、**线性的**或**非线性的**。

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

>  **例**	将 $D$ 维数据集 $\{x_n\},n=1,2,\cdots,N$ 降为 $M$ 维，其中 $ M < D$

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
\begin{equation*}
\color{blue}
S = \frac{1}{N} \sum_{n=1}^{N}\left(x_n - \bar{x} \right)^2
\end{equation*}
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


在 $M$ 维变量（$ M<D$）生成的空间中对其进行表示


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

在实际应用中，样本维数可能很高，远大于样本的个数在人脸识别中，1000 张人脸图像，每张图像 $100\times100$ 像素。$D$ 维空间，$N$ 个样本点，$ N<D$。$X$ 是 $N \times D$ 维的数据矩阵，其行向量为 $(x_n - \bar{x})^T$，则 $S$ 可以表示为


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

根据贝叶斯定理，有


$$
p(x) = \int p(x|z) p(z) dz = \mathcal{N}(x|\mu,C)
$$


其中


$$
C = WW^T + \sigma^2I
$$


因此


$$
\begin{aligned}
\mathbb{E}[x] &= \mathbb{E}[Wz + \mu +\epsilon] = \mu  \\
\mathrm{cov}[x] &= \mathbb{E}[(Wz+\epsilon)(Wz+\epsilon)^T] \\
&=\mathbb{E}[Wzz^TW^T] + \mathbb{E}[\epsilon\epsilon^T] \\
&=WW^T + \sigma^2I
\end{aligned}
$$


一般利用最大似然估计进行求解，给定 $X = \{x_n\}$，求其对数似然函数


$$
\begin{aligned}
\ln p(X|\mu,W,\sigma^2) &= \sum_{n = 1}^N\ln p(x_n|\mu,W,\sigma^2) \\
&=-\frac{ND}{2}\ln(2\pi) - \frac{N}{2}\ln|C| - \frac{1}{2} \sum_{i=1}^N(x_n-\mu)^TC^{-1}(x_n-\mu)
\end{aligned}
$$


对 $\mu$ 求导并置为 0 得


$$
\frac{\partial \ln p(X|\mu,W,\sigma^2)}{\partial \mu} = \sum_{i=1}^N(x_n-\mu) C^{-1} = 0
$$


于是，有


$$
\hat{\mu} = \frac{1}{N}\sum_{i = 1}^N x_i
$$


代入似然函数，有


$$
\begin{aligned}
\ln p(X|\mu,W,\sigma^2) &= -\frac{N}{2}\{D\ln(2\pi) + \ln|C| +\mathrm{Tr}(C^{-1}S) \}
\end{aligned}
$$


其中 $S$ 为相关矩阵。因此


$$
\hat{\sigma}^2 = \frac{1}{D-M}\sum_{i=M+1}^D \lambda_i
$$


## 讨论

### PCA的优点

- 具有很**高普适性**，最大程度地保持了原有数据的信息；
- 可对**主元的重要性进行排序**，并根据需要略去部分维数，达到降维从而简化模型或对数据进行压缩的效果；
- 完全**无参数限制**，在计算过程中不需要人为设定参数或是根
  据任何经验模型对计算进行干预，最终结果只与数据相关。

![](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page46_Image1.jpg)

### PCA的局限性

- 假设模型是**线性的**，也就决定了它能进行的主元分析之间的关系也是线性的；
- 假设概率分布模型是**指数型**；
- 假设数据具有**较高信噪比**，具有最高方差的一维向量被看作是主元，而方差较小的变化被认为是噪声

![](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page47_Image1.jpg)

### PCA vs. LDA

- PCA 追求降维后能够最大化保持数据内在信息，并通过衡量在投影方向上的数据方差来判断其重要性。但这**对数据的区分作用并不大**，反而可能使得数据点混杂在一起。

![PCA ($\phi_2$) vs LDA ($\phi_1$)](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page48_Image1.jpg)

- LDA 所追求的目标与 PCA 不同，不是希望保持数据最多的信息，而是希望数据在降维后能够**很容易地被区分开**。

![PCA (${\color{\magenta}红色}$) vs LDA (${\color{green}绿色}$)](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page49_Image1.jpg)

## 核主成分分析

将主成分分析的线性假设一般化使之适应非线性数据

- 传统 PCA：$D$ 维样本 $\{x_n\}, n=1,2,\cdots,N$，$\sum_{n=1}^N = 0$

$$
\begin{aligned}
S u_i &= \lambda_i u_i \\
S &= \frac{1}{N}\sum_{n=1}^Nx_nx_n^T \\
u_i^Tu_i &= 1
\end{aligned}
$$

- 核 PCA：非线性映射 $\phi(x),x\to \phi(x_n)$，$\sum_{n=1}^N\phi(x_n) = 0$

$$
\begin{aligned}
Cv_i &= \lambda_i v_i\\
c &= \frac{1}{N}\sum_{n=1}^N \phi(x_n) \phi(x_n)^T\\
& \Downarrow \\
\frac{1}{N}\sum_{n=1}^N\phi(x_n) \{ \phi(x_n)^T v_i \} &= \lambda_i v_i \\
& \Downarrow \\
v_i &= \sum_{n=1}^N  a_{\mathrm{in}} \phi(x_n)\\
& \Downarrow \\
\frac{1}{N}\sum_{n=1}^N\phi(x_n)  \phi(x_n)^T \sum_{m=1}^N  a_{\mathrm{in}} \phi(x_m)  &= \lambda_i \sum_{n=1}^N  a_{\mathrm{in}} \phi(x_n) \\
& \Downarrow k(x_n,x_m) = \phi(x_n)^T\phi(x_m) \\
\frac{1}{N}\sum_{n=1}^N k(x_l,x_n)  \sum_{m=1}^N  a_{\mathrm{in}} k(x_n,x_m)  &= \lambda_i \sum_{n=1}^N  a_{\mathrm{in}} k(x_l,x_n) \\
& \Downarrow \\
K^2a_i &= \lambda_iNKa_i\\
& \Downarrow \\
Ka_i&=\lambda_iNa_i
\end{aligned}
$$

![核 PCA](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page52_Image1.jpg)

![核 PCA](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page52_Image2.jpg)

![核 PCA](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page53_Image1.jpg)

## 等距映射（ISO-Metric Mapping）

### 概述

等距映射的思想：保持数据点内在几何性质 (**测地距离**)

![测地距离与欧氏距离](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page54_Image1.jpg)

对给定数据 $X=\{x_1,x_2,\cdots,x_N\}$，构造图 $G = \{V,E\}$。其中 $V$ 是顶点集合，$E$ 是边的集合。若 $\mathrm{d}(i,j) = \mathrm{dist}(x_i,x_j)$ 小于某个值 $\epsilon$ （$\epsilon$-ISOMAP），或 $j$ 是 $i$ 的 $K$ 近邻（$K$-ISOMAP），则顶点 $i$ 和 $j$ 的边权值设为 $d(i,j)$，否则为 0。

![图 $G = \{V,E\}$](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page55_Image1.jpg)

计算图 $G = \{V,E\}$ 中任意两点间的最短距离，得到矩阵 $D_G(i,j)$，可选算法有

- Dijkstra 最短路径算法
- Floyd–Warshall 算法

![带权图](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page56_Image1.jpg)

![$k$-近邻](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page56_Image2.jpg)

令 $H = I_n -\frac{1}{N}$ 为中心矩阵（Centering Matrix），并定义平方距离矩阵


$$
S(i,j) = D^2_G(i,j)
$$


求矩阵 $L = -\frac{1}{2} HSH$ 的特征值与特征向量（按特征值降序排列），$\lambda_p$ 为第 $p$ 个特征值， $v_p$为对应的特征向量。则，降维矩阵为


$$
\begin{pmatrix}
\sqrt{\lambda_1}v_1&\cdots&\sqrt{\lambda_d}v_d
\end{pmatrix}_{N\times d}
$$


例如，原始数据如下：

![原始数据](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page58_Image1.jpg)

等距映射后的数据如下：

![降维后数据](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page58_Image2.jpg)

### 计算步骤

- 构造临近关系图：对每一个点，将它与指定半径邻域内所有点相连(或与指定个数最近邻相连)
- 计算最短路径：计算临近关系图所有点对之间的最短路径，得到距离矩阵
- 多尺度分析：将高维空间中的数据点投影到低维空间，使投影前后的距离矩阵相似度最大

### 优缺点

- **优点**：非线性、非迭代、全局最优、参数可调节
- **缺点**：容易受噪声干扰、在大曲率区域存在短路现象、不适用于非凸参数空间、大样本训练速度慢

![](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page60_Image1.jpg)

## 局部线性嵌入

### Local Linear Embedding (LLE)

**基本思想**：保持数据点的原有流形结构

![Local Linear Embedding (LLE)](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page61_Image3.jpg)

**前提假设**：采样数据所在的低维流形在局部是线性的，每个采样点可以用它的近邻点线性表示。
**学习目标**：在低维空间中保持每个邻域中的权值不变，即假设嵌入映射在局部是线性的条件下，最小化重构误差。

### 计算过程

- 寻找每个样本点的 $K$ 近邻 $x_{ij}~(j=1,\cdots,k)$
- 对每个点用 $K$ 个近邻进行重建，即求一组权值 $w_{ij}$，满足 $\sum_{j}w_{ij} = 1$，使得

$$
\min~\sum_i|x_i-\sum_jw_{ij}x_{ij}|^2
$$

- 求低维空间中的点集 $y_i$，使得

$$
\min~\sum_{i} |y_i-\sum_j{w_{ij}y_{ij}}|^2
$$

- 计算权值，首先构造局部协方差矩阵 $C^i$

$$
C_{jk}^i= (x_i-x_j)\cdot(x_i-x_k)
$$

- 最小化

$$
\begin{aligned}
\min ~~&\sum_i|x_i-\sum_j w_{ij}x_{ij}|^2\\
\mathrm{s.t.}~~&\sum_jw_{ij} =1
\end{aligned}
$$

- 求得

$$
w_{ij} = \frac{\sum_k (C_{jk}^i)^{-1}}{\sum_j\sum_k (C_{jk}^i)^{-1}}
$$

- 计算低维数据

$$
\begin{aligned}
\min ~ &\sum_i |y_i - \sum_j w_{ij} y_{ij} |^2\\
&\Downarrow M= (I-W)^T(I-W) \\
\min ~ &\sum_i\sum_jM_{ij}(y_i\cdot y_j) \\
&\Downarrow \\
MY &= \lambda Y
\end{aligned}
$$

取 $Y$ 为 $M$ 的最小 $d$ 个非零特征值所对应的特征向量，最终的输出结果即为$N\times d$ 大小的矩阵。

![Local Linear Embedding (LLE) 计算步骤](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page65_Image1.jpg)

### 简单例子

![一个例子](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-1-5/data_reduce_Page68_Image1.jpg)

