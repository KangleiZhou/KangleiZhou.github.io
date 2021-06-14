---
layout: post
category: GNN
title: 【GNN】Context Aware Graph Convolution for Action Recognition
tagline: by Kanglei Zhou
author: Kanglei Zhou
tags: 
  - GNN
published: true

---

> 目前的一些图卷积方法大多是聚合局部邻居信息的方式完成卷积，特征仅从中心顶点的一个小邻域中提取。但在实践中，运动是多部位关节相互协调配合的过程。例如，`书写` 和 `打字` 需要双手的配合来完成，它们在图结构中是不相邻的。 因此，只研究一只手的运动而不考虑另一只手的运动是很难全面理解动作的。

# Motivation

**一个大的感受野有助于整体理解基于关节的运动**，可以有以下两种思路：

- 堆叠多个图卷积层可以直接增加感受野，但多层叠加需要解决长期依赖问题，计算效率低，增加了网络优化的难度。最重要的是，远距离关节之间只能通过跨卷积层的中间关节进行隐式连接，这阻碍了信息的交换，带来了冗余计算。
- 采用适合长距任务的 `self-attention` 机制。本文中，作者提出一种上下文感知的图卷积来丰富每个身体关节的局部响应与所有其他关节的信息，其实质就是采用 `self-attention` 的思想。

![Context aware graph convolution at the red vertex. Green and purple arrows denote integration of context information and graph convolution along spatial and temporal dimensions. Yellow vertices are convolution participants.](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-6-13/1623556338339-image.png)

# Method

在基于骨骼的动作识别任务中，目标是利用全局上下文丰富局部卷积操作，即通过图卷积和上下文项收集其他关节信息来促进局部运动的提取。

## Context Generation

上下文感知的图卷积是为了丰富传统的局部图卷积，使中心顶点不仅关注其周围的邻居，而且能够感知远处的顶点。


$$
\mathbf{C}^l = \sigma 
	 \left (
	  \mathrm{softmax} 
	  \left (
	  \mathrm{Rele}(\mathbf{Q}^l, \mathbf{K}^l) 
	  \right )
	  \mathbf{V}^l 
	  + \mathbf{b}_{\mathrm{c}}^l
	  \right )
$$


其中，$\mathrm{Rele}(\cdot)$ 是特征匹配得分函数（Alignment score function）或者叫相似度函数，$\mathbf{Q}^l, \mathbf{K}^l, \mathbf{V}^l$ 分别是第 $l$ 层特征 $\mathbf{Z}^l$ 的高维嵌入，$\sigma(\cdot)$ 是激活函数。

> 得到上下文信息的过程和 `self-attention` 如出一辙，文中在外层加了一个激活函数 $\sigma(\cdot)$ 后再与特征图聚合，本殿下认为不加应该也没有问题。
> $$
> \mathbf{C}^l = 
> 	  \mathrm{softmax} 
> 	  \left (
> 	  \mathrm{Rele}(\mathbf{Q}^l, \mathbf{K}^l) 
> 	  \right )
> 	  \mathbf{V}^l
> $$

### Alignment score function

下表中，列举一些常见的 Attention 机制及其对应的 `Alignment score function`。

