#计算机 #软件 #机器学习 

# 基本形式
- 给定 $d$ 个属性描述的示例 $\boldsymbol{x} = (x_1; x_2; \cdots; x_d)$, 其中 $x_i$ 是 $\boldsymbol{x}$ 在第 $i$ 个属性上的取值
- 线性模型
	- $f(\boldsymbol{x})=w_1x_1+w_2x_2+\cdots+w_dx_d+b$
	- 向量形式: $f(\boldsymbol{x})=\boldsymbol{w}^\mathrm{T}\boldsymbol{x}+b$, $\boldsymbol{w} = (w_1; w_2; \cdots; w_d)$.
	- $\boldsymbol{w}$ 和 $b$ 学得之后模型就得以确定
- 非线性模型可在线性模型的基础上得到
	- 引入层次结构
	- 高维映射

# 线性回归
```ad-def
title: 线性回归
给定数据集 $D = \{(\boldsymbol{x}_1, y_1), (\boldsymbol{x}_2, y_2), \cdots, (\boldsymbol{x}_m, y_m)\}$, 其中 $\boldsymbol{x}_i = (x_{i1}; x_{i2}; \cdots; x_{id}), y_i\in\mathbb{R}$. **线性回归** linear regression 试图学得一个线性模型以尽可能准确地预测实值输出标记.
```


下面先考虑一个最简单的模型, 即输入属性的数目只有一个. 对于离散属性, 如果属性值之间存在序关系, 可通过连续化将其转化为连续值.
```ad-note
title: 一元线性回归
我们忽略关于属性的下标, 即 $D = \{(x_i, y_i)\}^m_{i=1}, x_i\in \mathbb{R}$.

线性回归试图学得 
$$
	f(x_i) = w x_i + b\quad\text{s.t.}\quad f(x_i)\simeq y_i
$$
[[Chap 2. 模型评估与选择#^b0298d|均方误差]]是回归任务中最常用的性能度量, 因此我们可试图让均方误差最小化, 即
$$
(w^*, b^*)=\underset{(w, b)}{\mathrm{argmin}}\sum_{i = 1}^m (f(x_i) - y_i)^2 = \underset{(w, b)}{\mathrm{argmin}}\sum_{i = 1}^m (y_i - w x_i - b)^2
$$
我们使用**最小二乘法**, 记 $E_{(w, b)}=\sum_{i = 1}^m (y_i - w x_i - b)^2$, 对其求导得
$$
\begin{gather}
\frac{\partial E_{(w, b)}}{\partial w} = 2\left(w\sum_{i = 1}^m x_i^2 - \sum_{i = 1}^m(y_i - b)x_i \right)\\
\frac{\partial E_{(w, b)}}{\partial w} = 2\left(mb - \sum_{i = 1}^m(y_i - w x_i) \right)
\end{gather}
$$
令上面的式子为零即可得到 $w$ 和 $b$ 最优解的**闭式** closed-form 解
$$
\begin{gather}
w = \frac{\displaystyle \sum_{i = 1}^m y_i(x_i - \bar x)}{\displaystyle \sum_{i = 1}^m x_i^2 - \frac{1}{m}\left(\sum_{i = 1}^m x_i\right)^2}, \quad \bar x=\frac{1}{m}\sum_{i = 1}^m x_i \\
b = \frac{1}{m}\sum_{i = 1}^m (y_i - w x_i)
\end{gather}
$$
```
更一般的情况是样本由 $d$ 个属性描述.
```ad-note
title: 多元线性回归
线性回归试图学得 
$$
	f(\boldsymbol{x}_i) = \boldsymbol{w}^\mathrm{T} \boldsymbol{x}_i + b\quad\text{s.t.}\quad f(\boldsymbol{x}_i)\simeq y_i
$$
为便于讨论, 我们把 $\boldsymbol{w}$ 和 $b$ 吸收入向量形式
$$
\hat{\boldsymbol{w}} = (\boldsymbol{w}; b)
$$
把数据集 $D$ 表示为一个 $m\times (d + 1)$ 大小的矩阵 $\mathbf{X}$
$$
\mathbf{X} = \begin{pmatrix} 
x_{11} & x_{12} & \cdots & x_{1d} & 1 \\ 
x_{21} & x_{22} & \cdots & x_{2d} & 1 \\ 
\vdots & \vdots & \ddots & \vdots & \vdots \\
x_{m1} & x_{m2} & \cdots & x_{md} & 1 \\ 
\end{pmatrix}
= \begin{pmatrix}
\boldsymbol{x}_1^\mathrm{T} & 1 \\ 
\boldsymbol{x}_2^\mathrm{T} & 1 \\ 
\vdots & \vdots \\
\boldsymbol{x}_m^\mathrm{T} & 1 \\ 
\end{pmatrix}
$$
把标记也写成向量形式
$$
\boldsymbol{y} = (y_1; y_2; \cdots; y_m)
$$
那么就有
$$
\hat{\boldsymbol{w}}^* = \underset{\hat{\boldsymbol{w}}}{\mathrm{argmin}} (\boldsymbol{y} - \mathbf{X}\hat{\boldsymbol{w}})^\mathrm{T} (\boldsymbol{y} - \mathbf{X}\hat{\boldsymbol{w}})
$$
令 $E_{\hat{\boldsymbol{w}}} = (\boldsymbol{y} - \mathbf{X}\hat{\boldsymbol{w}})^\mathrm{T} (\boldsymbol{y} - \mathbf{X}\hat{\boldsymbol{w}})$, 对 $\hat{\boldsymbol{w}}$ 求导得到
$$
 \frac{\partial E_{\hat{\boldsymbol{w}}}}{\partial \hat{\boldsymbol{w}}} = 2\mathbf{X}^\mathrm{T}(\mathbf{X}\hat{\boldsymbol{w}} - \boldsymbol{y})
$$
令上式为零即可得到 $\hat{\boldsymbol{w}}$ 最优解的闭式解. 由于涉及矩阵逆运算, 比单变量情况复杂, 下面进行一个简单的讨论.
- 当 $\mathbf{X}^\mathrm{T}\mathbf{X}$ 为满秩矩阵或正定矩阵时, 可得 $$\hat{\boldsymbol{w}}^* = (\mathbf{X}^\mathrm{T}\mathbf{X})^{-1}\mathbf{X}^\mathrm{T}\boldsymbol{y}$$ 令 $\hat{\boldsymbol{x}}_i = (\boldsymbol{x}_i;1)$, 则最终学得的多元线性回归模型为 $$f(\hat{\boldsymbol{x}}_i) = \hat{\boldsymbol{x}}_i^\mathrm{T}(\mathbf{X}^\mathrm{T}\mathbf{X})^{-1}\mathbf{X}^\mathrm{T}\boldsymbol{y}$$
- 当 $\mathbf{X}^\mathrm{T}\mathbf{X}$ 不是满秩矩阵或正定矩阵时 (例如变量数超过样例数), 此时可解出多个 $\hat{\boldsymbol{w}}^*$, 由归纳偏好决定选择哪一个解作为输出, 常见的做法是引入*正则化 regularization* 项
```
为便于观察, 我们将线性回归模型简写为
$$
y = \boldsymbol{w}^\mathrm{T}\boldsymbol{x} + b
$$
```ad-def
title: 指数线性回归
即
$$
\ln y = \boldsymbol{w}^\mathrm{T}\boldsymbol{x} + b
$$
它实际上是在试图让 $\mathrm{e}^{\boldsymbol{w}^\mathrm{T}\boldsymbol{x} + b}$ 逼近 $y$.
它在形式上仍是线性回归, 但实际上已是在求取输入空间到输出空间的非线性函数映射.
```
```ad-def
title: 广义线性模型
更一般地, 考虑单调可微函数 $g(\cdot)$, 令
$$
y = g^{-1} (\boldsymbol{w}^\mathrm{T}\boldsymbol{x} + b)
$$
这样得到的模型称为**广义线性模型** generalized linear model, 函数 $g(\cdot)$, 称为**联系函数** link function.
```

