# Lagrange 乘子

## 代数表述

设 $(x^*,y^*)$ 是 $f(x,y)$ 在曲线 $g(x,y)=c$ 上的局部极小 / 极大点，且
$$
\begin{equation*}
\nabla g(x^*,y^*) \neq 0
\end{equation*}
$$
则存在 $\lambda^*$ 使得
$$
\begin{equation*}
\begin{aligned}
g(x^*,y^*) &= c \\
\nabla f(x^*,y^*) + \lambda^* \nabla g(x^*,y^*) &= 0
\end{aligned}
\end{equation*}
$$
![函数 $f(x,y)$](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2020-12-25/1608888227973-f_x_y.png)

## 几何意义

存在曲面 $z = f(x,y)$ 的等高线 
$$
\begin{equation*}
f(x,y) = f(x^*,y^*)
\end{equation*}
$$
与曲线 $g(x,y)=c$ 在最优解 $(x^*,y^*)$ 处有**公共切线**。

如下图所示，上述结论等价于 $\nabla f(x^*,y^*)$ 与 $\nabla g(x^*,y^*)$ 共线，或 $\nabla f(x^*,y^*) + \lambda^* \nabla g(x^*,y^*) = 0$。

![Lagrange 乘子的几何直观](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2020-12-25/1608888020530-Lagrange_multiplier.png)

## 等式约束问题的 Lagrange 乘子法

考虑等式约束问题
$$
\begin{equation*}\begin{aligned}
\min ~~& f({x})\\
\text{s.t.} ~~& c_i(x) = 0, i = 1,2,\cdots,m
\end{aligned}\end{equation*}
$$

### 性质

> **定理**	记 $a_i = \nabla c_i(x) = a_i(x)$， 设 $x^*$ 是问题的解，且 $a_i^*, i\in \mathcal{E}$ **线性无关**，则存在 $\lambda_1^*,\lambda_2^*,\cdots,\lambda_m^*$ 满足
> $$
> \begin{equation*}
> \begin{aligned}
> g^* + \lambda_1^* a_1^* + \cdots+\lambda_m^*a_m^* & = 0\\
> c_i(x^*) &=0
> \end{aligned}
> \end{equation*}
> $$
> 其中，$i= 1，2,\cdots, m$。

构造 Lagrange 函数
$$
\mathcal{L}(x,\lambda) = f(x) + \lambda_1c_1(x) + \cdots+\lambda_mc_m(x)
$$
一阶必要条件为
$$
\nabla\mathcal{L}(x^*,\lambda^*) = 0
$$
其中，$\nabla = \begin{pmatrix} \nabla_x \\ \nabla_y \end{pmatrix}$ 是 $m + n$ 维向量空间的导算子。

## 约束优化

考虑问题
$$
\begin{equation*}\begin{aligned}
\min ~~& f({x})\\
\text{s.t.} ~~& c_i(x) = 0, i \in \mathcal{E} \\
~~& c_i(x) = 0, i \in \mathcal{I} \\
\end{aligned}\end{equation*}
$$
其中，$\mathcal{E}$ 和 $\mathcal{I}$ 为有限等式和不等式指标集。

> 在设计和分析算法时，通常假设 $f(x)$ 和 $c_i(x)$ 连续可微（**二次连续可微**或**偏导数存在并且连续**）！

### 可行域

