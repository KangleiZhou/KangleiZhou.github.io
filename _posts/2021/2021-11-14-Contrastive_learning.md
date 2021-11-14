---
layout: post
category: DL
title: 【Contrastive learning】一文读懂对比学习
tagline: by Kanglei Zhou
author: Kanglei Zhou
tags: 
  - Contrastive learning
published: true

---

何恺明大神新作[MAE](https://arxiv.org/pdf/2111.06377.pdf "Masked Autoencoders Are Scalable Vision Learners")震惊了学术圈，在阅读学习的过程中遇到了一些知识漏洞。本期，了解和学习[对比学习（Contrastive Learning）](https://ankeshanand.com/blog/2020/01/26/contrative-self-supervised-learning.html "对比自监督学习")。对比自监督学习技术是一类很有前途的方法，它通过学习编码使两个事物相似或不同的东西来构建表示。

# 对比学习原理

对比学习的主要思想是学习表示，使相似的样本保持相近，而不同的样本则相差很远。对比学习可以应用于有监督和无监督的数据，并已被证明在各种视觉和语言任务中取得良好的表现。[总结起来就是](https://mp.weixin.qq.com/s/EOlXjdd1gCruiF-B1JoIgg "对比学习火了")：

> 对一条样本$x_1$通过**数据增强**得到$x_2$，那么这就是一对正样本对，和其他样本就是负样本对。
>
> ![对比学习](https://files.mdnice.com/user/3650/41ea5299-4d76-4c53-bb11-891b77e57a18.png)
>
> 通常的做法就是：一个`batch`假设大小是$M$(假设其中一个样本是$x_1$)，那么通过数据增强得到$2M$($x_1$数据增强得到$x_2$)，正样本就是$x_1$和$x_2$，负样本就是$x_1$和`batch`内其他的样本。
>
> 其实大家可以类比到其他领域，说不定也是个创新点，文本领域大家发paper的思路点，主要就是围绕在
>
> (1) 数据增强的方式上：比如替换词，dropout等等
>
> (2)一些其他思路应用上：比如多模态，$x_1$是图片，$x_2$是图片对应的文字等等。

# SimCLRv2

在本文中，我将重点介绍**SimCLRv2**，这是 Google Brain 团队最近提出的最先进的对比学习方法之一。[我们](https://towardsdatascience.com/understanding-contrastive-learning-d5b19fd96607 "了解对比学习")可以进一步剖析这种对比学习方法，将其分解为三个主要步骤：**数据增强、编码和损失最小化。**

## 数据增强

随机执行以下增强的任意组合：裁剪、调整大小、颜色失真、灰度。我们*对批次中的每张图像*执行*两次*此操作*，*以创建一**对正对**的两个增强图像。

![来源：Amit Chaudhary 的 Illustrated SimCLR 框架](https://files.mdnice.com/user/3650/7f42b9cd-2590-46df-8c6f-61b15851e4f1.gif)

# 编码

然后，我们用我们的BIG-CNN的神经网络，我们可以认为作为一个简单的*功能*，即
$$
\mathbf{h} = f(\mathbf{x})
$$
*其中$\mathbf{x}$是我们增强图像之一，*以*编码*我们两个图像作为矢量表示。

![](https://files.mdnice.com/user/3650/28d1b4b2-e177-40e8-8f12-f9ba81a0f640.png)

然后将 CNN 的输出输入到一组 Dense Layers 中，
$$
\mathbf{z} = g(\mathbf{h})
$$
以将数据转换到另一个空间。经验表明，这个额外的步骤可以提高性能

通过将我们的图像压缩为潜在空间表示，该模型能够*学习*图像*的高级特征*。

事实上，随着我们继续训练模型以最大化相似图像之间的向量相似度，我们可以想象模型正在学习潜在空间中相似数据点的*集群*。

例如，猫表示将更接近，但与狗表示更远，因为这是我们训练模型学习的内容。

## 损失最小化

现在我们有两个向量$\mathbf{z}$，我们需要一种方法来**量化它们之间的相似性。**

![](https://files.mdnice.com/user/3650/1ac073cf-4c69-48c6-be70-ef618f34a701.png)

由于我们正在比较两个向量，因此自然选择**余弦相似度，**它基于*空间中两个向量之间的角度*。

![余弦相似度](https://files.mdnice.com/user/3650/1cc44dd8-e181-462d-b20b-7e91777455da.png)

我们还需要一个可以最小化的**损失函数**。一种选择是 NT-Xent。

我们首先计算*两个增强图像相似*的概率。

![](https://files.mdnice.com/user/3650/e5302ebe-1f00-4108-992b-218c6f16266c.png)

请注意，分母是 $e^{\mathrm{similarity}}$（*所有对*，包括**负对**）的总和。通过在我们的增强图像和我们批次中的所有其他图像之间创建**对**来获得**负对**。

最后，我们将该值包装在$-\log(\cdot)$ 中，*以便*最小化*此损失函数对应于*最大化两个增强图像相似的概率。

![](https://files.mdnice.com/user/3650/f27485a9-d025-4730-934e-6a29ed68dfea.png)

# 应用

当我们的*标签很少*，或者很难获得特定任务的标签时，我们希望能够同时使用标记数据*和未标记数据*来优化模型的性能和学习能力。这就是**半监督学习**的定义**。**

在文献中越来越受欢迎的一种方法是*无监督的预训练、有监督的微调、知识蒸馏*范式。

![](https://files.mdnice.com/user/3650/c4638a36-05ac-4d62-bd34-8cb9ee4e3d95.png)

在这个范式中，自监督对比学习方法是一个关键的“预处理”步骤，它允许 Big CNN 模型（即 ResNet-152）在尝试使用有限标记的图像对图像进行分类之前首先从未标记的数据中学习一般特征数据。

Google Brain 团队证明，这种半监督学习方法的*标签效率非常高*，并且更大的模型可以带来更大的改进，尤其是对于低标签分数。

![随着模型变大，模型在各种标签分数下的 Top-1 准确度](https://files.mdnice.com/user/3650/fa9dd534-4a80-4354-a805-3e82a4f1d5c5.png)

有趣的是，类似的自监督方法已经广泛用于自然语言处理领域。