^fe593c

# 对数几率回归
> 将回归模型用于分类任务

考虑二分类任务, 其输出标记为 $y\in \{0, 1\}$, 而线性回归模型产生的预测值 $z = \boldsymbol{w}^\mathrm{T}\boldsymbol{x}+b$ 是实值. 于是我们需要将实值 $z$ 转化为 0/1 值, 最理想的是**单位阶跃函数** unit-step function, 即若预测值 $z$ 大于零就判为正例, 小于零就判为反例, 预测值为临界值零则可任意判别.
```ad-def
title: 单位阶跃函数
$$
y = \begin{cases}
0, & z < 0; \\
0.5, & z = 0; \\
1, &z > 0,
\end{cases}
$$
> 单位阶跃函数不连续, 不可直接用作[[#^fe593c|广义线性模型]]中的 $g^{-1}(\cdot)$
```

^baef7f

**对数几率函数** logistic function 是一个近似单位阶跃函数的**替代函数** surrogate function, 而且单调可微.

```ad-def
title: 对数几率函数

$$
y = \frac{1}{1 + \mathrm e^{-z}} = \frac{1}{1 + \mathrm e^{-(\boldsymbol{w}^\mathrm{T}\boldsymbol{x}+b)}}\tag{1}
$$
它是一种 **Sigmoid 函数**, 将 $z$ 值转化为一个接近 0 或 1 的 $y$ 值, 并且输出值在 $z = 0$ 附近变化很陡.

![[Sigmoid.svg]]
```

^2200c4

```ad-def
title: 几率与对数几率
将对数几率函数变化为
$$
\ln \frac{y}{1-y}=\boldsymbol{w}^\mathrm{T}\boldsymbol{x}+b \tag{2}
$$
若将 $y$ 视为样本 $\boldsymbol{x}$ 为正例的可能性, 则 $1-y$ 是其反例的可能性, 两者的比值 $\displaystyle \frac{y}{1-y}$ 称为**几率** odds, 反映了 $\boldsymbol{x}$ 作为正例的相对可能性. 几率取对数则得到**对数几率** log odds/logit 为 $\displaystyle\ln \frac{y}{1-y}$
```

