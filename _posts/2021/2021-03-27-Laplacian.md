---
layout: post
category: Laplace
title: 【流体】散度与拉普拉斯
tagline: by 奇凡
tags: 
  - Laplace
published: true

---



# 散度算子 $\nabla$




二维笛卡尔坐标系下：


![](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-3-25/1616661327348-image.png)

三维笛卡尔坐标系下：


![](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-3-25/1616661379472-image.png)


散度算子作用在矢量场上，其**物理含义是描述矢量场中某点发散或是汇聚的强弱程**度。



![散度示意图](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-3-25/1616662418289-image.png)

# 拉普拉斯算子 $\nabla \cdot \nabla / \nabla ^2 / \Delta$

拉普拉斯算符(Laplacian) $\nabla^2$ 被定义为梯度的散度($\nabla \cdot \nabla$)，有点类似于单变量函数的2阶导数。2阶导数与拉普拉斯算符都是描述速度的变化(加速度)，它们的差别大约可类比于弦和鼓的差别。

![2维鼓膜之于Laplacian如同1维琴弦之于2阶导数](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-3-25/1616663350463-image.png)


函数值稳定的增长或下架，变化的速度保持稳定不变，梯度的散度等于0，Laplacian就等于0；相反，如果在逼近本地最大/最小值时，变化的速度会下降或增长，梯度的散度/Laplacian也会变小或变大。



![image](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-3-25/1616664285674-image.png)




二维笛卡尔坐标系下：


$$
\nabla \cdot \nabla f  = \frac{\partial^2 f}{\partial x^2} + \frac{\partial^2 f}{\partial y^2} = 0
$$

可理解为：在 $x,y$ 方向上的加速度和为0。使用拉普拉斯算符 $\nabla$ 来表示拉普拉斯方程名正言顺且简洁优美： $\nabla^2 u = 0$ 。




三维笛卡尔坐标系下：

$$
\nabla \cdot \nabla f  = \frac{\partial^2 f}{\partial x^2} + \frac{\partial^2 f}{\partial y^2} + \frac{\partial^2 f}{\partial z^2} = 0
$$

拉普拉斯算子是标量函数梯度场的散度，**物理含义是一个量离它周围的平均值有多远**。

拉普拉斯算子也可作用于向量场或矩阵场，相当于把拉普拉斯算子分别作用于每一个分量。





