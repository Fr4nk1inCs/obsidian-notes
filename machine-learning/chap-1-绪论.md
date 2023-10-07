#计算机 #软件 #机器学习

# 引言
什么是机器学习 (machine learning)?
- 机器学习致力于研究如何通过计算的手段, 利用经验来改善系统自身的性能
	- "经验" 通常以 "数据" 的形式存在
- 学习算法 (learning algorithm): 在计算机上从数据中产生 "模型" (model) 的算法

> 形式化定义 (Given by Mitchell, 1977)
> 
> 假设用 P 来评估计算机程序在某任务类 T 上的性能, 若一个程序通过利用经验 E 在 T 中任务上获得了性能改善, 则我们就说关于 T 和 P, 该程序对 E 进行了学习

# 基本术语
- 数据集 (data set): 一组记录的集合
- 示例 (instance)/样本 (sample): 数据集中的每条记录, 它是关于一个事件或者对象的描述
- 属性 (attribute)/特性 (feature): 反映事件或对象某方面的表现或性质的事项
	- 属性值 (attribute value): 属性上的取值
	- 属性空间 (attribute space)/样本空间 (sample space)/输入空间: 属性张成的空间
		- 特征向量 (feature vector): 每个示例都可在属性空间中对应一个向量

> 用 $D=\{\boldsymbol{x}_1, \boldsymbol{x}_2,\cdots, \boldsymbol{x}_m\}$ 表示包含 $m$ 个示例的数据集, 每个示例由 $d$ 个属性描述, 则每个示例 $\boldsymbol{x}_i=(x_{i1};x_{i2};\cdots;x_{id})$ 是 $d$ 维样本空间 $\mathcal{X}$ 中的一个向量, $\boldsymbol{x}_i\in\mathcal{X}$, 其中 $x_ij$ 是 $x_i$ 在第 $j$ 个属性上的取值, $d$ 称为样本 $\boldsymbol{x}_i$ 的 "**维数**" (dimensionality)

- 学习 (learning)/训练 (training): 从数据中学得模型的过程
	- 训练数据 (training data): 训练过程使用的数据
	- 训练样本 (training sample): 训练数据中的每个样本 ^29c889
	- 训练集 (training set): 训练样本组成的集合 ^5ec70b
	- 假设 (hypothesis): 学得模型, 它对应了关于数据的某种潜在的规律
	- 真相/真实 (ground truth): 模型所对应的规律本身
	- 学习器 (learner): 模型, 可以看作学习算法在给定数据和参数空间上的实例化

- 预测 (prediction): 模型对于未见样本给出的结果
- 标记 (label): 关于示例结果的信息
- 样例 (example): 拥有标记信息的示例
- 标记空间 (label space)/输出空间: 所有标记的集合

> 用 $(\boldsymbol{x}_i, y_i)$ 表示第 $i$ 个样例.<br>
> 其中 $y_i\in\mathcal{Y}$ 是示例 $\boldsymbol{x}_i$ 的标记, $\mathcal{Y}$ 是标记空间.
- 学习任务
	- 分类 (classification): 欲预测的是离散值
		- 二分类 (binary classification): 只涉及两个类别</br>
		通常称其中一个类是正类 (positive class), 另一个类是反类 (negative class)
		- 多分类 (multi-class classification): 涉及多个类别
	- 回归 (regression): 欲预测的是连续值
	
> 预测任务是希望通过对训练集 $\{(\boldsymbol{x}_1, y_1),(\boldsymbol{x}_2, y_2),\cdots,(\boldsymbol{x}_m, y_m)\}$ 进行学习, 建立一个从输入空间 $\mathcal{X}$ 到输出空间 $\mathcal{Y}$ 的映射 $f:\mathcal{X}\mapsto\mathcal{Y}$.
> - 对二分类任务, $\lvert\mathcal{Y}\rvert = 2$, 通常令 $\mathcal{Y}=\{-1,+1\}\text{ 或 }\{0,1\}$
> - 对多分类任务, $\lvert\mathcal{Y}\rvert > 2$
> - 对回归任务, $\mathcal{Y}=\mathbb{R}$

- 聚类 (clustering)
	- 将数据集中的样本分成若干组, 每个组称为一个 "簇"(cluster)

- 根据训练数据是否拥有标记信息
	- 监督学习 (supervised learning): 有
		- 分类
		- 回归
		- etc.
	- 无监督学习 (unsupervised learning): 无
		- 聚类
		- etc.
- 泛化 (generalization) 能力: 学得模型适用于新样本的能力 ^10a1e4

# 假设空间
- 归纳 (induction): 从特殊到一般的 "泛化" (generalization) 过程
	- 从样例中学习被称为 "归纳学习" (inductive learning)
- 演绎 (deduction): 从一般到特殊的 "特化" (specialization) 过程


- 归纳学习
	- 广义: 从概念中学习
	- 狭义: 从训练数据中学得概念 (concept), 亦称 "概念学习" 或 "概念形成"
- 机械学习: 记住训练样本

可以把学习过程看作一个在所有假设 (hypothesis) 组成的空间中进行搜索的过程, 搜索的目标是找到与数据集 "匹配" (fit) 的假设, 即能够将训练集中的样本判断正确的假设. 假设的表示一旦确定, 假设空间及其规模大小就确定了.

> 假设样本空间 $\mathcal{X}$ 的维数为 $d$, 每个属性可能的取值有 $n_i$ 个, 则假设空间的大小为 $1 + \prod_{i=1}^d (n_i + 1)$, 其中单独的 1 表示需要形成的概念根本不存在, 记为 $\varnothing$, 各项中所加的 1 表示通配, 即概念与该属性无关.

现实问题中我们常面临很大的样本空间, 但学习过程是基于有限的样本训练集进行的, 因此可能有多个假设与训练集一致, 即存在着一个与训练集一致的假设集合, 称为 "版本空间" (version space).

# 归纳偏好
> 对一个训练集, 会有多个与训练集一致的假设

- 归纳偏好 (inductive bias): 机器学习算法在学习过程中对某种类型假设的偏好
	- 任何一个有效的机器学习算法都必有其归纳偏好, 否则它将被假设空间中看似在训练集上 "等效" 的假设所迷惑, 而无法产生确定的学习结果
- "奥卡姆剃刀" (Occam's razor): 若有多个假设与与观察一致, 则选最简单的那个
	- 奥卡姆剃刀本身存在不同的诠释, 使用奥卡姆剃刀并不平凡
	- 什么是 "简单" 的假设?
- NFL 定理 (No Free Lunch Theorem, "没有免费的午餐" 定理): 任何两个学习算法的期望性能都是相同的 ^c516ab
	- **重要前提**: 所有 "问题" 出现的机会都是相同的
	- 我们只关注自己试图解决的问题 (具体的任务), 希望为它找到一个解决方案, 而对它在其他问题, 甚至是相似问题上的性能并不关心.
	- 意义: 脱离具体问题, 空泛地谈论 "什么算法更好" 毫无意义.学习算法自身的归纳偏好与问题是否匹配往往会起到决定性的作用.
# 发展历程
略

# 应用现状
略