```ad-note
title: 对数几率回归
式 (1) 实际上是用线性回归模型的预测结果去逼近真实标记的对数几率, 因此其对应的模型被称为**对数几率回归** logistic/logit regression.
- 虽然它的名字是回归, 但实际上是一种分类学习方法
- 优点
	- 直接对分类可能性进行建模, 无需事先假设数据分布
		- 避免了假设分布不准确所带来的问题
	- 不是仅预测出 "类别", 而是可得到近似概率预测
		- 对许多需利用概率辅助决策的任务很有用
	- 求解的目标函数是任意阶可导的凸函数, 有很好的数学性质
		- 现有的许多数值优化算法都可直接用于求解最优解
```

````ad-note
title: 确定 $\boldsymbol{w}$ 和 $b$
将式 (1) 中的 $y$ 视为类后验概率估计 $p(y=1\vert \boldsymbol{x})$, 则式 (2) 可重写为
$$
\ln \frac{p(y=1\vert \boldsymbol{x})}{p(y=0\vert \boldsymbol{x})} = \boldsymbol{w}^\mathrm{T}\boldsymbol{x} + b
$$
显然有
$$
\begin{align}
p(y=1\vert \boldsymbol{x}) &= \frac{\mathrm e^{\boldsymbol{w}^\mathrm{T}\boldsymbol{x}+b}}{1 + \mathrm e^{\boldsymbol{w}^\mathrm{T}\boldsymbol{x}+b}}\tag{3} \\
p(y=1\vert \boldsymbol{x}) &= \frac{1}{1 + \mathrm e^{\boldsymbol{w}^\mathrm{T}\boldsymbol{x}+b}}\tag{4}
\end{align}
$$
可以通过**极大似然法** maximum likelihood method 来估计 $\boldsymbol{w}$ 和 $b$.

给定数据集 $\{(\boldsymbol{x}_i, y_i)\}^m_{i=1}$, 对数回归模型最大化**对数似然** log-likelihood
$$
\ell (\boldsymbol{w}, b) = \sum_{i = 1}^m \ln p(y_i \vert \boldsymbol{x}_i; \boldsymbol{w}, b) \tag{5}
$$
即令每个样本属于其真实标记的概率越大越好.

令 $\boldsymbol{\beta}=(\boldsymbol{w};b)$, $\hat{\boldsymbol{x}}=(\boldsymbol{x};1)$, 则 $\boldsymbol{w}^{\mathrm{T}}\boldsymbol{x}+b$ 可简写为 $\boldsymbol{\beta}^{\mathrm{T}}\hat{\boldsymbol{x}}$. 再令 $p_1(\hat{\boldsymbol{x}};\boldsymbol{\beta}) = p(y=1\vert \hat{\boldsymbol{x}}; \boldsymbol{\beta})$, $p_0(\hat{\boldsymbol{x}};\boldsymbol{\beta}) = p(y=0\vert \hat{\boldsymbol{x}}; \boldsymbol{\beta}) = 1-p_1(\hat{\boldsymbol{x}};\boldsymbol{\beta})$, 则有
$$
p(y_i \vert \boldsymbol{x}_i; \boldsymbol{w}, b) = y_i p_1(\hat{\boldsymbol{x}};\boldsymbol{\beta}) + (1-y_i)p_0(\hat{\boldsymbol{x}};\boldsymbol{\beta})
$$
代入式 (5), 并根据式 (3) 和 (4) 可知, 最大化式 (5) 等价于最小化
$$
\ell (\boldsymbol{\beta}) = \sum_{i=1}^m\left(-y_i\boldsymbol{\beta}^{\mathrm{T}}\hat{\boldsymbol{x}}_i+\ln \left(1+\mathrm{e}^{\boldsymbol{\beta}^{\mathrm{T}}\hat{\boldsymbol{x}}_i}\right)\right)\tag{6}
$$
式 (6) 是关于 $\boldsymbol{\beta}$ 的高阶可导连续凸函数, 根据凸优化理论, 经典的数值优化算法如**梯度下降法** gradient descent method, 牛顿法 Newton method 都可求得其最优解, 于是就得到
$$
\boldsymbol{\beta}^*=\underset{\boldsymbol{\beta}}{\mathrm{argmin}}\ \ell(\boldsymbol{\beta})
$$

```ad-example
title: 牛顿法
以牛顿法为例, 从当前 $\boldsymbol{\beta}$ 生成下一轮迭代解的更新公式为
$$
\boldsymbol{\beta}' = \boldsymbol{\beta} - \left(\frac{\partial^2\ell(\boldsymbol{\beta})}{\partial\boldsymbol{\beta}\partial\boldsymbol{\beta}^{\mathrm{T}}} \right)^{-1} \frac{\partial\ell(\boldsymbol{\beta})}{\partial\boldsymbol{\beta}}
$$
其中关于 $\boldsymbol{\beta}$ 的一阶, 二阶导数分别为
$$
\begin{align*}
\frac{\partial\ell(\boldsymbol{\beta})}{\partial\boldsymbol{\beta}} &= -\sum_{i = 1}^m\hat{\boldsymbol{x}}_i(y_i-p_1(\hat{\boldsymbol{x}}_i; \boldsymbol{\beta})) \\
\frac{\partial^2\ell(\boldsymbol{\beta})}{\partial\boldsymbol{\beta}\partial\boldsymbol{\beta}^{\mathrm{T}}} &= \sum_{i = 1}^m\hat{\boldsymbol{x}}_i\hat{\boldsymbol{x}}_i^{\mathrm{T}}p_1(\hat{\boldsymbol{x}}_i; \boldsymbol{\beta})(1-p_1(\hat{\boldsymbol{x}}_i; \boldsymbol{\beta}))
\end{align*}
$$
```
````

