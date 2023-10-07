#计算机 #软件 #机器学习 

# 基本流程
```ad-def
title: 决策树 Decision Tree
**决策树** decision tree 是一类常见的机器学习方法. 

以二分类任务为例, 希望从给定训练数据集学得一个模型用以对新示例进行分类, 这个把样本分类的任务, 可看作对*当前样本属于正类吗?* 这个问题的**决策**或**判定**过程.

决策树是基于树结构来进行决策的, 这恰是人类在面临决策问题时的一种很自然的处理机制.
- 决策过程的最终结果对应类我们所希望的判定结果.
- 决策过程中提出的每个问题都是对某个属性的 "测试".
- 每个测试的结果或是到处最终结论, 或是导出进一步的判定问题.
	- 其考虑范围是在上次决策结果的限定范围之内.
```

```ad-note
title: 决策树的基本结构
一般地, 一棵决策树包含一个根结点, 若干内部结点和若干叶结点.
- 叶结点对应于决策结果, 其他每个结点对应与一个属性测试.
- 每个结点包含的样本集合根据属性测试的结果被划分到子结点中.
- 根结点包含样本全集.

从根结点到每个叶结点的路径对应了一个判定测试序列.

![[DT-Example.svg#center|决策树的一个例子]]
```

```ad-code
title: 决策树学习基本算法
决策树的学习的目的是为了产生一棵泛化能力强, 即处理未见示例能力强的决策树, 其基本流程遵循简单的**分而治之** [[Chap 1. Recursion#The Pattern|divide-and-conquer]] 策略, 如下.
![[algo-1.svg|600]]

有下面三种情况对结点进行标记:
1. 当前结点包含的样本全属于同一类别, 无需划分;
2. 当前属性集为空, 或是所有样本在所有属性上取值相同, 无法划分;
	- 利用当前结点的后验分布设定类别.
3. 当前结点包含的样本合集为空, 不能划分.
	- 把父节点的样本分布作为当前结点的先验分布设定类别.
```

# 划分选择
从[[algo-1.svg|算法 1]] 可以看出, 决策树学习的关键是第 10 行, 即如何选择最优划分属性.

一般而言, 随着划分过程不断进行, 我们希望决策树的分支结点所包含的样本尽可能属于同一类别, 即结点的**纯度** purity 越来越高.

## 信息增益
**信息熵** information entropy 是度量样本集合纯度最常用的一种指标.
```ad-def
title: 信息熵 Information Entropy
假定当前样本集合 $D$ 中第 $k$ 类样本所占的比例为 $p_k$ ($k = 1, 2, \cdots, \lvert\mathcal{Y}\rvert$), 则 $D$ 的信息熵定义为
$$
\mathrm{Ent}(D) = -\sum^{\lvert\mathcal{Y}\rvert}_{k=1}p_k\log_2 p_k
$$
$\mathrm{Ent}(D)$ 的值越小, 则 $D$ 的纯度越高.
```

^c690e1

假定离散属性 $a$ 有 $V$ 个可能的取值 $\{a^1, a^2, \cdots, a^V\}$, 若使用 $a$ 来对样本集 $D$ 进行划分, 则会产生 $V$ 个分支结点. 
- 其中第 $v$ 个分支结点包含了 $D$ 中所有在属性 $a$ 上取值为 $a^v$ 的样本, 记为 $D^v$.
- 计算出 $D^v$ 的信息熵.
- 再考虑到不同的分支结点所包含的样本数不同, 给分支结点赋予权重 $\lvert D^v\rvert/\lvert D\rvert$.
	- 即样本数越多的分支结点的影响越大.

就可计算出用属性 $a$ 对样本集 $D$ 进行划分所获得的**信息增益** information gain.
```ad-def
title: 信息增益 Information Gain
$$
\mathrm{Gain}(D, a) = \mathrm{Ent}(D) - \sum^{V}_{v = 1}\frac{\lvert D^v\rvert}{\lvert D\rvert}\mathrm{Ent}(D^v)
$$
```

^e21e12

- 一般而言, 信息增益越大, 则意味着使用属性 $a$ 来进行划分所获得的 "纯度提升" 越大.
- 因此可用信息增益来进行决策树的划分属性选择, 即在[[algo-1.svg|算法 1]] 的第 10 行选择 $$a_* = \arg\max_{a\in A}\mathrm{Gain}(D, a)$$
- 著名的 ID3 决策树学习算法 \[Quinlan, 1986\] 就是以信息增益为准则来选择划分属性.
- 信息增益准则对可取数值较多的属性有所偏好.
	- 例如编号.
	- 可能带来不利影响.

