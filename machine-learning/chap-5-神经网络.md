#计算机 #软件 #机器学习 

# 神经元模型
```ad-def
title: 神经网络 Neural Network
**神经网络**是由具有适应性的简单单元组成的广泛并行互连的网络, 它的组织能够模拟生物神经系统对真实世界物体所作出的交互反应.
```

神经网络中最基本的成分是**神经元模型** neuron model, 即上述定义中的简单部分. 

```ad-def
title: 神经元 Neuron
在生物神经网络中, 每个**神经元**与其他神经元相连, 当它*兴奋*时, 就会向相连的神经元发送化学物质, 从而改变这些神经元内的电位; 如果某神经元的电位超过了一个**阈值** threshold, 那么它就会被激活, 即*兴奋*起来, 向其他神经元发送化学物质.
```

> 1943 年, \[McCulloch and Pitts, 1943\] 将上述模型抽象为 **M-P 神经元模型**, 沿用至今.

```ad-def
title: M-P 神经元模型
如图, 神经元接收到来自 $n$ 个其他神经元传递过来的输入信号, 这些输入信号通过带权重的**连接** connection 进行传递, 神经元接收到的总输入值将与神经元的阈值进行比较, 然后通过**[[#^6a048d|激活函数]]** activation function 处理产生神经元的输出.

![[M-P-Neuron-Model.svg|500]]
```

^ca0236

```ad-example
title: 激活函数 Activation Function
理想中的激活函数是[[Chap 3. 线性模型#^baef7f|阶跃函数]] (注意这里 $f(0) = 1$), 它将输入值映射到 $\{0,1\}$, 显然 1 对应神经元兴奋, 0 对应神经元抑制, 如下所示.

实际常用 Sigmoid 函数作为激活函数, [[Chap 3. 线性模型#^2200c4|典型的 Sigmoid 函数]]如下所示, 它把可能在较大范围内变化的输入值挤压到 $(0,1)$, 因此有时也称**挤压函数** squashing function.

![[Activation-Function.svg|500]]

```

^6a048d

将许多个神经元按一定的层次连接起来, 就得到了神经网络.

```ad-abstract
title: 神经网络的数学模型
从计算机科学的角度看, 可以先不考虑神经网络是否真的模拟了生物神经网络, 只需将神经网络视为包含了许多参数的数学模型, 这个模型是若干个函数, 形如 $y_j = f\left(\sum_i w_ix_i - \theta_j\right)$ 相互 (嵌套) 代入而得.

> 有效的神经网络学习算法大多以数学证明为支撑.

```

# 感知机与多层网络
## 感知机
```ad-def
title: 感知机 Perceptron
**感知机**由两层神经元组成, 输入层接收外界输入信号后传递给输出层, 输出层是 [[#^ca0236|M-P 神经元]], 亦称**阈值逻辑单元** threshold logic unit.

![[Perceptron.svg]]
```

```ad-example
title: 实现逻辑与/或/非运算
感知机能容易地实现逻辑与/或/非运算. 假定激活函数 $f\left(\sum_i w_ix_i - \theta_j\right)$ 为阶跃函数.

- **与** $\bigwedge_{i=0}^n x_i$: 令 $w_1 = w_2 = \cdots = w_n = 1$, $\theta\in (n - 1, n]$ (这里取 $n$), 即 $$y = f\left(1\cdot x_1 + 1\cdot x_2 + \cdots + 1\cdot x_n - n\right)$$ 仅在 $x_1 = x_2 = \cdots = x_n = 1$ 时, $y = 1$.
- **或** $\bigvee_{i = 0}^n x_i$: 令 $w_1 = w_2 = \cdots = w_n = 1$, $\theta\in (0,1]$ 即可 (这里取 1), 即 $$y = f\left(1\cdot x_1 + 1\cdot x_2 + \cdots + 1\cdot x_n - 1\right)$$ 仅在 $x_1 = x_2 = \cdots = x_n = 0$ 时, $y = 0$.
- **非** $\lnot x_1$: 令 $w_1 < \theta < 0$ 即可 (比如 $w_1 = -0.6$, $\theta = -0.5$), 即 $$y = f(-0.6\cdot x_1 + 0.5)$$ 当 $x_1 = 1$ 时, $y = 0$; 当 $x_1 = 0$ 时, $y = 1$.
```