# 线性判别分析
```ad-note
title: 线性判别分析 Linear Discriminant Analysis
![[LDA.svg|inlineR]]
- 针对二分类问题的线性学习方法
- 最早由 \[Fisher, 1936\] 提出, 亦称 Fisher 判别分析
- LDA 的思想非常朴素
	- 给定训练样例集, 设法将样例投影到一条直线上
	- 使得同类样例的投影点尽可能接近
	- 异类样例的投影点尽可能远离
	- 在对新样例进行分类时, 将其投影到同样的这条直线上, 再根据投影点的位置来确定新样本的类别
```

```ad-note
title: 数学模型
- 给定数据集 $D=\{(\boldsymbol{x}_i, y_i)\}_{i=1}^m, y_i\in\{0, 1\}$.
- 令 $X_i$, $\boldsymbol{\mu}_i$, $\boldsymbol{\Sigma}_i$ 分别表示第 $i\in\{0, 1\}$ 类示例的集合, 均值向量, 协方差矩阵.
- 将数据投影到直线 $\boldsymbol{w}$ 上. ($\boldsymbol{w}$ 即为直线的方向向量)
	- 两样本的中心在直线上的投影分别为 $\boldsymbol{w}^{\mathrm{T}}\boldsymbol{\mu}_0$ 和 $\boldsymbol{w}^{\mathrm{T}}\boldsymbol{\mu}_1$.
	- 若所有样本都投影到直线上, 则两类样本的协方差分别为 $\boldsymbol{w}^{\mathrm{T}}\boldsymbol{\Sigma}_0\boldsymbol{w}$ 和 $\boldsymbol{w}^{\mathrm{T}}\boldsymbol{\Sigma}_1\boldsymbol{w}$.

欲同类样例的投影点尽可能接近, 可以让同类样例投影点的协方差尽可能小, 即 $\boldsymbol{w}^{\mathrm{T}}\boldsymbol{\Sigma}_0\boldsymbol{w} + \boldsymbol{w}^{\mathrm{T}}\boldsymbol{\Sigma}_1\boldsymbol{w}$ 尽可能小. 欲异类样例的投影点尽可能远离, 可以让类中心之间的距离尽可能大, 即 $\lVert \boldsymbol{w}^{\mathrm{T}}\boldsymbol{\mu}_0 - \boldsymbol{w}^{\mathrm{T}}\boldsymbol{\mu}_1 \rVert_2^2$ 尽可能大. 即可得到欲最大化的目标
$$
\begin{align*}
J &= \frac{\lVert \boldsymbol{w}^{\mathrm{T}}\boldsymbol{\mu}_0 - \boldsymbol{w}^{\mathrm{T}}\boldsymbol{\mu}_1 \rVert_2^2}{\boldsymbol{w}^{\mathrm{T}}\boldsymbol{\Sigma}_0\boldsymbol{w} + \boldsymbol{w}^{\mathrm{T}}\boldsymbol{\Sigma}_1\boldsymbol{w}}\\
&= \frac{\boldsymbol{w}^{\mathrm{T}}(\boldsymbol{\mu}_0-\boldsymbol{\mu}_1)(\boldsymbol{\mu}_0-\boldsymbol{\mu}_1)^{\mathrm{T}}\boldsymbol{w}}{\boldsymbol{w}^{\mathrm{T}}(\boldsymbol{\Sigma}_0 + \boldsymbol{\Sigma}_1)\boldsymbol{w}}\tag{7}
\end{align*}
$$
定义
- **类内散度矩阵** within-class scatter matrix$$\begin{align*}\mathbf{S}_w &= \boldsymbol{\Sigma}_0 + \boldsymbol{\Sigma}_1\\&= \sum_{\boldsymbol{x}\in X_0}(\boldsymbol{x}-\boldsymbol{\mu}_0)(\boldsymbol{x}-\boldsymbol{\mu}_0)^{\mathrm{T}} + \sum_{\boldsymbol{x}\in X_1}(\boldsymbol{x}-\boldsymbol{\mu}_1)(\boldsymbol{x}-\boldsymbol{\mu}_1)^{\mathrm{T}}\end{align*}$$
- **类间散度矩阵** between-class scatter matrix
$$
\mathbf{S}_b = (\boldsymbol{\mu}_0-\boldsymbol{\mu}_1)(\boldsymbol{\mu}_0-\boldsymbol{\mu}_1)^{\mathrm{T}}
$$

式 (7) 可重写为
$$
J = \frac{\boldsymbol{w}^{\mathrm{T}}\mathbf{S}_b\boldsymbol{w}}{\boldsymbol{w}^{\mathrm{T}}\mathbf{S}_w\boldsymbol{w}}\tag{8}
$$
这就是 LDA 欲最大化的目标, 即 $\mathbf{S}_b$ 和 $\mathbf{S}_w$ 的**广义瑞利商** generalized Rayleigh quotient.
```