## 增益率
```ad-def
title: 固有值 Intrinsic Value
$$
\mathrm{IV}(a) = - \sum^{V}_{v = 1}\frac{\lvert D^v\rvert}{\lvert D\rvert}\log_2\frac{\lvert D^v\rvert}{\lvert D\rvert}
$$
称为属性 $a$ 的**固有值** intrinsic value \[Quinlan, 1993\]. 属性 $a$ 的可能取值数目越多 ($V$ 越大), 则 $\mathrm{IV}(a)$ 的值通常会越大.
```

```ad-def
title: 增益率 Gain Ratio
$$
\mathrm{Gain\_ratio}(D,a) = \frac{\mathrm{Gain}(D, a)}{\mathrm{IV}(a)} 
$$
```

- 增益率准则对可取数值较少的属性有所偏好.
- 著名的 C4.5 决策树算法 \[Quinlan\] 不直接采用信息增益, 而是使用**增益率** gain ratio 来选择划分最优属性.
	- 减小了信息增益准则偏好可能带来的不利影响.
	- 不是直接选择增益率最大的候选划分属性.
	- **启发式**: 先从候选划分属性中找出信息增益高于平均水平的属性, 再从中选择增益率最高的.

## 基尼指数
```ad-def
title: 基尼值 Gini Value
$$
\mathrm{Gini}(D) = \sum^{\lvert\mathcal{Y}\rvert}_{k = 1}\sum_{k'\neq k}p_k p_{k'} = 1 - \sum^{\lvert\mathcal{Y}\rvert}_{k = 1}p_k^2
$$
- 直观来说, 基尼值反映了从数据集 $D$ 中随机抽取两个样本, 其类别标记不一致的概率.
- $\mathrm{Gini}(D)$ 越小, 数据集 $D$ 的纯度越高.
```
```ad-def
title: 基尼指数 Gini Index
$$
\mathrm{Gini\_index}(D, a) = \sum^{V}_{v = 1}\frac{\lvert D^v\rvert}{\lvert D\rvert}\mathrm{Gini}(D^v)
$$
```
- 可用基尼指数来进行决策树的划分属性选择, 基尼指数越小, 纯度提升越大. $$a_*=\underset{a\in A}{\mathrm{argmin}}\ \mathrm{Gini\_index}(D, a)$$
- 著名的 CART 决策树 \[Breiman et al., 1984\] 就使用基尼指数来选择划分属性.
	- CART - Classification and Regression Tree.
	- 分类和回归任务都可用.

# 剪枝处理
```ad-note
title: 剪枝处理过拟合
**剪枝** pruning 是决策树学习算法对付 "过拟合" 的主要手段. 

在决策树学习中, 为了尽可能正确分类训练样本, 结点划分过程将不断重复, 有时会造成决策树分支过多, 这时就可能因训练样本学得 "太好" 了, 以至于把训练集自身的一些特点当作所有数据都具有的一般性质导致过拟合.

可通过主动去掉一些分支来降低过拟合的风险.
```

剪枝的基本策略有**预剪枝** pre-pruning 和**后剪枝** post-pruning \[Quinlan, 1993\].

```ad-question
title: 如何判断决策树泛化性能是否提升?
可以使用 [[Chap 2. 模型评估与选择#评估方法]], 划分出测试集用于 "验证".
```

```ad-def
title: 预剪枝 Pre-pruning
在决策树生成过程中, 对每个结点在划分前先进行估计, 若当前结点的划分不能带来决策树泛化性能提升, 则停止划分并将当前结点标记为叶结点.
```

```ad-note
title: 预剪枝的利弊
- 使得决策树的很多分支都没有 "展开".
	- 降低了过拟合的风险.
	- 显著减少了决策树的训练时间开销和测试时间开销.
- 有些分支的当前划分虽不能提升泛化性能, 甚至可能导致泛化性能暂时下降, 但在其基础上进行的后续划分却有可能导致性能显著提高.
- 基于 "贪心" 本质禁止分支展开, 给预剪枝决策树带来了欠拟合的风险.
```

```ad-def
title: 后剪枝 Post-pruning
先从训练集生成一棵完整的决策树, 然后自底向上地对非叶结点进行考察, 若将该结点对应的子树替换为叶结点能带来决策树泛化性能的提升, 则将该子树替换为叶结点.
```

```ad-note
title: 后剪枝的利弊
- 通常比预剪枝决策树保留了更多分支.
- 一般情况下, 欠拟合风险很小, 泛化性能远远优于预剪枝决策树.
- 训练时间开销比未剪枝决策树和预剪枝决策树都要大很多.
```

