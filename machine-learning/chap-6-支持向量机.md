# 线性可分情况
> 在样本空间中寻找一个超平面, 将不同类别的样本分开.

```ad-question
title: 将训练样本分开的超平面可能有很多, 哪一个好呢?
选择**正中间**的超平面将样本分开: 容忍性好, 鲁棒性高, 泛化能力强.
![[Pasted image 20221026195421.png|200]]
```


假设超平面方程为 $\boldsymbol{w}^\mathrm{T} \boldsymbol{x} + b = 0$, 则点 $\boldsymbol{x}_{A}$ 与超平面之间的距离为 $r = \dfrac{\left| \boldsymbol{w}^\mathrm{T}\boldsymbol{x}_{A} + b \right|}{\| \boldsymbol{w} \|}$.

假设超平面能够将训练样本正确分类, 即对于 $(x_{i}, y_{i})\in D$, 有
$$
\begin{cases}
\boldsymbol{w}^\mathrm{T} \boldsymbol{x} + b > 0 & y_{i} = +1 \\
\boldsymbol{w}^\mathrm{T} \boldsymbol{x} + b < 0 & y_{i} = -1
\end{cases}
\implies
y_{i} (\boldsymbol{w}^\mathrm{T} \boldsymbol{x} + b) > 0
$$
且上式与向量 $\boldsymbol{w}' = (\boldsymbol{w}; b)$ 的长度无关, 因此若对所有 $i$ 都有 $y_{i} (\boldsymbol{w}^\mathrm{T} \boldsymbol{x} + b) \geq \epsilon$, 则令 $\boldsymbol{w}' \gets \dfrac{\boldsymbol{w}'}{\epsilon}$, 得到
$$
y_{i}(\boldsymbol{w}^\mathrm{T} \boldsymbol{x}_{i} + b) \geq 1 \iff
\begin{cases}
\boldsymbol{w}^\mathrm{T} \boldsymbol{x}_{i} + b \geq +1 & y_{i} = +1 \\
\boldsymbol{w}^\mathrm{T} \boldsymbol{x}_{i} + b \leq -1 & y_{i} = -1
\end{cases}
\tag{1}
$$
如下图所示, 距离超平面最近的几个训练样本点使得上式 (1) 的等号成立, 它们被称为**支持向量** support vector, 两个异类支持向量到超平面的距离之和被称为**间隔** margin, 为
$$
\gamma = \frac{2}{\|\boldsymbol{w}\|}
$$
![[Pasted image 20221026195355.png|300]]

```ad-note
title: 支持向量机的基本型
目标是找到具有**最大间隔** maximum margin 的划分超平面, 也就是要找到满足式 (1) 中约束的参数 $\boldsymbol{w}$ 和 $b$, 使得 $\gamma$ 最大, 即
$$
\max_{\boldsymbol{w}, b} \frac{2}{\|\boldsymbol{w}\|} \quad \text{s.t.} \quad y_{i}(\boldsymbol{w}^\mathrm{T} \boldsymbol{x}_{i} + b) \geq 1,\ i = 1, 2, \cdots, m
$$
等价于最小化 $\|\boldsymbol{w}\|^2$, 即
$$
\min_{\boldsymbol{w}, b} \frac{1}{2}\|\boldsymbol{w}\|^{2} \quad \text{s.t.} \quad y_{i}(\boldsymbol{w}^\mathrm{T} \boldsymbol{x}_{i} + b) \geq 1,\ i = 1, 2, \cdots, m \tag{2}
$$
这就是**支持向量机** Support Vector Machine 的基本型.
```

> 式 (2) 是一个**凸二次规划** convex quadratic programming 问题. 

```ad-note
title: 寻找对偶问题
使用拉格朗日乘子法, 首先得到广义拉格朗日函数
$$
L(\boldsymbol{w}, b, \boldsymbol{\alpha}) = \frac{1}{2}\|\boldsymbol{w}\|^{2} -  \sum_{i = 1}^{m} \alpha_{i} \left( y_{i}(\boldsymbol{w}^\mathrm{T} \boldsymbol{x}_{i} + b) - 1 \right), \quad \boldsymbol{\alpha}\succcurlyeq 0
$$
那么式 (2) 可转换为其**对偶问题** dual problem.
$$
\begin{gather}
\max_{\boldsymbol{\alpha}} g(\boldsymbol{\alpha}) \quad \text{s.t.} \quad \boldsymbol{\alpha} \succcurlyeq 0 \\
g(\boldsymbol{\alpha}) = \min_{\boldsymbol{w}, b} L(\boldsymbol{w}, b, \boldsymbol{\alpha})
\end{gather}\tag{3}
$$
首先求解 $g(\boldsymbol{\alpha})$, 因为 $L(\boldsymbol{w}, b, \boldsymbol{\alpha})$ 是关于 $\boldsymbol{w}$ 和 $b$ 的凸函数, 所以会在其梯度为 0 时取得最小值:
$$
\begin{align}
\frac{ \partial L(\boldsymbol{w}, b, \boldsymbol{\alpha}) }{ \partial \boldsymbol{w} }  = 0 & \implies \boldsymbol{w} = \sum_{i = 1}^{m} \alpha_{i}y_{i}\boldsymbol{x}_{i} \\
\frac{ \partial L(\boldsymbol{w}, b, \boldsymbol{\alpha}) }{ \partial b } = 0 & \implies \sum_{i = 1}^{m} y_{i} \alpha_{i} = 0
\end{align}
$$
代入至式 (3), 得到**对偶问题**
$$
\max_{\boldsymbol{\alpha}} g(\boldsymbol{\alpha}) = \sum_{i = 1}^{m} \alpha_{i} - \frac{1}{2} \sum_{i = 1}^{m} \sum_{j = 1}^{m} \alpha_{i}\alpha_{j}y_{i}y_{j}\boldsymbol{x}_{i}^\mathrm{T}\boldsymbol{x}_{j} \quad \text{s.t.} \quad \boldsymbol{\alpha} \succcurlyeq 0 \text{ and } \boldsymbol{\alpha}^{\mathrm{T}}\boldsymbol{y} = \sum_{i = 1}^{m} \alpha_{i} y_{i} = 0 \tag{4}
$$
解出拉格朗日乘子 $\boldsymbol{\alpha}$ 后, 求出 $\boldsymbol{w}$ 和 $b$ 即可得到模型
$$
f(\boldsymbol{x}) = \boldsymbol{w}^{\mathrm{T}} \boldsymbol{x} + b = \sum_{i = 1}^{m} \alpha_{i}y_{j} \boldsymbol{x}_{i}^\mathrm{T} \boldsymbol{x} + b
$$
```

```ad-note
title: KKT 条件
注意到在求解对偶问题过程中要保证 $y_{i}(\boldsymbol{w}^\mathrm{T} \boldsymbol{x}_{i} + b) \geq 1$, 因此求解过程要满足 Karush-Kuhn-Tucker 条件, 要求
$$
\begin{cases}
\alpha_{i} \geq 0 \\
y_{i}f(\boldsymbol{x}_{i}) - 1 \geq 0 \\
\alpha_{i} (y_{i}f(\boldsymbol{x}_{i}) - 1) = 0
\end{cases}
$$
```

```ad-note
title: SVM 解的稀疏性
KKT 条件说明, 对于任意训练样本 $(\boldsymbol{x}_{i}, y_{i})$, 总有 $\alpha_{i} = 0$ 或 $y_{i}f(\boldsymbol{x}_{i}) = 1$.
- 若 $\alpha_{i} = 0$, 则该样本不会对模型 $f(\boldsymbol{x})$ 有任何影响.
- 若 $y_{i}f(\boldsymbol{x}_{i}) = 1$, 则该样本是一个支持向量.

显示出 SVM 解的**稀疏性**: <u>训练完成后, 大部分的训练样本都不需保留, 最终模型仅与支持向量有关</u>
```

> 通用的二次规划算法规模正比于样本数, 会造成很大开销.

````ad-code
title: SMO 算法
Sequential Minimum Optimization 是一个高效的求解式 (4) 的算法, 其基本思想是不断执行如下两个步骤直至收敛
1. **选取**一对需更新的变量 $\alpha_{i}$ 和 $\alpha_{j}$.
2. **固定** $\alpha_{i}$ 和 $\alpha_{j}$ 以外的参数, **求解**对偶问题更新 $\alpha_{i}$ 和 $\alpha_{j}$.

```ad-question
title: 如何选取两个变量?
若 $\alpha_{i}$ 和 $\alpha_{j}$ 有一个违反了 KKT 条件，目标函数就会在迭代后增大. KKT 条件违背的程度越大，则变量更新后可能导致的目标函数值增幅越大. 因此
- 第一个变量选取**违背 KKT 条件程度最大**的变量
- 第二个变量应选择使目标函数值增长最快的变量, 但是这样需要的时间代价过高.
	- 采用启发式: 选择**与第一个变量的间隔最大**的变量.
```
```ad-question
title: 如何更新变量?
因为固定了 $\alpha_{i}$ 和 $\alpha_{j}$ 以外的参数, 记 $\mathbf{K}_{ij} = \boldsymbol{x}_{i}^{\mathrm{T}}\boldsymbol{x}_{j}$, 式 (4) 退化为
$$
\begin{gather}
\max_{\alpha'_{i}, \alpha'_{j}} \alpha'_{i} + \alpha'_{j} - \frac{1}{2} \left( \alpha'^{2}_{i}\mathbf{K}_{ii} + \alpha'^{2}_{j}\mathbf{K}_{jj} +  2 \alpha'_{i} \alpha'_{j} y_{i} y_{j} \mathbf{K}_{ij} + 2 \alpha'_{i} y_{i} \sum_{k \neq i, j} 
 \alpha_{k} y_{k} \mathbf{K}_{ik} + 2 \alpha'_{j} y_{j} \sum_{k \neq i, j} \alpha_{k} y_{k} \mathbf{K}_{jk}\right) \\
\text{s.t.} \quad \alpha'_{i}y_{i} + \alpha'_{j}y_{j} = \alpha_{i}y_{i} + \alpha_{j}y_{j} \text{ and } \alpha'_{i} \geq 0 \text{ and } \alpha'_{j} \geq 0
\end{gather}
$$
记 $c = \alpha_{i}y_{i} + \alpha_{j}y_{j}$, 则 $\alpha'_{j}y_{j} = c - \alpha'_{i}y_{i}$, 代入上式, 进一步退化为
$$
\begin{gather}
\min_{\alpha'_{i}} \alpha'^{2}_{i} \left( \mathbf{K}_{ii} + \mathbf{K}_{jj} - 2 \mathbf{K}_{ij} \right) + 2 \alpha'_{i} \left( y_{i} \sum_{k = 1}^{m} \alpha_{k}y_{k} \left( \mathbf{K}_{ik} - \mathbf{K}_{jk} \right) - 1 - \alpha_{i} \left( \mathbf{K}_{ii} + \mathbf{K}_{jj} - 2 \mathbf{K}_{ij} \right) + y_{i}y_{j} \right)    \\
\text{s.t.} \quad  \alpha'_{i} \geq 0 \text{ and } \left( \alpha_{i} - \alpha'_{i} \right) y_{i}y_{j} + \alpha_{j} \geq 0
\end{gather}
$$
这个问题能较轻松地找到闭式解, 用闭式解更新参数即可.
```
````

```ad-note
title: 确定偏移项
注意到对任意支持向量 $\left( \boldsymbol{x}_{s}, y_{s} \right)$ 都有 $y_{s}f \left( \boldsymbol{x}_{s} \right) = 1$, 因此
$$
b = \frac{1}{y_{s}} - \sum_{i\in S}\alpha_{i}y_{i}\boldsymbol{x}_{i}^\mathrm{T}\boldsymbol{x}_{s}
$$
现实任务中采用一种更鲁棒的做法: 使用**所有支持向量求解的平均值**
$$
b = \frac{1}{|S|} \sum_{s\in S} \left( \frac{1}{y_{s}} - \sum_{i\in S}\alpha_{i}y_{i}\boldsymbol{x}_{i}^\mathrm{T}\boldsymbol{x}_{s} \right) 
$$
```

# 线性不可分情况
## 核支持向量机
```ad-question
title: 若不存在一个能正确划分两类样本的超平面, 怎么办?
将样本从原始空间映射到一个**更高维**的的特征空间, 使得样本在这个特征空间内线性可分.
```

```ad-note
title: 核支持向量机
设样本 $\boldsymbol{x} \mapsto \phi(\boldsymbol{x})$, 划分超平面为 $\boldsymbol{w}^\mathrm{T} \phi(\boldsymbol{x}) + b = 0$.
- 原问题 $$\min_{\boldsymbol{w}, b} \frac{1}{2}\|\boldsymbol{w}\|^{2} \quad \text{s.t.} \quad y_{i}(\boldsymbol{w}^\mathrm{T} \phi \left( \boldsymbol{x}_{i} \right)  + b) \geq 1,\ i = 1, 2, \cdots, m$$
- 对偶问题 $$\max_{\boldsymbol{\alpha}} g(\boldsymbol{\alpha}) = \sum_{i = 1}^{m} \alpha_{i} - \frac{1}{2} \sum_{i = 1}^{m} \sum_{j = 1}^{m} \alpha_{i}\alpha_{j}y_{i}y_{j}\phi \left( \boldsymbol{x}_{i} \right)^\mathrm{T}\phi \left( \boldsymbol{x}_{j} \right) \quad \text{s.t.} \quad \boldsymbol{\alpha} \succcurlyeq 0 \text{ and } \boldsymbol{\alpha}^{\mathrm{T}}\boldsymbol{y} = \sum_{i = 1}^{m} \alpha_{i} y_{i} = 0$$
- 预测函数 $$f(\boldsymbol{x}) = \boldsymbol{w}^{\mathrm{T}} \phi \left( \boldsymbol{x} \right) + b = \sum_{i = 1}^{m} \alpha_{i}y_{j} \phi \left( \boldsymbol{x}_{i} \right)^\mathrm{T} \phi \left( \boldsymbol{x} \right) + b$$
```

> 可以设计**核函数** $\kappa(\boldsymbol{x}, \boldsymbol{y}) = \phi(\boldsymbol{x})^{\mathrm{T}}\phi(\boldsymbol{y})$ 简化运算.

```ad-def
title: 核函数
令 $\mathcal{X}$ 为输入空间, $\mathcal{H}$ 为特征空间, 如果存在 $\phi(\cdot): \mathcal{X} \to \mathcal{H}$, 使得对所有 $\boldsymbol{x}, \boldsymbol{z}\in \mathcal{X}$, 函数 $\kappa(\cdot, \cdot)$ 满足条件
$$
\kappa \left( \boldsymbol{x}, \boldsymbol{z} \right) = \phi \left( \boldsymbol{x} \right)^\mathrm{T} \phi \left(  \boldsymbol{z}  \right)
$$
则称 $\kappa(\cdot, \cdot)$ 为核函数.
```

```ad-thm
title: Mercer 定理
令 $\mathcal{X}$ 为输入空间, $\kappa(\cdot, \cdot)$ 是定义在 $\mathcal{X}^2$ 上的对称函数, 则 $\kappa(\cdot,\cdot)$ 是核函数当且仅当对于任意数据集 $D = \left\{ \boldsymbol{x}_{1}, \cdots, \boldsymbol{x}_{m} \right\}$, 核矩阵 $\mathbf{K}$ 总是半正定的
$$
\mathbf{K} = 
\begin{pmatrix}
\kappa(\boldsymbol{x}_{1}, \boldsymbol{x}_{1}) & \cdots & \kappa(\boldsymbol{x}_{1}, \boldsymbol{x}_{m})  \\
\vdots & \ddots & \vdots \\
\kappa(\boldsymbol{x}_{m}, \boldsymbol{x}_{1}) & \cdots & \kappa(\boldsymbol{x}_{m}, \boldsymbol{x}_{m})  \\
\end{pmatrix}
$$
```

```ad-example
title: 常见核函数与核函数组合
| 名称       | 表达式                                                                                                                                      | 参数                      |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------- |
| 线性核     | $\kappa(\boldsymbol{x}_{i}, \boldsymbol{x}_{j}) = \boldsymbol{x}_{i}^\mathrm{T}\boldsymbol{x}_{j}$                                          |                           |
| 多项式核   | $\kappa(\boldsymbol{x}_{i}, \boldsymbol{x}_{j}) = \left( \boldsymbol{x}_{i}^\mathrm{T}\boldsymbol{x}_{j} \right)^d$                         | $d\geq 1$ 为多项式的次数  |
| 高斯核     | $\kappa(\boldsymbol{x}_{i}, \boldsymbol{x}_{j}) = \exp \left( - \dfrac{\|\boldsymbol{x}_{i} - \boldsymbol{x}_{j}\|^{2}}{2\delta^2} \right)$ | $\delta>0$ 为高斯核的带宽 |
| 拉普拉斯核 | $\kappa(\boldsymbol{x}_{i}, \boldsymbol{x}_{j}) = \exp \left( - \dfrac{\|\boldsymbol{x}_{i} - \boldsymbol{x}_{j}\|}{\delta} \right)$        | $\delta>0$                |
| Sigmoid 核 | $\kappa(\boldsymbol{x}_{i}, \boldsymbol{x}_{j}) = \tanh \left( \beta \boldsymbol{x}_{i}^{\mathrm{T}}\boldsymbol{x}_{j} + \theta \right)$    | $\beta > 0 , \theta < 0$                          |

核函数之间还可进行组合得到新的核函数
- 线性组合 $\gamma_{1}\kappa_{1} + \gamma_{2}\kappa_{2}$ 也是核函数.
- 直积 $\kappa_{1} \otimes \kappa_{2}(\boldsymbol{x}, \boldsymbol{z}) = \kappa_{1}(\boldsymbol{x}, \boldsymbol{z})\kappa_{2}(\boldsymbol{x}, \boldsymbol{z})$ 也是核函数.
- 对于任意函数 $g(\boldsymbol{x})$, $g(\boldsymbol{x})\kappa(\boldsymbol{x}, \boldsymbol{z})\kappa(\boldsymbol{z})$ 也是核函数.
```

## 软间隔支持向量机
```ad-caution
- 很难确定合适的核函数使得训练样本在特征空间中线性可分.
- 一个线性可分的结果也衡南断定是否是由过拟合造成的.
```

```ad-def
title: 软间隔
允许支持向量机在一些样本上不满足约束.
```

```ad-note
title: 引入松弛变量
为每个样本 $\left( \boldsymbol{x}_{i}, y_{i} \right)$ 引入松弛变量 $\xi_{i} \geq 0$.
- 需要引入正则化项: $C\sum_{i} \xi_{i}$.

原问题变为
$$
\min_{\boldsymbol{w}, b, \boldsymbol{\xi}} \frac{1}{2} {\|\boldsymbol{w}\|}^2 + C \sum_{i = 1}^m \xi_{i} \quad \text{s.t.} \quad 
\forall i = 1, 2, \cdots, m, 
\begin{cases}
y_{i} \left( \boldsymbol{w}^\mathrm{T} \phi \left( \boldsymbol{x}_{i} \right) + b \right) \geq 1 - \xi_{i} \\ 
\xi_{i} \geq 0
\end{cases}\tag{5}
$$
广义拉格朗日函数为
$$
\begin{gather}
L(\boldsymbol{w}, b, \boldsymbol{\xi}, \boldsymbol{\alpha}, \boldsymbol{\beta}) = \frac{1}{2} \| \boldsymbol{w} \|^2 + C \sum_{i = 1}^m \xi_{i} - \sum_{i = 1}^m \alpha_{i} \left( y_{i} \left( \boldsymbol{w}^\mathrm{T} \phi \left( \boldsymbol{x}_{i} \right) + b \right) - 1 + \xi_{i} \right) - \sum_{i = 1}^m \mu_{i}\xi_{i} \\
\boldsymbol{\alpha}, \boldsymbol{\mu} \succcurlyeq 0
\end{gather}
$$
对偶问题为
$$
\max_{\boldsymbol{\alpha}} g(\boldsymbol{\alpha}) = \sum_{i = 1}^m \alpha_{i} - \frac{1}{2} \sum_{i = 1}^m \sum_{j = 1}^m \alpha_{i}\alpha_{j}y_{i}y_{j}\phi(\boldsymbol{x}_{i})\phi(\boldsymbol{x}_{j}) \quad \text{s.t.} \quad 
C \succcurlyeq \boldsymbol{\alpha} \succcurlyeq 0 \text{ and } \sum_{i = 1}^m\alpha_{i} y_{i} = 0
$$
终止条件: KKT 条件
$$
\begin{cases}
\alpha_{i} \geq 0, \mu_{i} \geq 0 \\
y_{i} \left( \boldsymbol{w}^\mathrm{T} \phi \left( \boldsymbol{x}_{i} \right) + b \right) \geq 1 - \xi_{i} \\
\alpha_{i} \left( y_{i} \left( \boldsymbol{w}^\mathrm{T} \phi \left( \boldsymbol{x}_{i} \right) + b \right) - 1 + \xi_{i} \right) = 0 \\
\xi_{i} \geq 0, \mu_{i}\xi_{i} = 0
\end{cases}
$$
```

````ad-note
title: 目标函数视角
基本思想: 最大化间隔的同时, 让不满足约束的样本尽可能少.
$$
\min_{\boldsymbol{w}, b} \frac{1}{2} \| \boldsymbol{w} \|^2 + C \sum_{i = 1}^m \ell_{0 / 1} \left( y_{i} \left( \boldsymbol{w}^\mathrm{T} \phi \left( \boldsymbol{x}_{i} \right) + b \right) - 1 \right) 
$$
其中 $\ell_{0 / 1}$ 是 0/1 损失函数
$$
\ell_{0 / 1}(z) = 
\begin{cases}
1 & z < 0 \\
0 & \text{otherwise}
\end{cases}
$$
```ad-caution
0/1 损失函数非凸, 非连续, 不易优化.
```
---
将损失函数修改为 $\ell(z) = \max(0, -z)$, 则目标函数为
$$
\min_{\boldsymbol{w}, b} \frac{1}{2} \| \boldsymbol{w} \|^2 + C \sum_{i = 1}^m \max \left( 0, 1 - y_{i} \left( \boldsymbol{w}^\mathrm{T} \phi \left( \boldsymbol{x}_{i} \right) + b \right) \right) 
$$
可以证明该式与式 (5) 等价.

> 这个目标函数的性质很好, 能够较方便地进行学习

````

```ad-note
title: 替代损失函数
令 $z_{i} = y_{i} \left( \boldsymbol{w}^\mathrm{T} \phi \left( \boldsymbol{x}_{i} \right) + b \right)$ 为函数间隔. 可以取不同的损失函数:

![[Pasted image 20221026223432.png|300]]

- $\ell_{\text{exp}}(z) = \exp(-z)$
- $\ell_{\text{log}}(z) = \log(1 + \exp(-z))$: 对率回归
- $\ell_{\text{hinge}}(z) = \max(0, 1 - z)$: 软间隔 SVM

> 替代损失函数数学性质较好, 一般是 0/1 损失函数的上界.
```

# 正则化
```ad-note
title: 支持向量机学习模型的更一般形式
$$
\min_{f} \underbrace{ \Omega(f) }_{ \text{结构风险} } + \underbrace{ C \sum_{i}^m \ell \left( f(x_{i}), y_{i} \right)  }_{ \text{经验风险} }
$$
- 结构风险描述模型的某些性质
- 经验风险描述模型与训练数据的契合程度
- 通过替换这两部分, 可以得到许多其他学习模型
	- 对数几率回归 Logistic Regression
	- 最小绝对收缩选择算子 LASSO
	- ...
```

# 支持向量回归
允许模型输出和实际输出间存在 $2\epsilon$ 的偏差. 

```ad-note
title: 损失函数
落入中间 $2\epsilon$ 间隔带的样本不计算损失, 从而使得模型获得稀疏性.
- 最小二乘损失函数 $\ell(z) = z^2$
- **支持向量回归损失函数** $\ell_{\epsilon}(z) = \max(0, |z| - \epsilon)$

得到目标
$$
f \left( \boldsymbol{x}\right) = \boldsymbol{w}^\mathrm{T} \boldsymbol{x} + b \qquad
\min_{\boldsymbol{w}, b} \frac{1}{2} \| \boldsymbol{w} \|^2 + C \sum_{i = 1}^m \ell_{\epsilon}\left( f \left( \boldsymbol{x}_{i} \right) - y_{i} \right) 
$$
原问题
$$
\min_{\boldsymbol{w}, b, \boldsymbol{\xi}} \frac{1}{2}\| \boldsymbol{w} \|^2 + C \sum_{i = 1}^m \left( \xi_{i} + \hat{\xi}_{i} \right) \quad \text{s.t.} \quad
\begin{cases}
- ( \epsilon + \hat{\xi}_{i} ) \leq f \left( \boldsymbol{x}_{i} \right) - y_{i} \leq \epsilon + \xi_{i} \\
\xi_{i} \geq 0, \hat{\xi}_{i} \geq 0
\end{cases}
$$
对偶问题
$$
\begin{gathered}
    \max_{\boldsymbol{\alpha}, \hat{\boldsymbol{\alpha}}}
    g \left( \boldsymbol{\alpha}, \hat{\boldsymbol{\alpha}}\right)
    = - \frac{ 1 }{ 2 } \sum_{i = 1}^{m} \sum_{j = 1}^{m}
    \left( \alpha_{i} - \hat{\alpha}_{i}\right)
    \left( \alpha_{j} - \hat{\alpha}_{j}\right)
    \kappa \left( \boldsymbol{x}_{i}, \boldsymbol{x}_{j}\right)
    + \sum_{i = 1}^{m} \left(
    y_{i}\left( \hat{\alpha}_{i} - \alpha_{i}\right) - \epsilon
    \left( \hat{\alpha}_{i} + \alpha_{i}\right)
    \right) \\
    \text{s.t. } C \succcurlyeq \boldsymbol{\alpha},
    \hat{\boldsymbol{\alpha}} \succcurlyeq 0 \text{ and }
    \sum_{i = 1}^{m} \left( \alpha_{i} - \hat{\alpha}_{i}\right) = 0
\end{gathered}
$$
预测函数
$$
f(\boldsymbol{x}) = \boldsymbol{w}^{\mathrm{T}} \phi \left( \boldsymbol{x} \right) + b = \sum_{i = 1}^{m} \left( \hat{\alpha_{i}} - \alpha_{i} \right) y_{j} \kappa\left( \boldsymbol{x}_{i} , \boldsymbol{x} \right) + b
$$
```

# 核方法

```ad-thm
title: 表示定理
```