````ad-note
title: 确定 $\boldsymbol{w}$
不难发现式 (8) 的值与 $\boldsymbol{w}$ 的长度无关, 只与其方向有关. 不失一般性, 令 $\boldsymbol{w}^{\mathrm{T}}\mathbf{S}_w\boldsymbol{w} = 1$, 则式 (8) 等价于
$$
\begin{align*}

\min_{\boldsymbol{w}}&-\boldsymbol{w}^{\mathrm{T}}\mathbf{S}_b\boldsymbol{w}\\
\text{s.t.} \ &\boldsymbol{w}^{\mathrm{T}}\mathbf{S}_w\boldsymbol{w} = 1
\end{align*}
$$
由拉格朗日乘子法, 上式等价于
$$
\frac{\partial}{\partial\boldsymbol{w}}\left[-\boldsymbol{w}^{\mathrm{T}}\mathbf{S}_b\boldsymbol{w}+\lambda\left(\boldsymbol{w}^{\mathrm{T}}\mathbf{S}_w\boldsymbol{w} - 1\right) \right] = 0\Rightarrow
\\
\mathbf{S}_b\boldsymbol{w}=\lambda\mathbf{S}_w\boldsymbol{w}\tag{9}
$$
其中 $\lambda$ 是拉格朗日乘子. 因为 $(\boldsymbol{\mu}_0-\boldsymbol{\mu}_1)^{\mathrm{T}}\boldsymbol{w}$ 是标量, 所以 $\mathbf{S}_b\boldsymbol{w}$ 的方向恒为 $\boldsymbol{\mu}_0-\boldsymbol{\mu}_1$, 不妨令
$$
\mathbf{S}_b\boldsymbol{w}=\lambda(\boldsymbol{\mu}_0-\boldsymbol{\mu}_1)
$$
代入式 (9) 即得
$$
\boldsymbol{w}=\mathbf{S}_w^{-1}(\boldsymbol{\mu}_0-\boldsymbol{\mu}_1)
$$

考虑到数值解的稳定性, 在实践中通常是对 $\mathbf{S}_w$ 进行奇异值分解, 即 $\mathbf{S}_w = \mathbf{U}\mathbf{\Sigma}\mathbf{V}^\mathrm{T}$, 然后再由 $\mathbf{S}_w^{-1} = \mathbf{V}\mathbf{\Sigma}^{-1}\mathbf{U}^\mathrm{T}$ 得到 $\mathbf{S}_w^{-1}$.

```ad-def
title: 奇异值分解 singular value decomposition
- 奇异值, 奇异向量
	- 一个非负实数 $\sigma$ 是 $\mathbf{S}\in\mathbb{R}^{m\times n}$ 的一个奇异值仅当存在 $\boldsymbol{u}\in\mathbb{R}^{m\times 1}$ 和 $\boldsymbol{v}\in\mathbb{R}^{n\times 1}$ 使得 $$\mathbf{S}\boldsymbol{v}=\sigma\boldsymbol{u}\quad\mathbf{S}^{\mathrm{T}}\boldsymbol{u}=\sigma\boldsymbol{v}$$ 其中 $\boldsymbol{u}$ 和 $\boldsymbol{v}$ 分别为 $\sigma$ 的左奇异向量和右奇异向量
	- 对于 $\mathbf{S}\in\mathbb{R}^{m\times n}$, $\mathbf{S}\mathbf{S}^{\mathrm{T}}\in\mathbb{R}^{m\times m}$ 和 $\mathbf{S}^{\mathrm{T}}\mathbf{S}\in\mathbb{R}^{n\times n}$ 的非零特征值相同, 记为 $\{\lambda_i\}_{i=1}^{r}$, $r\le\mathrm{rank}(\mathbf{S})\le\min (m,n)$, 则 $\mathbf{S}$ 的奇异值均为 $\lambda_i$ 的算术平方根, 即 $\{\sigma_i = \sqrt{\lambda_i}\}_{i=1}^{r}$, 其中 $\mathbf{S}\mathbf{S}^{\mathrm{T}}$ 和 $\mathbf{S}^{\mathrm{T}}\mathbf{S}$ 对应 $\lambda_i$ 的特征向量 $\boldsymbol{u}_i$ 和 $\boldsymbol{v}_i$ 分别为 $\sigma_i$ 的左奇异向量和右奇异向量
- 奇异值分解
	- 设 $\mathbf{S}\in\mathbb{R}^{m\times n}$, 则存在一个分解使得 $$\mathbf{S} = \mathbf{U}\mathbf{\Sigma}\mathbf{V}^\mathrm{T}$$
		- $\mathbf{U}\in\mathbb{R}^{m\times m}$ 为酉矩阵, $\mathbf{\Sigma}\in\mathbb{R}^{m\times n}$ 为非负实数对角矩阵, $\mathbf{V}\in\mathbb{R}^{n\times n}$ 为酉矩阵
		- 这样的分解就称作 $\mathbf{S}$ 的**奇异值分解**, $\mathbf{\Sigma}$ 对角线上的元素 $\mathbf{\Sigma}_{i,i}$ 即为 $\mathbf{S}$ 的**奇异值**.
	- 给定矩阵 $\mathbf{S}$
		- $\mathbf{U}$ 的列向量 (左奇异向量) 是 $\mathbf{S}\mathbf{S}^{\mathrm{T}}$ 的特征向量.
		- $\mathbf{\Sigma}$ 的非零对角元 (非零奇异值) 是 $\mathbf{S}\mathbf{S}^{\mathrm{T}}$ 或 $\mathbf{S}^{\mathrm{T}}\mathbf{S}$ 的非零特征值的平方根.
		- $\mathbf{V}$ 的列向量 (右奇异向量) 是 $\mathbf{S}^{\mathrm{T}}\mathbf{S}$ 的特征向量.
```
````