^d7c8c8

```ad-code
title: 感知机学习
给定训练数据集, 权重 $w_i (i = 1, 2, \cdots, n)$ 以及阈值 $\theta$ 可通过学习得到.

阈值 $\theta$ 可看作一个固定输入为 $-1.0$ 的*哑结点* dummy node, 所对应的连接权重 $w_{n+1}=\theta$.
- 将权重和阈值的学习统一为权重的学习.

学习规则: 对训练样例 $(\boldsymbol{x}, y)$, 若当前感知机的输出为 $\hat{y}$, 则感知机权重这样调整
$$
w_i \leftarrow w_i + \Delta w_i = w_i + \eta \left(y - \hat{y}\right)x_i
$$
- $\eta \in (0,1)$ 为**学习率** learning rate.
- 若感知机对样例预测正确, $\hat{y} = y$, 感知机不发生变化.
- 否则将根据错误的程度进行权重调整.
```

````ad-caution
title: 感知机的缺陷
感知机只有输出层神经元进行激活函数处理, 即只拥有一层*功能神经元* functional neuron, 其学习能力非常有限.

可以证明 \[Minsky and Papert, 1969\]:
- 若两类模式是*线性可分的* linearly separable (即存在一个线性超平面能将它们分开), 则感知机的学习过程一定会*收敛* converge 而求得适当的权向量 $\boldsymbol{w} = \left(w_1;w_2;\cdots;w_{n+1}\right)$.
- 否则感知机学习过程将会发生*振荡* fluctuation, $\boldsymbol{w}$ 难以稳定下来, 不能求得合适解.