上述约束问题的可行域为
$$
\begin{equation*}
\Omega \mathop{\colon=} \{ 
x \in \mathbb{R} \colon c_i(x) = 0,i\in \mathcal{E}; c_i(x)=0,i\in \mathcal{I}
\}
\end{equation*}
$$
则约束问题化为
$$
\begin{equation*}
\min_{x\in\Omega} f(x)
\end{equation*}
$$
记 
$$
\begin{equation*}
\begin{aligned}
\nabla f(x) ~~~~~~~~~& \Leftrightarrow~~~~~~~~~~~ g(x) \\
\nabla^2 f(x) ~~~~~~~~~& \Leftrightarrow~~~~~~~~~~~ G(x) \\
\nabla c_i(x) ~~~~~~~~~& \Leftrightarrow~~~~~~~~~~~ a_i(x) \\
a_i(x^*) ~~~~~~~~~& \Leftrightarrow~~~~~~~~~~~ a_i^*\\
a_i(x') ~~~~~~~~~& \Leftrightarrow~~~~~~~~~~~ a_i'
\end{aligned}
\end{equation*}
$$
![约束的法向量](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2020-12-25/1608889954719-constraint_normal.png)

## 积极约束

**例**	考虑优化问题
$$
\begin{equation*}
\begin{aligned}
\min ~~& f(x) = -x_1 - x_2 \\
\mathrm{s.t.} ~~& c_1(x) = x_1^2 -x_2 \leq 0\\
~~& c_2(x) = x_1^2 + x_2^2 - 1 \leq 0
\end{aligned}
\end{equation*}
$$
![积极约束和非积极约束](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2020-12-25/1608894584945-active.png)

定义积极约束 / 紧约束集为
$$
\mathcal{A}' = \mathcal{A}(x') = \{ i:c_i(x') = 0 \}
$$
若 $x'$ 可行，即 $x' \in \Omega$，则

- 显然有 $\mathcal{E} \subset \mathcal{A}$

- 积极不等式约束
  $$
  \begin{equation*}
  \mathcal{I}' = \mathcal{I}(x') = \{i: c_i(x) = 0,i \in \mathcal{I}\}
  \end{equation*}
  $$

综上所述，$\mathcal{A}'  = \mathcal{E} \cup \mathcal I'$。

如下图所示，Lagrange 函数 $\mathcal{L}(x,\lambda) = f(x) + \lambda c(x)$，考虑不等式约束 $c(x) \leq 0$ 是否为积极约束

- 左图中，一阶条件为
  $$
  \begin{equation*}
  \nabla_x \mathcal{L}(x^*,\lambda^*) = 0, {\color{blue} \lambda^* = 0},c(x^*) < 0
  \end{equation*}
  $$
  因此为非积极约束。

- 右图中，一阶条件为
  $$
  \begin{equation*}
  \nabla_x \mathcal{L}(x^*,\lambda^*) = 0, {\color{blue} \lambda^* > 0},c(x^*) = 0
  \end{equation*}
  $$
  因此为积极约束。

![积极约束和非积极约束](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2020-12-25/1608894868712-activate_2.png)

因此，我们必须认识到的**是只有 $x^*$ 处的积极约束 $\mathcal{A}'$ 对最优性有影响**，且将任何非积极约束的 Lagrange 乘子取 0。

## KKT 条件

构造问题
$$
\begin{equation*}\begin{aligned}
\min ~~& f({x})\\
\text{s.t.} ~~& c_i(x) = 0, i \in \mathcal{E} \\
~~& c_i(x) = 0, i \in \mathcal{I} \\
\end{aligned}\end{equation*}
$$
Lagrange 函数
$$
\begin{equation*}
\mathcal{L}(x,\lambda) = f(x) + \sum_{i \in \mathcal{E}} \lambda_i c_i(x) + \sum_{j \in \mathcal{I}} \lambda_j c_j(x)
\end{equation*}
$$
**定理**（一阶必要条件）	若 $x^*$ 是局部极小点且 $x^*$ 处正则性假设 
$$
\begin{equation*}
F^* \cap D^* = \mathcal{F}^* \cap \mathcal{D}^*
\end{equation*}
$$
成立，则存在 Lagrange 乘子 $\lambda^*$ 使得 $x^*,\lambda^*$ 满足
$$
\begin{equation*}
\begin{aligned}
\nabla_x \mathcal{L}(x^*,\lambda^*) & = 0\\
c_i(x^*) & = 0, i \in \mathcal{E} \\
c_i(x^*) & \leq 0, i \in \mathcal{I} \\
\lambda_i^* &\geq 0, i \in \mathcal{I} \\
\lambda_i^*c_i(x^*) & = 0, i \in \mathcal{I}
\end{aligned}
\end{equation*}
$$

- 称 $\lambda^*$ 为与 $x^*$ 对应的 Lagrange 乘子向量
- 称条件 $\lambda_i^*c_i(x^*) = 0$  为互补条件、若不存在 $\forall i$ 使得 $c_i(x^*) = 0, \lambda_i^* = 0$ 则为严格互补条件
- 称这些条件为 Karush-Kuhn-Tucker 条件（KKT 条件），满足这些条件的点为 KKT 点

- 正则性假设是对向量 $a' ~ (i \in \mathcal{A}')$ 的进一步放松

**例** 1	考虑优化问题
$$
\begin{equation*}
\begin{aligned}
\min ~~& x_1 + x_2 \\
\mathrm{s.t.} ~~& x_1^2 + x_2^2 - 2 = 0
\end{aligned}
\end{equation*}
$$
![例 1 不同可行点处约束函数和目标得梯度](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2020-12-25/1608906245670-example_1.png)

从上图可知，在 $x^* = (-1，-1)^T$ 处，目标函数的梯度 $\nabla f$ 和约束的梯度 $\nabla c_1$ 反向共线，解 $g + \lambda a_1 = 0$，得 $\lambda^* = \frac{1}{2}$。

**例** 2	考虑只有一个不等式约束得优化问题
$$
\begin{equation*}
\begin{aligned}
\min ~~& x_1 + x_2 \\
\mathrm{s.t.} ~~& x_1^2 + x_2^2 - 2 \leq 0
\end{aligned}
\end{equation*}
$$
![例 2 不同可行点处约束函数和目标得梯度](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2020-12-25/1608906261899-example_2.png)

该问题得可行域由圆周上的点和其内部点组成，在圆周上仍然有和**例** 1 中相同得解。

**例** 3	考虑两个不等式约束得优化问题
$$
\begin{equation*}
\begin{aligned}
\min ~~& x_1 + x_2 \\
\mathrm{s.t.} ~~& x_1^2 + x_2^2 - 2 \leq 0 \\
~~& x_2 \geq 0
\end{aligned}
\end{equation*}
$$

- 考虑点 $(-{\sqrt{2}},0)$

![解](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2020-12-25/1608907288296-example_3_1.png)

不等式积极约束 $\mathcal{I}^* = \{1,2\}$，解得 $\lambda^* = (\frac{1}{2\sqrt{2}},1)^T$，因此是 KKT 点。

- 考虑点 $({\sqrt{2}},0)$

![非解](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2020-12-25/1608907300834-example_3_2.png)

不等式积极约束 $\mathcal{I}^* = \{2\}$，易得非 KKT 点。

> KKT 条件是必要条件
>
> - 仅当所考虑的点是目标函数和约束函数的可微点，且正则性条件成立时，KKT 点才是最优解的必要条件！
>
> - KKT 条件仅仅是一个必要条件，考虑问题
>   $$
>   \begin{equation*}
>   \begin{aligned}
>   \min ~~& x^3 \\
>   \mathrm{s.t.} ~~& -1 \leq x \leq 1
>   \end{aligned}
>   \end{equation*}
>   $$
>   易得 $x^*$ 是 KKT 点，也满足线性约束规范（LCQ），但 $x^* = 0 $ 不是局部极小点。

## 参考文献

[1]  刘红英，夏勇，周永生. 数学规划基础，北京，北京航空航天大学出版社，2012.