```ad-info
LDA 可从贝叶斯决策理论的角度来阐释, 并可证明, 当两类数据同先验, 满足高斯分布且协方差相等时, LDA 可达到最优分类.
```

```ad-note
title: LDA 推广到多分类学习
- 假设存在 $N$ 个类.
- 第 $i$ 类示例数为 $m_i$.
- $\boldsymbol{\mu}$ 是所有示例的均值向量.

定义
- **全局散度矩阵**$$\begin{align*}\mathbf{S}_t &= \mathbf{S}_b + \mathbf{S}_w\\&= \sum_{i = 1}^m(\boldsymbol{x}_i-\boldsymbol{\mu})(\boldsymbol{x}_i-\boldsymbol{\mu})^{\mathrm{T}}\end{align*}$$
- 类内散度矩阵$$\mathbf{S}_w = \sum_{i = 1}^N \mathbf{S}_{w_i}=\sum_{i = 1}^N\sum_{\boldsymbol{x}\in X_i}(\boldsymbol{x}-\boldsymbol{\mu}_i)(\boldsymbol{x}-\boldsymbol{\mu}_i)^{\mathrm{T}}$$
- 类间散度矩阵$$\begin{align*}\mathbf{S}_b &= \mathbf{S}_t - \mathbf{S}_w\\&= \sum_{i = 1}^N m_i(\boldsymbol{\mu}_i-\boldsymbol{\mu})(\boldsymbol{\mu}_i-\boldsymbol{\mu})^{\mathrm{T}}\end{align*}$$

多分类 LDA 可以有多种实现方法: 使用 $\mathbf{S}_b$, $\mathbf{S}_w$, $\mathbf{S}_t$ 三者中的任何两个即可.

常见的一种实现方法是采用优化目标
$$
\max_{\mathbf{W}}\frac{\mathrm{tr}\left(\mathbf{W}^{\mathrm{T}}\mathbf{S}_b\mathbf{W}\right)}{\mathrm{tr}\left(\mathbf{W}^{\mathrm{T}}\mathbf{S}_w\mathbf{W}\right)}\tag{10}
$$
其中 $\mathbf{W}\in \mathbb{R}^{d\times (N-1)}$, 式 (10) 可通过如下广义特征值问题求解
$$
\mathbf{S}_b\mathbf{W}=\lambda\mathbf{S}_w\mathbf{W}
$$
$\mathbf{W}$ 的闭式解则是 $\mathbf{S}_w^{-1}\mathbf{S}_b$ 的 $d'$ 个最大非零广义特征值所对应的特征向量组成的矩阵, $d'\le N-1$.

若将 $\mathbf{W}$ 视为一个投影矩阵, 则多分类 LDA 将样本投影到 $d'$ 维空间, $d'$ 通常远小于数据原有属性数 $d$. 于是可通过这个投影来减小样本点的维数, 且投影过程中使用了类别信息, 因此 LDA 也常被视为一种经典的监督降维技术.
```

# 多分类学习
> 基于一些基本策略, 利用二分类学习器来解决多分类问题.

考虑 $N$ 个类别 $C_1, C_2, \cdots, C_N$,  多分类学习的基本思路是**拆解法**, 即将多分类任务拆为若干个二分类任务求解.
- 先对问题进行拆分, 然后为拆出的每个二分类任务训练一个分类器;
- 在测试时, 对这些分类器的预测结果进行集成以获得最终的多分类结果.
- 关键是如何对多分类任务进行拆分, 以及如何对多个分类器进行集成.

本节主要介绍拆分策略, 最经典的拆分策略有三种
- **一对一** One vs. One, 简称 OvO
- **一对其余** One vs. Rest, 简称 OvR
- **多对多** Many vs. Many, 简称 MvM

给定数据集 $D = \{(\boldsymbol{x}_1, y_1), (\boldsymbol{x}_2, y_2), \cdots, (\boldsymbol{x}_m, y_m)\}, y_i\in\{C_1, C_2, \cdots, C_N\}$.

