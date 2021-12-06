---
layout: post
category: DL
title: 迁移学习简介
tagline: by Kanglei Zhou
author: Kanglei Zhou
tags: 
  - 迁移学习
published: true
---

[公众号阅读更佳](https://mp.weixin.qq.com/s?__biz=MzUyMTE2NDYxMQ==&mid=2247490001&idx=1&sn=454b99fae03147c916e663794b530874&chksm=f9de1bfdcea992eb781e1ba4d49508742d9c4948edb298840fadd6b3315cdf3612c7d6f81134&token=514611700&lang=zh_CN#rd)

迁移学习是机器学习的前沿研究方向之一，其目标是[将某个领域或任务上学习到的知识或模式应用到不同的但相关的领域或问题中](http://ise.thss.tsinghua.edu.cn/~mlong/doc/phd-thesis-mingsheng-long.pdf "龙明盛. 迁移学习问题与方法研究[D]. 清华大学, 2014")。

与半监督学习、主动学习等标注数据稀缺解决风范的本质不同在于，迁移学习**放宽了训练数据和测试数据服从独立同分布这一假设**，从而使得参与学习的领域或任务可以服从不同的边缘概率分布或条件概率分布。迁移学习的主要思想是，从相关的辅助领域中迁移标注数据或知识结构、完成或改进目标领域或任务的学习效果，其工作原理如下所示：

![迁移学习原理：从辅助领域的大量标注数据中迁移知识、改进目标领域学习任务](https://gitee.com/ZhouKanglei/jidianxia/raw/master/2021-12-6/1638779907214-%E8%BF%81%E7%A7%BB%E5%AD%A6%E4%B9%A0%E5%8E%9F%E7%90%86%EF%BC%9A%E4%BB%8E%E8%BE%85%E5%8A%A9%E9%A2%86%E5%9F%9F%E7%9A%84%E5%A4%A7%E9%87%8F%E6%A0%87%E6%B3%A8%E6%95%B0%E6%8D%AE%E4%B8%AD%E8%BF%81%E7%A7%BB%E7%9F%A5%E8%AF%86%E3%80%81%E6%94%B9%E8%BF%9B%E7%9B%AE%E6%A0%87%E9%A2%86%E5%9F%9F%E5%AD%A6%E4%B9%A0%E4%BB%BB%E5%8A%A1.png)



# 动机

- 解决标注数据稀缺性

**标注数据稀缺性**会导致经典监督学习出现严重的**过拟合问题**，虽然传统半监督学习、协同训练、主动学习等也可以解决数据稀缺性，但它们都要求目标领域中存在相当程度的标注数据；当标注数据十分稀缺且获取代价太大时，仍然需要从辅助领域迁移知识来提高目标领域学习效果。

- 非平稳泛化误差分析

具备理论保证是统计机器学习得以成功的关键因素之一。然而在非平稳环境中，不同数据领域不再服从独立同分布假设，使得经典学习理论不再成立，这给异构数据分析挖掘带来了理论上的风险。广义上看，迁移学习是经典学习在非平稳环境下的推广；换句话说，但凡**经典学习不能取得很好学习效果时**均**可能是因为训练数据和测试数据之间存在概率分布漂移**。

# 定义

迁移学习涉及领域（Domain）和任务（Task）两个重要概念：

- 域 $\mathcal{D}$ 定义为由 $d$ 维特征空间 $\mathcal{X}$ 和边缘概率分布 $P(\pmb{x})$ 组成，即 
$$
\mathcal{D} = 
\{
\mathcal{X}, P(\pmb{x})
\}
,
\pmb{x} \in \mathcal{X}$$
- 给定域 $\mathcal{D}$，任务 $\mathcal{T}$ 定义为由类别空间 $\mathcal{Y}$ 和预测模型 $f(\pmb{x})$ 组成，即
$$\mathcal{T} = \{\mathcal{Y}, f(\pmb{x})\}，y \in \mathcal{Y}$$
- 按统计观点预测模型 $f(\pmb{x}) = P(y\mid \pmb{x})$ 解释为条件概率分布。

> **定义** （*迁移学习*）  给定**源**域 $\mathcal{D}_{\mathrm{s}} = \{(\pmb{x}^{\mathrm{s}}_1, y^{\mathrm{s}}_1),(\pmb{x}^{\mathrm{s}}_2, y^{\mathrm{s}}_2),\cdots,(\pmb{x}^{\mathrm{s}}_{n_\mathrm{s}}, y^{\mathrm{s}}_{n_\mathrm{s}})\}$ 及其对应的源任务 $\mathcal{T}_{\mathrm{s}}$ 和**目标**域 $\mathcal{D}_{\mathrm{t}} = \{(\pmb{x}^{\mathrm{t}}_1, y^{\mathrm{t}}_1),(\pmb{x}^{\mathrm{t}}_2, y^{\mathrm{t}}_2),\cdots,(\pmb{x}^{\mathrm{t}}_{n_\mathrm{t}}, y^{\mathrm{t}}_{n_\mathrm{t}})\}$ 及其对应的目标任务 $\mathcal{T}_{\mathrm{t}}$，迁移学习的目标是在 $\mathcal{D}_{\mathrm{s}} \neq \mathcal{D}_{\mathrm{t}}$ 或 $\mathcal{T}_{\mathrm{s}} \neq \mathcal{T}_{\mathrm{t}}$ 条件下，降低目标域预测模型 $f_{\mathrm{t}}(\pmb{x}^{\mathrm{t}})$ 的泛化误差。


根据特征空间、类别空间、边缘概率分布、条件概率分布的异同，迁移学习可以进一步分为多个子类。

# 分类

## 基于标注的迁移学习分类

如下图所示，根据域标注数据有无，迁移学习可以分成三类：**推导迁移学习**（inductive transfer learning），**转导迁移学习**（tranductive transfer learning）和**无监督迁移学习**（unsupervised transfer learning）。

![An Overview of Different Settings of Transfer[](https://www.cse.ust.hk/~qyang/Docs/2009/tkde_transfer_learning.pdf "Pan S J, Yang Q. A survey on transfer learning[J]. IEEE Transactions on Knowledge and Data Engineering, 2010")](https://gitee.com/ZhouKanglei/jidianxia/raw/master/2021-12-6/1638779220739-An%20Overview%20of%20Different%20Settings%20of%20Transfer.png)

> **描述推导迁移学习** 在 $\mathcal{T}_{\mathrm{s}} \neq \mathcal{T}_{\mathrm{t}}$ 时，推导迁移学习使用源领域 $\mathcal{D}_{\mathrm{s}}$ 和 $\mathcal{T}_{\mathrm{s}}$ 中的知识提升或优化目标领域 $\mathcal{D}_{\mathrm{t}}$ 中目标预测函数 $f_{\mathrm{t}}$ 的学习效果。



在推导迁移学习中，源任务 $\mathcal{T}_{\mathrm{s}}$ 与目标任务 $\mathcal{T}_{\mathrm{t}}$ 一定不同，目标域 $\mathcal{D}_{\mathrm{t}}$ 与源域 $\mathcal{T}_{\mathrm{s}}$ 可以相同，也可以不同。在这种情况下，目标域需要一部分带标记的数据用于建立目标领域的预测函数 $f_{\mathrm{t}}(\cdot)$。

- 当源域中有很多标记样本时，推导迁移学习与多任务学习（multitask learning）类似。

**推导迁移学习 vs 多任务学习**
通过从源域迁移知识，推导迁移学习只注重提升目标领域的效果；但多任务学习注重同时提升源领域和目标领域的效果。

- 当源域没有标记样本时，推导迁移学习与自学习（self-taught learning）类似。



> **描述转导迁移学习** 在 $\mathcal{T}_{\mathrm{s}} = \mathcal{T}_{\mathrm{t}}$ 且 $\mathcal{D}_{\mathrm{s}} \neq \mathcal{D}_{\mathrm{t}}$ 时，转导迁移学习使用源领域 $\mathcal{D}_{\mathrm{s}}$ 和 $\mathcal{T}_{\mathrm{s}}$ 中的知识提升或优化目标领域 $\mathcal{D}_{\mathrm{t}}$ 中目标预测函数 $f_{\mathrm{t}}$ 的学习效果。

在转导迁移学习中，源任务 $\mathcal{T}_{\mathrm{s}}$ 与目标任务 $\mathcal{T}_{\mathrm{t}}$ 相同，但域 $\mathcal{D}_{\mathrm{s}}$ 和 $\mathcal{D}_{\mathrm{t}}$ 不同。这种情况下，源域有大量标记样本，但目标领域没有标记样本。根据 $\mathcal{D}_{\mathrm{s}}$ 和 $\mathcal{D}_{\mathrm{t}}$ 的不同，可以把转到学习分为两个类：

- 源域和目标域特征空间不同，即 $\mathcal{X}_{\mathrm{s}}$ 不等于 $\mathcal{X}_{\mathrm{t}}$，对应 domain adaption

- 特征空间相同，但边缘概率不同，即 $P(\pmb{x}^{\mathrm{s}})$ 不等于 $P(\pmb{x}^{\mathrm{t}})$，对应 covariate shift

> **描述无监督移学习** 在 $\mathcal{T}_{\mathrm{s}} = \mathcal{T}_{\mathrm{t}}$ 且标签空间 $\mathcal{Y}_{\mathrm{s}}$ 和 $\mathcal{Y}_{\mathrm{t}}$ 不可观测时，无监督迁移学习使用源领域 $\mathcal{D}_{\mathrm{s}}$ 和 $\mathcal{T}_{\mathrm{s}}$ 中的知识提升或优化目标领域 $\mathcal{D}_{\mathrm{t}}$ 中目标预测函数 $f_{\mathrm{t}}$ 的学习效果。

在无监督迁移学习中，目标任务与源任务不同但却相关。此时，无监督迁移学习主要解决目标领域中的无监督学习问题，类似于传统的**聚类**、降维和密度估计等机器学习问题。

![迁移学习概述](https://gitee.com/ZhouKanglei/jidianxia/raw/master/2021-12-6/1638783389559-%E8%BF%81%E7%A7%BB%E5%AD%A6%E4%B9%A0%E6%A6%82%E8%BF%B0.png)



## 基于特征空间的迁移学习分类
按特征空间、类别空间、边缘分布、条件分布等问题因素在领域间的异同，迁移学习可大致地划分为两大类。

![迁移类型分类体系](https://gitee.com/ZhouKanglei/jidianxia/raw/master/2021-12-6/1638788865904-%E8%BF%81%E7%A7%BB%E7%B1%BB%E5%9E%8B%E5%88%86%E7%B1%BB%E4%BD%93%E7%B3%BB.png)

### 同构迁移学习：特征维度相同分布不同
根据边缘概率分布和条件概率分布异同，同构迁移学习可进一步分为数据集偏移、领域适配、多任务学习三种子类型。



### 异构迁移学习：特征维度不同或特征本身就不同
根据领域间特征空间和类别空间的异同，异构迁移学习包括异构特征空间和异构类别空间两种子类型。


# 应用

基于上述理论方面的原因，迁移学习已经被广泛应用于很多重要实际问题中如自然语言处理、计算机视觉、医疗健康和生物信息学等。


![跨仪器设备医疗影像识别：从 CT、X 射线标注图像到 MRI 核磁共振图像识别](https://gitee.com/ZhouKanglei/jidianxia/raw/master/2021-12-6/1638789280325-%E8%B7%A8%E4%BB%AA%E5%99%A8%E8%AE%BE%E5%A4%87%E5%8C%BB%E7%96%97%E5%BD%B1%E5%83%8F%E8%AF%86%E5%88%AB%EF%BC%9A%E4%BB%8E%20CT%E3%80%81X%20%E5%B0%84%E7%BA%BF%E6%A0%87%E6%B3%A8%E5%9B%BE%E5%83%8F%E5%88%B0%20MRI%20%E6%A0%B8%E7%A3%81%E5%85%B1%E6%8C%AF%E5%9B%BE%E5%83%8F%E8%AF%86%E5%88%AB.png)