# 连续与缺失值
## 连续值处理
```ad-question
title: 如何在决策树学习中使用连续属性?
不能直接根据连续属性的可取值来对结点进行划分, 需要使用连续属性离散化技术.
```

最简单的策略是采用**二分法** bi-partition 对连续值属性进行处理
- C4.5 决策树算法中采用的机制 \[Quinlan, 1993\].

```ad-def
title: 二分法 Bi-partition
给定样本集 $D$ 和连续属性 $a$, 假定 $a$ 在 $D$ 上出现了 $n$ 个不同的取值, 从小到大排列为 $\{a^1, a^2, \cdots, a^n\}$.

基于划分点 $t$ :
- 可将 $D$ 分为子集 $D_t^-$ 和 $D_t^+$.
	- $D_t^-$ 包含在属性 $a$ 上取值不大于 $t$ 的样本.
	- $D_t^+$ 包含在属性 $a$ 上取值大于 $t$ 的样本.
- 对相邻的属性取值 $a^i$ 和 $a^{i+1}$ 来说, $t$ 在区间 $[a^i, a^{i+1})$ 中取任意值所产生的划分结果相同.
	- 因此可取中位点 $\displaystyle \frac{a^i+a^{i+1}}{2}$ 作为候选划分点.

对于连续属性 $a$ 可以考察包含 $n - 1$ 个元素的候选划分点集合:
$$
T_a = \left\{\left.\frac{a^i + a^{i+1}}{2}\right\vert 1 \leq i \leq n - 1 \right\}
$$
可[[#划分选择|像离散值一样来考察]]这些划分点, 取最优的划分点进行样本集合的划分. 
```

```ad-example
title: 信息增益改造
例如对[[#^e21e12|信息增益]]进行改造
$$
\begin{align}
\mathrm{Gain}(D, a) &= \max_{t\in T_a} \mathrm{Gain}(D, a, t)\\
&= \max_{t\in T_a} \mathrm{Ent}(D) - \sum_{\lambda\in\{-, +\}}\frac{\lvert D_t^\lambda\rvert}{\lvert D\rvert}\mathrm{Ent}(D_t^\lambda)
\end{align}
$$
- $\mathrm{Gain}(D, a, t)$ 是样本集 $D$ 基于划分点 $t$ 二分后的信息增益.
- 选择使 $\mathrm{Gain}(D, a, t)$ 最大化的划分点.
```

```ad-caution
title: 注意
与离散属性不同, 若当前结点划分属性为连续属性, 该属性还可作为其后代结点的划分属性.
```

## 缺失值处理
```ad-question
title: 如何对不完整 (某些属性值缺失) 样本进行处理?
- 若简单地放弃不完整样本, 仅使用无缺失值的样本来进行学习, 是对数据信息的极大浪费.
	- 在属性数目较多的情况下, 往往有大量样本出现缺失值.
- 需要解决两个问题
	1. 如何在属性值缺失的情况下进行划分属性选择?
	2. 给定划分属性, 若样本在该属性上的值缺失, 如何对样本进行划分?
```

````ad-note
title: 解决问题 1
给定训练集 $D$ 和属性 $a$, 令 $\tilde{D}$ 表示 $D$ 中在属性 $a$ 上没有缺失值的样本子集.
- 对问题 1, 显然仅可根据 $\tilde{D}$ 来判断属性 $a$ 的优劣.

假定属性 $a$ 有 $V$ 个可取值 $\{a^1, a^2, \cdots, a^V\}$.
- 令 $\tilde{D}^v$ 表示 $\tilde{D}$ 中在属性 $a$ 上取值为 $a^v$ 的样本子集.
- 令 $\tilde{D}_k$ 表示 $\tilde{D}$ 中属于第 $k$ 类 ($k = 1, 2, \cdots, \lvert\mathcal{Y}\rvert$) 的样本子集.

显然有
$$
\tilde{D} = \bigcup_{k = 1}^{\lvert\mathcal{Y}\rvert}\tilde{D}_k = \bigcup_{v=1}^V \tilde{D}^v
$$