例如上述[[#^d7c8c8|与/或/非问题]]都是线性可分的, 而感知机不能解决异或这样简单的非线性可分问题. 如下
```ad-example
title: 线性可分和非线性可分
![[Linearly-Separable.svg|400]]
```
````

## 多层网络
> 要解决非线性可分问题, 需考虑使用多层功能神经元.
```ad-example
title: 两层感知机解决异或问题
![[NN-XOR.svg|400]]

- **隐层/隐含层** hidden layer: 输入层和输出层之间的一层神经元
- 隐含层和输出层都是拥有激活函数的功能神经元.
```

```ad-def
title: 多层前馈神经网络 Multi-layer Feedforward Neural Networks
- 每层神经元与下一层神经元**全互连**
	- 神经元之间不存在*同层连接*, 也不存在*跨层连接*.
- **输入层**神经元接收外界输入
	- 不进行函数处理
- **隐层**与**输出层**神经元对信号进行加工
	- 包含功能神经元
- 最终结果由**输出层**神经元输出
- 神经网络的学习过程: 根据训练数据来调整神经元之间的**连接权** connection weight 以及每个功能神经元的阈值.

![[MFNN.svg]]
```

# 误差逆传播算法/反向传播算法
```ad-note
title: 误差逆传播算法 error BackPropagation
**误差逆传播算法** error BackPropagation, 亦称*反向传播算法*, 下文简称 BP 算法.
- 迄今最成功的神经网络学习算法.
- 现实任务中使用神经网络时, 大多是在使用 BP 算法进行训练.
- 不仅可用于多层前馈神经网络, 还可用于其他类型的神经网络.
```

```ad-code
title: 误差逆传播算法中的变量符号
- 训练集 $D = \left\{ \left( \boldsymbol{x}_{1}, \boldsymbol{y}_{1} \right), \left( \boldsymbol{x}_{2}, \boldsymbol{y}_{2} \right)), \cdots, \left( \boldsymbol{x}_{m}, \boldsymbol{y}_{m} \right) \right\}, \boldsymbol{x}_{i}\in\mathbb{R}^d, \boldsymbol{y}_{i}\in\mathbb{R}^l$
	- 输入示例由 $d$ 个属性描述, 输出 $l$ 维实值向量.

> 如下一个拥有 $d$ 个输入神经元, $l$ 个输出神经元, $q$ 个隐层神经元的多层前馈神经网络结构
> ![[BP-vars.svg|500]]

- 阈值
	- 输出层第 $j$ 个神经元的阈值用 $\theta_{j}$ 表示
	- 隐层第 $h$ 个神经元的阈值用 $\gamma_{h}$ 表示
- 连接权
	- 输入层第 $i$ 个神经元与隐层第 $h$ 个神经元之间的连接权为 $v_{ih}$
	- 隐层第 $h$ 个神经元与输出层第 $j$ 个神经元之间的连接权为 $w_{hj}$
- 输入/输出
	- 输入层接受并输出样例 $\boldsymbol{x}$, 第 $i$ 个神经元的输入/输出均为 $x_{i}$
	- 隐层第 $h$ 个神经元接受到的输入为 $\alpha_{h} = \sum_{i = 1}^d v_{ih}x_{i}$, 输出 $b_{h} = f\left( \alpha_{h} - \gamma_{h} \right)$
	- 输出层第 $j$ 个神经元接受到的输入为 $\beta_{j} = \sum_{h = 1}^q w_{hj}b_{h}$, 输出 $\hat{y}_{j} = f\left( \beta_{j} - \theta_{j} \right)$

---
- 矩阵简化运算
	- 记 $\mathbf{V} = \left( v_{ih} \right)_{d\times q}$, $\mathbf{W} = \left( w_{hj} \right)_{q\times l}$, $\boldsymbol{\gamma} = \left( \gamma_{1}, \gamma_{2}, \cdots, \gamma_{q}\right)$, $\boldsymbol{\theta} = \left( \theta_{1}, \theta_{2}, \cdots, \theta_{l} \right)$, $\left( f\left( \mathbf{A} \right) \right)_{ij} = f\left( A_{ij} \right)$.
	- 隐层输入为 $\boldsymbol{\alpha} = \boldsymbol{x}\mathbf{V}$, 输出为 $\boldsymbol{b} = f\left( \boldsymbol{\alpha} - \boldsymbol{\gamma} \right)$
	- 输出层输入为 $\boldsymbol{\beta} = \boldsymbol{b}\mathbf{W}$, 输出为 $\hat{\boldsymbol{y}} = f\left( \boldsymbol{\beta} -\boldsymbol{\theta} \right)$
```

```ad-code
title: 误差逆传递算法的参数更新

> 假设隐层和输出层神经元都使用 Sigmoid 函数 $f(x) = \dfrac{1}{1 + \mathrm{e}^{-x}}$

网络中有 $(d + l + 1)q + l$ 个参数需要确定:
- 输入层到隐层的 $d\times q$ 个权值
- 隐层到输出层的 $q\times l$ 个权值
- $q$ 个隐层神经元的阈值
- $l$ 个输出层神经元的阈值

BP 是一个迭代学习算法, 在迭代的每一轮中采用广义的感知机学习规则对参数进行更新估计: 
$$
v\gets v + \Delta v
$$

对训练例 $\left( \boldsymbol{x}_{k}, \boldsymbol{y}_{k} \right)$, 神经网络的输出为 $\hat{\boldsymbol{y}}_{k} = \left( \hat{y}_{1}^k, \hat{y}_{2}^k, \cdots, \hat{y}_{l}^k \right)$, 则网络在样例 $\left( \boldsymbol{x}_{k}, \boldsymbol{y}_{k} \right)$ 的均方误差为
$$
E_{k} = \frac{1}{2}\sum_{j = 1}^k\left( \hat{y}_{j}^k - y_{j}^k \right)^2 
$$ 
BP 算法基于**梯度下降** gradient descent 策略, 以目标的负梯度方向对参数进行调整, 给定学习率 $\eta$, 即得到
$$
\Delta v = -\eta \frac{ \partial E_{k} }{ \partial v } 
$$
因此就得到
$$
\begin{align}
\Delta w_{hj} &= -\eta \frac{ \partial E_{k} }{ \partial w_{hj} }  \\
\Delta \theta_{j} &= -\eta \frac{ \partial E_{k} }{ \partial \theta_{j} }  \\
\Delta v_{ih} &= -\eta \frac{ \partial E_{k} }{ \partial v_{ih} }  \\
\Delta \gamma_{h} &= -\eta \frac{ \partial E_{k} }{ \partial \gamma_{h} } 
\end{align}
$$
注意到:
- $w_{hj}$ 先影响到第 $j$ 个输出层神经元的输入值 $\beta_{j}$, 再影响到其输出值 $\hat{y}_{j}^k$, 然后影响到 $E_k$.
- $\theta_{j}$ 先影响到第 $j$ 个输出层神经元的输出值 $\hat{y}_{j}^k$, 再影响到 $E_k$.
- $v_{ih}$ 先影响到第 $h$ 个隐层神经元的输入值 $\alpha_{h}$, 再影响到其输出值 $b_{h}$, 然后影响到所有输出层神经元的输入值 $\left\{ \beta_{j} \right\}_{j = 1}^l$, 进而影响到所有输出层神经元的输出值 $\left\{ \hat{y}_{j}^k \right\}_{j = 1}^l$, 最后影响到 $E_{k}$.
- $\gamma_{h}$ 先影响到第 $h$ 个隐层神经元的输出值 $b_{h}$, 再影响到所有输出层神经元的输入值 $\left\{ \beta_{j} \right\}_{j = 1}^l$, 然后影响到所有输出层神经元的输出值 $\left\{ \hat{y}_{j}^k \right\}_{j = 1}^l$, 最后影响到 $E_{k}$.

就可知
$$
\begin{align}
\Delta w_{hj} &= -\eta \frac{ \partial E_{k} }{ \partial \hat{y}_{j}^k } \cdot \frac{ \partial \hat{y}_{j}^k }{ \partial \beta_{j} } \cdot \frac{ \partial \beta_{j} }{ \partial w_{hj} }   \\
\Delta \theta_{j} &= -\eta \frac{ \partial E_{k} }{ \partial \hat{y}_{j}^k } \cdot \frac{ \partial \hat{y}_{j}^k }{ \partial \theta_{j} } \\
\Delta v_{ih} &= -\eta \frac{ \partial E_{k} }{ \partial b_{h} } \cdot \frac{ \partial b_{h} }{ \partial \alpha_{h} } \cdot \frac{ \partial \alpha_{h} }{ \partial v_{ih} }  \\
\Delta \gamma_{h} &= -\eta \frac{ \partial E_{k} }{ \partial b_{h} } \cdot \frac{ \partial b_{h} }{ \partial \gamma_{h} }
\end{align}\tag{1}
$$
显然有
$$
\begin{align}
\frac{ \partial \beta_{j} }{ \partial w_{hj} } &= b_{h}\\ 
\frac{ \partial \alpha_{h} }{ \partial v_{ih} } &= x_{i}
\end{align}\tag{2}
$$
其中
$$
\frac{ \partial E_{k} }{ \partial b_{h} } = \sum_{j = 1}^l  \frac{ \partial E_{k} }{ \partial \hat{y}_{j}^k } \cdot \frac{ \partial \hat{y}_{j}^k }{ \partial \beta_{j} } \cdot \frac{ \partial \beta_{j} }{ \partial b_{h} } = \sum_{j = 1}^l \frac{ \partial E_{k} }{ \partial \hat{y}_{j}^k } \cdot \frac{ \partial \hat{y}_{j}^k }{ \partial \beta_{j} } w_{hj}\tag{3}
$$
选用的 Sigmoid 函数有一个很好的性质 $f'\left( x \right) = f\left( x \right)\left( 1 - f\left( x \right) \right)$, 就可知
$$
\begin{align}
\frac{ \partial \hat{y}_{j}^k }{ \partial \beta_{j} } &= -\frac{ \partial \hat{y}_{j}^k }{ \partial \theta_{j} } = \hat{y}_{j}^k\left( 1 - \hat{y}_{j}^k \right)  \\
\frac{ \partial b_{h} }{ \partial \alpha_{h} } &= -\frac{ \partial b_{h} }{ \partial \gamma_{h} } = b_{h}\left( 1 - b_{h} \right)
\end{align}\tag{4}
$$
而
$$
\frac{ \partial E_{k} }{ \partial \hat{y}_{j}^k } = y_{j}^k - \hat{y}_{j}^k\tag{5}
$$
记输出层神经元梯度项为 
$$
g_{j} = -\dfrac{ \partial E_{k} }{ \partial \hat{y}_{j}^k } \cdot \dfrac{ \partial \hat{y}_{j}^k }{ \partial \beta_{j} } = \hat{y}_{j}^k\left( 1 - \hat{y}_{j}^k \right)\left( y_{j}^k - \hat{y}_{j}^k \right)\tag{6}
$$
代入到式 (5) 就得到
$$
\frac{ \partial E_{k} }{ \partial b_{h} } = \sum_{j = 1}^l -g_{j} w_{hj}\tag{7}
$$
记隐层神经元梯度项为
$$
e_{h} = -\frac{ \partial E_{k} }{ \partial b_{h} } \cdot \frac{ \partial b_{h} }{ \partial \alpha_{h} } = b_{h}\left( 1 - b_{h} \right)\sum_{j = 1}^l g_{j} w_{hj} \tag{8}
$$
将 (2, 4, 6, 8) 式代入 (1) 式, 即可得到
$$
\begin{align}
\Delta w_{hj} &= \eta g_{j}b_{h} \\
\Delta \theta_{j} &= -\eta g_{j} \\
\Delta v_{ih} &= \eta e_{h}x_{i} \\
\Delta \gamma_{h} &= -\eta e_{h}
\end{align}\tag{9}
$$
> 学习率 $\eta\in(0, 1)$ 控制着算法每一轮迭代中的更新步长
> - 太大容易振荡
> - 太小收敛速度过慢
> - 常设置 $\eta = 0.1$
> 
> 有时为了做精细调节, 会让 $\Delta w_{hj}$ 和 $\Delta \theta_{j}$ 使用 $\eta_{1}$, $\Delta v_{ih}$ 和 $\Delta \gamma_{h}$ 使用 $\eta_{2}$, 二者未必相等.

---
- 矩阵简化运算
	- 当前样本输出向量 $\hat{\boldsymbol{y}}_{k} = f\big( f\left( \boldsymbol{x}_{k}\mathbf{V} - \boldsymbol{\gamma} \right)\mathbf{W} -\boldsymbol{\theta} \big)$
	- 均方误差 $E_{k} = \dfrac{1}{2} \left( \boldsymbol{y}_{k} - \hat{\boldsymbol{y}}_{k} \right)\left( \boldsymbol{y}_{k} - \hat{\boldsymbol{y}}_{k} \right)^\mathrm{T}$
	- 输出层梯度向量 $\boldsymbol{g} = \hat{\boldsymbol{y}}_{k}\left( \mathbf{1} - \hat{\boldsymbol{y}}_{k} \right)^\mathrm{T}\left( \boldsymbol{y}_{k} - \hat{\boldsymbol{y}}_{k} \right)$
	- 隐层梯度向量 $\boldsymbol{e} = \boldsymbol{b}\left( \mathbf{1} - \boldsymbol{b} \right)^\mathrm{T}\boldsymbol{g}\mathbf{W}$
	- 参数向量变化量
		- $\Delta\mathbf{W} = \eta \boldsymbol{b}^\mathrm{T}\boldsymbol{g}$
		- $\Delta \boldsymbol{\theta} = -\eta \boldsymbol{g}$
		- $\Delta\mathbf{V} = \eta \boldsymbol{x}^\mathrm{T}\boldsymbol{e}$
		- $\Delta \boldsymbol{\gamma} = -\eta \boldsymbol{e}$
```

```ad-code
title: 误差逆传递算法 (标准 BP 算法)
![[algo-2.svg|600]]
1. 将输入示例提供给输入层神经元, 然后逐层将信号前传, 直到产生输出层的结果.
2. 计算输出层的误差, 再将误差逆向传播至隐层神经元.
3. 根据隐层神经元的误差来对连接权和阈值进行调整.
4. 迭代过程循环进行, 直到达到某些条件为止.
	- 例如训练误差已达到一个很小的值
```

```ad-note
title: 累积误差逆传播算法 (累积 BP 算法)
需要注意 BP 算法的目标是要最小化训练集 $D$ 上的累积误差 $E = \dfrac{1}{m}\sum_{k = 1}^m E_{k}$, 但是上面的*标准* BP 算法每次仅针对一个训练样例更新连接权和阈值, 即更新规则是基于单个的 $E_{k}$ 推导而得.

如果类似地推导出基于累积误差最小化的更新规则, 就得到了**累积误差逆传播** accumulated error backpropagation 算法.

> 累积 BP 算法和标准 BP 算法都很常用

- 标准 BP 算法每次更新只针对单个样例, 参数更新得非常频繁, 而且对不同样例进行更新的效果可能出现*抵消*现象.
	- 为了达到同样的累积误差极小点, 标准 BP 算法往往需进行更多次数的迭代.
- 累积 BP 算法直接针对累积误差最小化, 它在读取整个训练集 $D$ 一遍后才对参数进行更新, 其参数更新的频率低很多.
	- 但在很多任务中, 累积误差下降到一定程度之后, 进一步下降会非常缓慢,
		- 这时标准 BP 往往会更快获得较好的解, 尤其是在训练集 $D$ 非常大时更明显.
- 小批量随机梯度下降法 Mini-Batch Stochastic Gradient Descent $$\frac{ \partial E }{ \partial \boldsymbol{\theta} } = \frac{1}{m} \sum_{k = 1}^m \frac{ \partial E_{k} }{ \partial \boldsymbol{\theta} } \approx \frac{1}{n} \sum_{k = 1}^n \frac{ \partial E_{k} }{ \partial \boldsymbol{\theta} }$$ $n \ll m$, 称为 batch size.
```

```ad-thm
\[Hornik et al., 1989\] 证明, 只需一个包含足够多神经元的隐层, 多层前馈神经网络就能以**任意精度**逼近**任意复杂度**的连续函数.
- 如何设置隐层神经元的数量仍是个未决问题, 实际应用中通常靠**试错法** trial-by-error 调整.
```

由于其强大的表示能力, BP 神经网络经常遭遇过拟合, 其训练误差持续降低, 但测试误差却可能上升.

```ad-note
title: 过拟合处理
有两种策略常用来缓解 BP 网络的过拟合
- **早停** early stopping
	- 将数据分成训练集和验证集, 训练集用来计算梯度/更新连接权和阈值, 验证集用来估计误差, 若训练集误差降低但验证集误差升高, 则停止训练, 同时返回具有最小验证集误差的连接权和阈值.
- **正则化** regularization
	- 基本思想是在误差目标函数中增加一个用于描述网络复杂度的部分.
	- 例如连接权与阈值的平方和.
		- 仍令 $E_{k}$ 表示第 $k$ 个训练样例上的误差, $w_{i}$ 表示连接权和阈值, 则误差目标函数改变为 $$E = \lambda\frac{1}{m}\sum_{k = 1}^m E_{k} + \left( 1 - \lambda \right) \sum_{i} w_{i}^2$$ 其中 $\lambda\in\left( 0,1 \right)$ 用于对经验误差与网络复杂度这两项进行折中, 常通过交叉验证法来估计.
```