```ad-def
title: 一对一 OvO
OvO 将 $N$ 个类别两两配对, 从而产生 $N(N-1)/2$ 个二分类任务. 在测试阶段, 新样本将同时提交给所有分类器, 于是我们将得到 $N(N-1)/2$ 个分类结果, 最终结果可通过投票产生, 即吧被预测得最多的类别作为最终分类结果.
```

```ad-def
title: 一对其余 OvR
OvR 每次将一个类的样例作为正例, 所有其他类的样例作为反例来训练 $N$ 个分类器. 在测试时若仅有一个分类器预测为正类, 则对应的类别标记作为最终分类结果. 若有多个分类器预测为正类, 则通常考虑各分类器的预测置信度, 选择置信度大的类别标记作为分类结果.
```

```ad-note
title: OvO & OvR
- OvR 只需训练 $N$ 个分类器, 而 OvO 需训练 $N(N-1)/2$ 个分类器, 因此, OvO 的**存储开销**和**测试时间开销**通常比 OvR 更大.
- 在训练时, OvR 的每个分类器均使用全部训练样例, 而 OvO 的分类器仅用到两个类的样例, 因此, 在类别很多时, OvO 的**训练时间开销**通常比 OvR 更小.
- 预测性能取决于具体的数据分布, 在多数情况下两者差不多.


![[OvO&OvR.svg|500]]
```

## 多对多 MvM

```ad-def
title: 多对多 MvM
MvM 是每次将若干个类作为正类, 其他若干个类作为反类. 显然 OvO 和 OvR 是 MvM 的特例.

MvM 的正, 反类构造必须有特殊的设计, 不能随意选取. 一种最常用的 MvM 技术为**纠错输出码** Error Correcting Output Codes, ECOC.
```

```ad-def
title: ECOC
ECOC \[Dietterich amd Bakiri, 1995\] 是将编码的思想引入类别拆分, 并尽可能在解码过程中具有容错性.

ECOC 的工作过程主要分为两步:
- 编码: 对 $N$ 个类别做 $M$ 次划分, 每次划分将一部分类别划为正类, 一部分划为反类, 从而形成一个二分类训练集; 这样一共产生 $M$ 个训练集, 可训练出 $M$ 个分类器.
- 解码: $M$ 个分类器分别对测试样本进行预测, 这些预测标记组成一个编码. 将这个预测编码与每个类别各自的编码进行比较, 返回其中距离最小的类别作为最终预测结果.

类别划分通过**编码矩阵** coding matrix 指定. 编码矩阵有多种形式, 常见的主要有**二元码** \[Dietterich amd Bakiri, 1995\] 和**三元码** \[Allwein et al., 2000\]. 前者将每个类别分别指定为正类和反类, 后者在正, 反类之外还可指定停用类.
```

```ad-example
title: ECOC 示意
![[ECOC.svg|500]]
- **编码**
	- 在 (a) 中, 分类器 $f_2$ 将 $C_1$ 类和 $C_3$ 类的样例作为正例, $C_2$ 类和 $C_4$ 类的样例作为反例.
	- 在 (b) 中, 分类器 $f_4$ 将 $C_1$ 类和 $C_4$ 类的样例作为正例, $C_3$ 类的样例作为反例.
- **解码**
	- 在 (a) 中, 基于[[#^c573c3|海明距离]]或[[#^ecefab|欧氏距离]], 预测结果为 $C_3$.
	- 在 (b) 中, 基于[[#^c573c3|海明距离]]或[[#^ecefab|欧氏距离]], 预测结果为 $C_2$.
```

```ad-def
title: 海明距离 Hamming Distance
记 $\boldsymbol{x} = (x_1, x_2, \cdots, x_m), \boldsymbol{y} = (y_1, y_2, \cdots, y_m)$. 定义函数 $E(\cdot, \cdot)$ 如下
$$
E(a, b) = \begin{cases}
0, & a=b\\
1, & a\neq b
\end{cases}
$$
则海明距离为
$$
\mathrm{Hamming\ Distance}(\boldsymbol{x}, \boldsymbol{y}) = \sum_{i = 1}^m E(x_i, y_i)
$$
```

^c573c3

```ad-def
title: 欧氏距离 Euclidean Distance
记 $\boldsymbol{x} = (x_1, x_2, \cdots, x_m), \boldsymbol{y} = (y_1, y_2, \cdots, y_m)$. 则欧氏距离为
$$
\mathrm{Euclidean\ Distance}(\boldsymbol{x}, \boldsymbol{y}) = \sqrt{\sum_{i=1}^m (x_i-y_i)^2}
$$
```

^ecefab

```ad-question
title: 为什么称为 "纠错输出码"?
因为在测试阶段, ECOC 码对分类器的错误具有一定的容忍和纠错能力. 例如上面 (a) 中对测试示例的正确预测编码是 $(-1, +1, +1, -1, +1)$, 假设在预测时某个分类器出错了, 例如 $f_2$ 出错从而导致了错误编码 $(-1, -1, +1, -1, +1)$, 但基于这个编码仍能产生正确的最终分类结果 $C_3$.
```

```ad-note
title: 码长与纠错能力
一般来说, 
- 对同一个学习任务, ECOC 编码越长, 纠错能力越强.
- 编码越长, 意味着所需训练的分类器越多, 计算/存储开销都会增大.
- 对有限类别数, 可能的组合数是有限的, 码长超过一定范围后就失去了意义.
```