我们为每个样本 $\boldsymbol{x}$ 都赋予一个相同的权重 $w_\boldsymbol{x}$, 并定义
$$
\begin{align}
\rho &= \frac{\sum_{\boldsymbol{x}\in\tilde{D}}w_{\boldsymbol{x}}}{\sum_{\boldsymbol{x}\in D}w_{\boldsymbol{x}}} \\
\tilde{p}_k &= \frac{\sum_{\boldsymbol{x}\in\tilde{D}_k}w_{\boldsymbol{x}}}{\sum_{\boldsymbol{x}\in \tilde{D}}w_{\boldsymbol{x}}}\quad (1 \le k \le \lvert\mathcal{Y}\rvert) \\
\tilde{r}_v &= \frac{\sum_{\boldsymbol{x}\in\tilde{D}^v}w_{\boldsymbol{x}}}{\sum_{\boldsymbol{x}\in \tilde{D}}w_{\boldsymbol{x}}}\quad (1 \le v \le V)
\end{align}
$$
- $\rho$ 表示对属性 $a$ 无缺失值样本所占的比例.
- $\tilde{p}_k$ 表示无缺失值样本中第 $k$ 类所占的比例.
- $\tilde{r}_v$ 表示无缺失值样本中在属性 $a$ 上取值为 $a^v$ 的样本所占的比例.

```ad-example
title: 推广信息增益
可将[[#^e21e12|信息增益]]推广为
$$
\begin{align}
\mathrm{Gain}(D, a) &= \rho \times \mathrm{Gain}(\tilde{D}, a)\\
&= \rho \times \left(\mathrm{Ent}(\tilde{D}) - \sum^{V}_{v = 1}\tilde{r}_v\mathrm{Ent}(\tilde{D}^v)\right)
\end{align}
$$
其中[[#^c690e1|信息熵]]为
$$
\mathrm{Ent}(\tilde{D}) = -\sum^{\lvert\mathcal{Y}\rvert}_{k=1}\tilde{p}_k\log_2 \tilde{p}_k
$$
```
```` 
```ad-note
title: 解决问题 2
每个样本 $\boldsymbol{x}$ 在最初都被赋予一个相同的权重 $w_\boldsymbol{x}$. 设样本 $\boldsymbol{x}$ 在当前结点的权值为 $w'_\boldsymbol{x}$.
- 若样本 $\boldsymbol{x}$ 在划分属性 $a$ 上的取值已知
	- 将 $\boldsymbol{x}$ 划入与其取值对应的子结点
	- 样本权值在子结点中保持 $w'_\boldsymbol{x}$.
- 若样本 $\boldsymbol{x}$ 在划分属性 $a$ 上的取值未知
	- 将 $\boldsymbol{x}$ 同时划入所有子结点.
	- 样本权值在与属性值 $a^v$ 对应的子结点中调整为 $\tilde{r}_v\cdot w'_\boldsymbol{x}$.

在最后统计所有类的权值, 将样本 $\boldsymbol{x}$ 划入权值最大的类中. 直观地看, 就是让一个样本以不同的概率划入到不同的子结点, 然后选取最终分类中概率最高的那一个.
```

# 多变量决策树
```ad-def
title: 分类边界
若将每个属性视为坐标空间中的一个坐标轴, 则 $d$ 个属性描述的样本就对应了 $d$ 维空间的一个数据点, 对样本分类则意味着在这个坐标空间中寻找不同类之间的分类边界.
```

```ad-example
title: 决策树的分类边界
![[DT-Boundary-Tree.svg|+grid]]
![[DT-Boundary-Boundary.svg|+grid]]
```

```ad-note
title: 单变量决策树的分类边界
- 单变量决策树所形成的分类边界有一个明显的特点: **轴平行** axis-parallel, 即分类边界由若干个与坐标轴平行的分段组成.
	- 使得分类结果有着较好的可解释性.
		- 每一段划分都直接对应了某个属性取值.
	- 在学习任务的真实分类边界比较复杂时, 必须使用很多段划分才能获得较好的近似.
		- 决策树会相当复杂.
		- 要进行大量的属性测试, 预测时间开销会很大.
```

```ad-example
title: 决策树对复杂分类边界的分段近似
![[Complex-Boundaries.svg|400]]
若能实现如红色线段所示的斜分类边界, 则决策树模型将大为简化.
```

```ad-def
title: 多分类决策树 Multivariate Decision Tree
**多变量决策树** multivariate decision tree 是能实现 "斜划分" 甚至更复杂划分的决策树.

以实现斜划分的决策树为例
- 非叶结点不再是仅针对某个属性, 而是对属性的线性组合进行测试.
- 每个叶结点是一个形如 $\sum^{d}_{i = 1}w_i a_i = t$ 的线性分类器, 权重 $w_i$ 和 $t$ 可在该结点所含的样本集和属性集上学得.
- 在学习过程中不是为每个属性寻找一个最优划分属性, 而是试图建立一个合适的线性分类器.
```

```ad-example
title: 多变量决策树及其分类边界
![[MDT-Tree.svg|+grid]]
![[MDT-CB.svg|+grid]]
```