| Attention              | Alignment score function                                     | Ref.                                                         |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Content-base attention | $$\mathrm{score}(\mathbf{s}_t, \mathbf{h}_i) = \mathrm{cosine}[\mathbf{s}_t, \mathbf{h}_i]$$ | [Graves2014](https://arxiv.org/abs/1410.5401 "Graves2014")   |
| Additive (*)           | $$\mathrm{score}(\mathbf{s}_t, \mathbf{h}_i) = \mathbf{v}_a^\top \tanh(\mathbf{W}_a[\mathbf{s}_t; \mathbf{h}_i])$$ | [Bahdanau2015](https://arxiv.org/pdf/1409.0473.pdf "Bahdanau2015") |
| Location-Base          | $$\alpha_{t,i} = \mathrm{softmax}(\mathbf{W}_a \mathbf{s}_t)$$<br/>注意：这简化了 `softmax` 对齐，只依赖于目标位置。 | [Luong2015](https://arxiv.org/pdf/1508.04025.pdf  "Luong2015") |
| General                | $$\mathrm{score}(\mathbf{s}_t, \mathbf{h}_i) = \mathbf{s}_t^\top\mathbf{W}_a\mathbf{h}_i$$<br/>其中 $$\mathbf{W}_a$$ 是注意力层中的可训练参数。 | [Luong2015](https://arxiv.org/pdf/1508.04025.pdf)            |
| Dot-Product            | $$\mathrm{score}(\mathbf{s}_t, \mathbf{h}_i) = \mathbf{s}_t^\top\mathbf{h}_i$$ | [Luong2015](https://arxiv.org/pdf/1508.4025.pdf)             |
| Scaled Dot-Product (^) | $$\mathrm{score}(\mathbf{s}_t, \mathbf{h}_i) = \frac{\mathbf{s}_t^\top\mathbf{h}_i}{\sqrt{n}}$$<br/>注：除一个比例因子外，与点积注意力非常相似；其中 $n$ 为隐变量的维数。 | [Vaswani2017](http://papers.nips.cc/paper/7181-attention-is-all-you-need.pdf "Vaswani2017") |

(*) $[;]$ 表示 `concatenate`（Luong, et al., 2015），additive attention 最早出现在 Vaswani, et al., 2017。<br>(^) 添加一个缩放因子 $1/\sqrt{n}$，考虑到输入很大，`softmax` 函数可能有一个非常小的梯度，难以有效学习。

### Asymmetric relevance measurement

在该任务中，第一层的顶点特征向量是关节的空间坐标。因此，即使两个关节在一个动作中高度相关，如果它们的坐标向量几乎是正交的，它们的相关性得分也接近于零，这显然是不恰当的。同样，当计算 $v_i$ 的上下文时， $v_i$ 和其他被集成的顶点对于上下文信息的作用是不同的，所以顶点之间的相关性度量应该是非对称的。


$$
\mathrm{Rele}(\mathbf{Q}^l, \mathbf{K}^l) \neq \mathrm{Rele}(\mathbf{K}^l, \mathbf{Q}^l)
$$


基于此，作者分别使用了内积函数（上表中的 `Dot-Product`）和双线性相关函数（表中的 `General`）的两种高级表示。关联度量中引入的不对称性使得当一个顶点作为接收点或发送点时，可以用不同的方式来考虑它。

## Context Aware Convolution

获得上下文特征图 $\mathbf{C}^l$ 后，应用集成函数 $\mathrm{Inte}(\cdot;\cdot)$ 来合并上下文信息 $\mathbf{C}^l$ 和原始特征图 $\mathbf{H}^l$ 得到上下文感知的特征图 $\mathbf{H}_{\mathrm{c}}^l$，在进行卷积时将全局上下文信息集成。
$$
\begin{align*}
	\mathbf{H}_{\mathrm{c}}^l &= \mathrm{Inte} (\mathbf{H}^l, \mathbf{C}^l) \\
	 	\mathbf{H}^{l+1} &= \sigma(\mathbf{D}^{-\frac{1}{2}} \tilde{\mathbf{A}} \mathbf{D}^{-\frac{1}{2}} \mathbf{H}_{\mathrm{c}}^l \star \mathbf{W})
\end{align*}
$$
本文中，作者提出了一个轻量级和高级的上下文感知方法，其本质在于是否对特征维度进行映射。将特征维度由低维映射到高维与直接使用特征向量相比，增加了灵活性；缺点是增加了模型参数和计算量，难以优化。

###　Edge feature based context-aware GCNs

在这一部分中，介绍如何将所提方法应用于基于边或同时基于顶点和边的GCNs

#### Node-node similarity score

$$
w_{\mathrm{\mathbf{z}}_i^l, \mathrm{\mathbf{z}}_j^l} = \mathrm{softmax}(\mathrm{Rele}(\mathrm{R}(\mathbf{z}_i^l), S(\mathbf{z}_j^l)))
$$

#### Node-edge similarity score

$$
w_{\mathrm{\mathbf{z}}_i^l, \mathrm{\mathbf{e}}_{i, j}^l} = \mathrm{softmax}(\mathrm{Rele}(\mathrm{R}(\mathbf{z}_i^l), S(\mathrm{\mathbf{e}}_{i, j}^l)))
$$

#### Context generation

$$
\mathbf{c}^l = \sigma \left( 
		\sum_{\mathbf{v}_k \in \mathcal{V}} w_{\mathrm{\mathbf{z}}_i^l} G(\mathbf{z}_k) + \sum_{\mathbf{e}_{i,j} \in \mathcal{E}} w_{\mathrm{\mathbf{z}}_i^l} G(\mathbf{e}_{i,j}^l) + \mathbf{b}^l_{\mathrm{c}}
		\right)
$$

# Experiments

## Ablation Study

首先在时间卷积中包含不同帧数（$K$）的模型上进行实验，结果如下：

![Results of models with different $K$](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-6-13/1623558579339-image.png)



对的 light CA-GCN 的不同相关性和整合功能进行消融研究，结果如下：

![Ablation study on different relevance functions and different integration functions on Kinetics.](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-6-13/1623558649614-image.png)



## Studies on Network Size

由于全局上下文信息可以通过叠加多个卷积层来消除扩展接受域的必要性，在本节中，测试了CA-GCNs和降低深度的基线。在这一部分中，使用了可训练相关性得分和串联积分。



![ Performance of CA-GCN and ST-GCN with different number of layers on Kinetics. Time costs are seconds per epoch.](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-6-13/1623558899079-image.png)



## Others

![](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-6-13/1623559116469-image.png)

更多实验结果，点击[【阅读原文】](https://mp.weixin.qq.com/s?__biz=MzUyMTE2NDYxMQ==&mid=2247489364&idx=1&sn=24717494138e8784d2c1c0aff34615cc&chksm=f9de1578cea99c6ee49988cc907825a4cd21b4456eabc282b31d9d60f4a455e0ceaa36536696&token=2040274357&lang=zh_CN#rd)。

# 小结

本文提出了一种基于全局上下文的图网络用于动作分类任务，从动作的全局性事实出发，利用 `self-attention` 的思想设计网络，值得借鉴！