```ad-note
title: 编码
对于同等长度的编码, 理论上来说, 任意两个类别之间的编码距离越远, 则纠错能力越强.
- 在码长较小时可根据这个原则计算出理论最优编码.
- 码长稍大一些就难以有效地确定最优编码, 这是 NP 难问题.

通常我们不需获得理论最优编码
- 非最优编码在实践中往往已能产生足够好的分类器.
- 并不是编码的理论性能越好, 分类性能就越好, 因为机器学习问题涉及很多因素.
	- 例如将很多类拆解为两个 "类别子集", 不同拆解方式所形成的两个类别子集的区分难度往往不同, 即导致的二分类问题的难度不同.
	- 需要在编码的理论性能和实际情况之间进行权衡.
```

# 类别不平衡问题
> 前面的分类算法基于一个共同的基本假设, 即不同类别的训练样例数目相当. 如果不同类别的训练样例数稍有差别, 通常影响不大, 但若差别很大, 则会对学习过程造成困扰.

```ad-def
title: 类别不平衡 class-imbalance
**类别不平衡** class-imbalance 就是指分类任务中不同类别的训练样例数目差别很大的情况.
```

```ad-attention
在现实的分类学习任务中, 经常会遇到类别不平衡, 例如在通过拆分法解决多分类问题时, 即使原始问题中不同类别的训练样例数相当, 在使用 OvR, MvM 策略后产生的二分类任务仍可能出现类别不平衡现象, 因此有必要了解类别不平衡性处理的基本方法.
```

```ad-def
title: 再缩放 Rescaling
从线性分类器的角度讨论, 在我们用 $y = g\left(\boldsymbol{w}^{\mathrm{T}}\boldsymbol{x}+b\right)$ 对新样本 $\boldsymbol{x}$ 进行分类时, 事实上是在用预测出的 $y$ 值与一个阈值进行比较, 例如 $y > 0.5$ 时判别为正例, 否则为反例. $y$ 实际上表达了正例的可能性, 几率 $y/(1-y)$ 则反映了正例可能性与反例可能性之比值, 阈值设置为 0.5 恰表明分类器认为真实正, 反例可能性相同, 即分类器决策规则为
$$
\frac{y}{1-y}>1\rightarrow \boldsymbol{x}\in C_+\tag{11}
$$
然而, 当训练集中正, 反例的数目不同时, 令 $m^+$ 表示正例数目, $m^-$ 表示反例数目, 则观测几率是 $m^+/m^-$, 由于我们通常假设**训练集是真实样本总体的无偏采样**, 因此观测几率就代表了真实几率. 于是, 只要分类器的预测几率高于观测几率就应判定为正例, 即
$$
\frac{y}{1-y}>\frac{m^+}{m^-}\rightarrow \boldsymbol{x}\in C_+\tag{12}
$$
但是我们的分类器是基于式 (11) 进行决策, 因此需对其预测值进行调整, 使其在基于 (11) 进行决策时实际上是在执行式 (12). 只需令
$$
\frac{y'}{1-y'}= \frac{y}{1-y}\times \frac{m^-}{m^+}\tag{13}
$$
这就是类别不平衡学习的一个基本策略 - **再缩放** rescaling.

再缩放的思想虽然简单, 但实际操作并不平凡, 因为 "训练集是真实样本总体的无偏采样" 这个假设往往不成立, 我们未必能有效地基于训练集观测几率来推断出真实几率.

再缩放也是**代价敏感学习** cost-sensitive learning 的基础. 再代价敏感学习中将式 (13) 中的 $m^-/m^+$ 用 $cost^-/cost^+$ 代替即可.
```

不失一般性, 假定正类样例较少, 反类样例较多. 现有技术大体上有三种做法:
- 对训练集里的反类样例进行**欠采样** undersampling: 去除一些反例使得正, 反例数目接近, 然后再进行学习.
- 对训练集里的正类样例进行**欠采样** oversampling: 增加一些正例使得正, 反例数目接近, 然后再进行学习.
- **阈值移动** threshold-moving: 直接基于原始训练集进行学习, 但在用训练好的分类器进行预测时, 将式 (13) 嵌入到其决策过程中.

```ad-note
title: 欠采样法与过采样法
- 欠采样法的时间开销通常远小于过采样法
	- 欠采样法丢弃了很多反例, 使得分类器训练集远小于初始训练集.
	- 过采样法增加了很多正例, 其训练集大于初始训练集.
- 欠采样法不能随机丢弃反例, 否则可能丢失一些重要信息.
	- 欠采样法的代表性算法 EasyEnsemble \[Liu et al., 2009\] 是利用集成学习机制, 将反例划分为若干个集合供不同学习器使用, 这样对每个学习器来看都进行了欠采样, 但在全局看来却不会丢失重要信息.
- 过采样法不能简单地对初始正例样本进行重复采样, 否则会导致严重的过拟合
	- 过采样法的代表性算法 SMOTE \[Chawla et al., 2002\] 是通过对训练集里的正例进行插值来产生额外的正例.
```

