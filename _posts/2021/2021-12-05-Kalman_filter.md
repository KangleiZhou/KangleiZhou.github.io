---
layout: post
category: DL
title: Kalman filter
tagline: by Kanglei Zhou
author: Kanglei Zhou
tags: 
  - 运动分析
published: true
---

[公众号阅读更佳](https://mp.weixin.qq.com/s?__biz=MzUyMTE2NDYxMQ==&mid=2247490001&idx=1&sn=454b99fae03147c916e663794b530874&chksm=f9de1bfdcea992eb781e1ba4d49508742d9c4948edb298840fadd6b3315cdf3612c7d6f81134&token=514611700&lang=zh_CN#rd)


1960 年，**鲁道夫·埃米尔·卡尔曼**（Rudolf. Emil. Kalman）发表了用递归方法解决离散数据线性滤波问题的论文（A New Approach to Linear Filtering and Prediction Problems），卡尔曼滤波（Kalman Filter）由此得名。

![鲁道夫·埃米尔·卡尔曼](https://gitee.com/ZhouKanglei/jidianxia/raw/master/2021-12-5/1638666601157-Rudolf_Kalman.jpg)

其是一个高效的**递归**（自回归）滤波器，它可以实现从一系列的噪声观测中，估计动态系统的状态。它的功能强大，可以估计信号的过去和当前状态，甚至能估计将来的状态，即使并不知道模型的确切性质。

![卡尔曼滤波器会追踪系统的估计状态，以及估计的变异量或是不确定性。会透过状态转换模型以及其测量来更新估计值。$\hat{\pmb{x}}_{k|k-1}$ 是指时间 $k$ 时的估计，还没有考虑 $k$ 次测量 $y_k$ 的资讯，$P_{k|k-1}$ 为对应的不确定性[](https://zh.wikipedia.org/wiki/%E5%8D%A1%E5%B0%94%E6%9B%BC%E6%BB%A4%E6%B3%A2 "维基百科-卡尔曼滤波")](https://gitee.com/ZhouKanglei/jidianxia/raw/master/2021-12-5/1638665499358-400px-Basic_concept_of_Kalman_filtering.svg.png)

本质上来讲，滤波就是一个**信号处理与变换**（去除或减弱不想要的成分，增强所需成分）的过程，可以通过**硬件**和**软件**来实现，卡尔曼滤波属于一种[软件滤波方法](https://file.elecfans.com/web1/M00/7F/8B/o4YBAFwnp8GAO_xSAAK4e11SknQ423.pdf "彭丁聪. 卡尔曼滤波的基本原理及应用[J]. 软件导刊, 2009, 8(11): 32-34.")。

> 其基本思想是：以**最小均方误差**为最佳估计准则，采用信号与噪声的状态空间模型，利用前一时刻的估计值和当前时刻的观测值来更新对状态变量的估计，求出当前时刻的估计值，算法根据建立的系统方程和观测方程对需要处理的信号做出满足最小均方误差的估计。

# 卡尔曼滤波器算法

卡尔曼滤波建立在**线性代数**和**隐马尔可夫模型**（hidden Markov model）上。其基本动态系统可以用一个**马尔可夫链**表示，该马尔可夫链建立在一个被高斯噪声（即正态分布的噪声）干扰的线性算子上的。系统的状态可以用一个元素为实数的向量表示。随着离散时间的每一个增加，这个线性算子就会作用在当前状态上，产生一个新的状态，并也会带入一些噪声，同时系统的一些已知的控制器的控制信息也会被加入。同时，另一个受噪声干扰的线性算子产生出这些隐含状态的**可见输出**。

## 卡尔曼滤波 vs 隐马尔可夫模型

卡尔曼滤波器可以被看作为类隐马尔科夫模型，它们的[显著不同点](https://longaspire.github.io/blog/%E5%8D%A1%E5%B0%94%E6%9B%BC%E6%BB%A4%E6%B3%A2/ "卡尔曼滤波器")在于：

- 隐状态变量的**取值空间**是一个**连续**的空间，而不是离散的状态空间；
- 隐马尔科夫模型可以描述下一个状态的一个**任意分布**，这也与应用于卡尔曼滤波器中的高斯噪声模型相反。


## 动态系统模型
为了从一系列有噪声的观察数据中用卡尔曼滤波器估计出被观察过程的内部状态，必须把这个过程在卡尔曼滤波的框架下建立模型。也就是说对于每一步 $k$，定义矩阵 $\mathbf{F}_k$, $\mathbf{H}_k$, $\mathbf{Q}_k$, $\mathbf{R}_k$，有时也需要定义 $\mathbf{B}_k$。

![卡尔曼滤波器的模型。圆圈代表向量，方块代表矩阵，星号代表高斯噪声，其协方差矩阵在右下方标出](https://gitee.com/ZhouKanglei/jidianxia/raw/master/2021-12-5/1638667513562-250px-Kalman_filter_model.png)

卡尔曼滤波模型假设 $k$ 时刻的真实状态是从 $(k − 1)$ 时刻的状态演化而来，符合下式：

$$
\pmb{x}_k = \mathbf{F}_k \pmb(x)_{k-1} + \mathbf{B}_k \pmb{u}_k + \pmb{w}_k
$$

其中，
- $\mathbf{F}_k$ 是作用在 $\pmb{x}_{k-1}$ 上的状态变换模型（矩阵）；
- $\mathbf{B}_k$ 是作用在控制量向量 $\pmb{u}_{k-1}$ 上的输入-控制模型；
- $\pmb{w}_k$ 是过程噪声，并假设其符合均值为 0，协方差矩阵为 $\mathbf{Q}_k$ 的多元正态分布，即
$$
\pmb{w}_k \sim \mathcal{N}(0, \mathbf{Q}_k)
$$

时刻 $k$，对真实状态 $\pmb{x}_k$ 的一个测量 $\pmb{z}_k$ 满足下式：
$$
\pmb{z}_k = \mathbf{H}_k \pmb{x}_k + \pmb{v}_k
$$
其中，
- $\mathbf{H}_k$ 是观测模型，它把真实状态空间映射成观测空间；
- $\pmb{v}_k$ 是观测噪声，其均值为 0，协方差矩阵为 $\mathbf{R}_k$ 的多元正态分布，即
$$
\pmb{v}_k \sim \mathcal{N}(0, \mathbf{R}_k)
$$

初始状态及每一时刻的噪声 $\{\pmb{x}_0, \pmb{w}_1, \cdots, \pmb{w}_k, \pmb{v}_1, \cdots, \pmb{v}_k\}$ 都认为是互相独立的。

实际上，**很多真实世界的动态系统都并不确切的符合这个模型**；但是由于卡尔曼滤波器被设计在有噪声的情况下工作，一个近似的符合已经可以使这个滤波器非常有用了。

## 卡尔曼滤波 vs 低通滤波

卡尔曼滤波器与大多数滤波器不同之处在于：
> 它是一种纯粹的**时域滤波器**，它不需要像低通滤波器等频域滤波器那样，需要在频域设计再转换到时域实现。

## 预测和更新
卡尔曼滤波器的操作包括两个阶段：预测与更新：
- 在**预测**阶段，滤波器使用上一状态的估计，做出对当前状态的估计。
- 在**更新**阶段，滤波器利用对当前状态的观测值优化在预测阶段获得的预测值，以获得一个更精确的新估计值。


### 预测

对于满足上面的条件（线性随机微分系统，过程和观测都是高斯白噪声），卡尔曼滤波器是最优的信息处理器。

首先，我们要利用系统的过程模型，来预测下一状态的系统。假设现在的系统状态是 $k$，根据系统的模型，可以基于系统的**上一状态而预测出现在状态**：

$$
\pmb{x}_{k \mid k-1} = \mathbf{F}_k \cdot \pmb{x}_{k - 1 \mid k-1} + \mathbf{B}_k \cdot \pmb{u}_k
$$

上述公式称为**预测的状态估计方程**。其中，
- $\pmb{x}_{k \mid k-1}$ 是利用上一状态预测的结果；
- $\pmb{x}_{k-1 \mid k-1}$ 是上一状态最优的结果；
- $\pmb{u}_k$ 为现在状态的控制量，如果没有控制量，它可以为 0。

到现在为止，我们的系统结果已经更新了，可是，对应于 $\pmb{x}_{k∣k−1}$ 的协方差 $\mathbf{P}_{k\mid k-1}$ 还没更新，它实际上描述了预测值的准确程度：

$$
\mathbf{P}_{k\mid k-1} = \mathbf{F}_k \cdot \mathbf{P}_{k-1\mid k-1} \cdot \mathbf{F}_k^T + \mathbf{Q}_k
$$

上述公式称为**预测的协方差矩阵估计方程**。其中，
- $\mathbf{P}_{k \mid k-1}$ 是 $\pmb{x}_{k\mid k-1}$ 对应的协方差；
- $\mathbf{P}_{k-1 \mid k-1}$ 是 $\pmb{x}_{k - 1 \mid k-1}$ 对应的协方差；
- $\mathbf{Q}_k$ 是系统过程的协方差。

### 更新

在进行更新之前，我们先计算三个值：

- 首先是观测余量（measurement residual）：
$$
\pmb{y}_k = \pmb{z}_k - \mathbf{H} \cdot \pmb{x}_{k\mid k-1}
$$
因为观测过程中存在一个观测误差的协方差矩阵，我们可以给出一个观测余量的协方差：

$$
\mathbf{S}_k = \mathbf{H}_k \cdot \mathbf{P}_{k\mid k-1}\cdot\mathbf{H}_k^T + \mathbf{R}_k
$$

接下来给出一个卡尔曼增益（Kalman Gain）：

$$
\mathit{Kg}_k = \mathbf{P}_{k \mid k-1} \cdot \mathbf{H}^T \cdot \mathbf{S}_k^{-1} = \mathbf{P}_{k\mid k-1} \cdot \mathbf{H}^T \cdot (\mathbf{H} \cdot \mathbf{P}_{k\mid k-1} \cdot \mathbf{H}^T + \mathbf{R}_k)^{-1}
$$

现在我们有了现在状态的预测结果，然后我们再收集现在状态的观测值。结合预测值和观测值，我们可以得到现在状态$k$ 的最优化估算值 $\pmb{x}_{k\mid k}$：

$$
\pmb{x}_{k\mid k} = \pmb{x}_{k\mid k-1} + \mathit{Kg}(k) \cdot \mathbf{y}_k
$$

上述方程为**更新的状态估计方程**。

到现在为止，我们已经得到了 $k$ 状态下最优的估算值 $\pmb{x}_{k\mid k}$。但是为了要使得卡尔曼滤波器不断的运行下去直到系统过程结束，我们还要更新 $k$ 状态下 $\pmb{x}_{k\mid k}$ 的协方差：

$$
\mathbf{P}_{k\mid k} = (\mathbf{I}-Kg_k \cdot \mathbf{H}) \cdot \mathbf{P}_{k\mid k-1}
$$
上述方程成为**更新的协方差矩阵估计方程**。其中，
- $\mathbf{I}$ 是单位矩阵，对于单模型观测，$\mathbf{I} = 1$。

当系统进入 $k+1$ 状态时，$\mathbf{P}_{k\mid k}$ 就是观测方程中的 $\mathbf{P}_{k-1\mid k-1}$。这样，算法就可以自回归地运算下去。

# 结合算法模型的举例

在算法模型的基础上，我们再进一步给出一个帮助理解的例子。

## 物体运动状态估计

考虑在无摩擦、无限长的直轨道上的一辆车。该车最初停在位置 0 处，但时不时受到随机的冲击。注意这里我们考虑没有外力的影响，因此忽略掉 $\mathbf{B}_k$ 和 $\pmb{u}_k$。同时考虑 $\mathbf{F}$, $\mathbf{H}$, $\mathbf{R}$, $\mathbf{Q}$ 为常数（此处不用下标）。

每 $\Delta t$ 秒即测量车地位置，但是这个测量是非精确的；我们想建立一个关于其位置以及速度的模型。车的位置以及速度（或者更加一般的，一个粒子的运动状态）可以被线性状态空间描述如下：

$$
{x}_k = 
\begin{pmatrix}
{x} \\ \dot{{x}}
\end{pmatrix}
$$

其中，$\dot{{x}}$ 是速度，即位置对时间地导数。

假设 $k-1$ 时刻和 $k$ 时刻之间，车受到 $a_k$ 的加速度，且符合均值为 0，标准差为 $\sigma_a$ 的正态分布。根据牛顿第一定理，我们推出
$$
x_k = \mathbf{F} x_{k-1} + \mathbf{G} \cdot a_k
$$
其中，两个转移矩阵 
$$
\mathbf{F} = 
\begin{pmatrix}
1 & \Delta t\\
0 & 1
\end{pmatrix}
\cdot \mathbf{G}

= \begin{pmatrix}
\Delta t^2 / 2\\
\Delta t
\end{pmatrix}
$$
可知
$$
\mathbf{Q} = \text{cov}(\mathbf{G} \cdot a) = E[(\mathbf{G} \cdot a) \cdot (\mathbf{G} \cdot a)^{T}] = \mathbf{G} \cdot E[a^2] \cdot \mathbf{G}^T = \mathbf{G} \cdot \sigma_a^{2} \cdot \mathbf{G}^T
$$
每一个时刻我们都会对其进行一个测量过程，测量受到噪声干扰，我们依然假设噪声服从正态分布，均值为 0，标准差为 $\sigma_z$。
$$
z_k = \mathbf{H} \cdot x_k + v_k
$$
其中，我们有
$$
\mathbf{H} = \begin{pmatrix} 1 & 0 \end{pmatrix}, \mathbf{R} = E[v_k \cdot v_k^T] = \sigma_z^2
$$
我们先要提出一个假设初始值，假设车最初的位置和速度是足够准确的：
$$
x_{0 \mid 0} = \begin{pmatrix} 0 \\\ 0 \end{pmatrix}
$$
并且，告诉滤波器初始的测量是准确的，给出一个协方差矩阵：
$$
P_{0 \mid 0} = \begin{pmatrix} 0 & 0 \\\ 0 & 0  \end{pmatrix}
$$
如果我们不确切知道最初的位置与速度，协方差矩阵可以初始化为一个对角线元素为 $B$ 的矩阵，$B$ 取一个比较大的数
$$
P_{0 \mid 0} = \begin{pmatrix} B & 0 \\\ 0 & B  \end{pmatrix}
$$
接下来我们就可以给出 $0+\Delta t$ 时刻对于车的状态的估计：
$$
x_{1\mid 0} = 
\begin{pmatrix}
1 & \Delta t \\
0 & 1
\end{pmatrix}
\cdot
\begin{pmatrix}
0 \\ 0
\end{pmatrix} +
\begin{pmatrix}
\Delta t^2 / 2 \\ \Delta t
\end{pmatrix}
a_0 =
\begin{pmatrix}
a_0 \cdot \Delta t^2 / 2\\
\Delta t \cdot a_0
\end{pmatrix}
$$
其中 $\begin{pmatrix}
a_0 \cdot \Delta t^2 / 2\\
\Delta t \cdot a_0
\end{pmatrix}$ 就是 $0 + \Delta t$ 时刻的位移和速度的估计。

接下来，我们可以得到预测的估计协方差矩阵：

$$
\begin{align*}
P_{1\mid 0} = 
&\begin{bmatrix} 1 & \Delta t \\\ 0 & 1 \end{bmatrix} \cdot \begin{bmatrix} 0 & 0 \\\ 0 & 0 \end{bmatrix} \cdot \begin{bmatrix} 1 & 0 \\\ \Delta t & 1 \end{bmatrix} 
\\ &+ \begin{bmatrix} \Delta t^2/2 \\\ \Delta t \end{bmatrix} \cdot \sigma_a^2 \cdot \begin{bmatrix} \Delta t^2/2 & \Delta t \end{bmatrix} \\
=& \sigma_a^2 \cdot \begin{bmatrix} \Delta t^4/4 & \Delta t^3/2 \\\ \Delta t^3/2 & {\Delta t}^2 \end{bmatrix}
\end{align*}
$$

注意这个时候预测的估计协方差矩阵 $P_{1|0}$ 相比 $P_{0|0}$ 就大一些，这主要是由于预测过程带来的误差。

完整部分请点击【**阅读原文**】

## 算法流程抽象

公式是不是很复杂，但是只要你跟着流程来一趟，应该能够明白 KF 的整个过程。跑完这个例子之后，我们把整个流程抽象出来：
- 先决定当前系统的初始状态，并根据预测方程（过程模型）得到一个下一个时刻预测的状态；
- 根据预测方程中过程的误差，得到当前预测的协方差估计；
- 进入更新阶段，我们根据目前系统的观测值和上一个时刻预测的状态，从转换方程（观测模型）入手，得到一个测量余量；
- 根据转换方程和上个时刻预测的协方差估计，也可以得到一个测量余量的协方差估计；
- 根据1)测量余量的协方差 $S_{k∣k}$、2) 转换方程 $\mathbf{H}$ 和 3) 上个时刻的预测协方差估计 $\mathbf{P}_{k∣k−1}$，我们得到卡尔曼增益 $\mathbf{K}_{k∣k}$；
- 根据卡尔曼增益和测量余量，我们从预测的状态中更新优化当前的状态的值，而这个值可以用来预测下一个时刻的状态；
- 同样，我们根据 1) 卡尔曼增益 $\mathbf{K}_{k∣k}$ 和 2) 上个时刻的预测协方差估计 $\mathbf{P}_{k∣k−1}$，我们把当前更新阶段的协方差 $\mathbf{P}_{k∣k}$ 估计也得到，帮助下一时刻的卡尔曼增益计算。

![Kalman Algorithm](https://gitee.com/ZhouKanglei/jidianxia/raw/master/2021-12-5/1638672242024-kalman_algorithm.jpg)

# 示例代码
给出一个 `matlab` 版本的小程序
```matlab
N = 200;
w(1) = 0;
w = randn(1, N)  % White Gaussian Noise of Prediction

x(1) = 25;
a = 1;  % prediction parameter
for k = 2:N;
  x(k) = a*x(k-1)+w(k-1);
end

V = randn(1, N); % White Gaussian Noise of Measurement
q1 = std(V);
Rvv = q1.^2; % covariance of Measurement
q2 = std(x);
Rxx = q2.^2;
q3 = std(w);
Rww = q3.^2; % covariance of prediction
c = 0.2;   % measurement parameter
Y = c*x+V;

p(1) = 10;  % initial prediction result
s(1) = 26;  % initial optimal result
for t = 2:N;
  p1(t) = a.^2*p(t-1)+Rww;  % covariance of t-1
  b(t) = c*p1(t)/(c.^2*p1(t)+Rvv); % kalman gain
  s(t) = a*s(t-1)+b(t)*(Y(t)-a*c*s(t-1)); % optimal result
  p(t) = p1(t)-c*b(t)*p1(t); % covariance of t
end

t = 1:N;
plot(t, s, 'r', t, Y, 'g', t, x, 'b');
```
