#计算机 #软件 #机器学习 

# 经验误差与过拟合
## 经验误差
- 错误率 (error rate): 分类错误的样本数占样本总数的比例
- 精度 (accuracy): 分类正确的样本数占样本总数的比例

```ad-example
如果在 $m$ 个样本中有 $a$ 个样本分类错误, 则错误率 $E = a/m$, 精度为 $1-a/m$
```

- 误差 (error): 学习器的实际预测输出与样本的真实输出之间的差异
	- 训练误差 (training error)/经验误差 (empirical error): 学习器在[[chap-1-绪论#^5ec70b|训练集]]上的误差
	- 泛化误差 (generalization error): 学习器在新样本上的误差

我们希望得到泛化误差小的学习器, 但是我们实际能做的是努力使经验误差最小化.

## 过拟合与欠拟合
我们希望得到在新样本上表现得很好的学习器. 因此应该从[[chap-1-绪论#^29c889|训练样本]]中尽可能学出适用于所有潜在样本的 "普遍规律".
- 过拟合 (overfitting): 学习器把训练样本自身的一些特点当作了所有潜在样本都会具有的一般性质, 导致[[chap-1-绪论#^10a1e4|泛化]]能力下降
	- 过拟合使机器学习面临的关键障碍, 各类学习算法都必有一些针对过拟合的措施
	- 过拟合是无法完全避免的, 我们所能做的只是 "缓解"
- 欠拟合 (underfitting): 学习器对训练样本的一般性质尚未学好
	- 通常由于学习能力低下造成, 比较容易克服

## 模型选择问题
- 模型选择 (model selection) 问题: 在现实任务中, 往往有多种学习算法可供选择, 甚至对同一个学习算法, 使用不同的参数配置时, 也会产生不同的模型. 应该使用哪一种学习算法, 使用哪一种参数配置呢?
- 解决方案: 对候选模型的泛化误差进行评估, 然后选择泛化误差最小的那个模型.
	- 无法直接获得泛化误差
	- 训练误差存在过拟合问题, 不适合作为标准

# 评估方法
我们可以使用实验测试来对学习器的泛化误差进行评估并进而做出选择
- 测试集 (testing set): 用于测试学习器对新样本的判别能力
- 测试误差 (testing error): 用于作为泛化误差的近似

```ad-note
- 通常我们假设测试样本也是从样本真实分布中独立同分布采样而得.
- 测试集应该尽可能与训练集互斥
- 我们只有一个包含 $m$ 个样例的数据集 $D=\{(\boldsymbol{x}_1, y_1),\cdots,(\boldsymbol{x}_m, y_m)\}$, 既要训练, 又要测试, 需要通过对 $D$ 进行适当的处理, 从中产生训练集 $S$ 和测试集 $T$
```
## 留出法
**留出法** (hold-out): 直接将数据集 $D$ 划分为两个互斥的集合, 其中一个集合作为训练集 $S$ 另一个作为测试集 $T$, 即 $D=S\cup T, S\cap T = \varnothing$
- 训练/测试集的划分要尽可能保持数据分布的一致性, 避免因数据划分过程引入额外的偏差而对最终结果产生影响
	- 例如在分类过程中至少要保持样本的类别比例相似
	> 从采样 (sampling) 的角度看待数据集的划分过程, 保留类别比例的采样方式通常称为 "分层采样" (stratified sampling)
- 即便在给定训练/测试集的样本比例后, 仍存在多种划分方式对初始数据集 $D$ 进行分割
	- 单次使用留出法得到的估计结果往往不够稳定可靠.
	- 在使用留出法时, 一般要采用若干次随即划分, 重复进行实验评估后取平均值作为留出法的评估结果
- 比例问题
	- 若令训练集 $S$ 包含绝大多数样本, 则训练出的模型可能更接近于用 $D$ 训练出的模型, 但由于 $T$ 比较小, 评估结果可能不够稳定准确
	- 若令测试集 $T$ 多包含一些样本, 则测试集 $S$ 与 $D$ 差别更大了, 被评估的模型与用 $D$ 训练出的模型相比可能有较大差别, 从而降低了评估结果的保真性 (fidelity)
	- 没有完美的解决方案, 常见的作法是将大约 $2/3\sim 4/5$ 的样本用于训练, 剩余样本用于测试

## 交叉验证法
```ad-note
title:交叉验证法 (cross validation)
- 先将数据集 $D$ 划分为 $k$ 个大小相似的互斥子集 ($D=D_1\cup D_2\cup\cdots\cup D_k, D_i\cap D_j=\varnothing(i\neq j)$)
	- 每个子集 $D_i$ 都尽可能保持数据分布的一致性, 即从 $D$ 中分层采样得到
- 然后每次用 $k-1$ 个子集的并集作为训练集, 余下的那个子集作为测试集
	- 这样就可以获得 $k$ 组训练/测试集, 从而可以进行 $k$ 次训练和测试
- 最后返回的是这 $k$ 个测试结果的均值.

交叉验证法评估结果的稳定性和保真性很大程度上取决于 $k$ 的取值, 为强调这一点, 通常把交叉验证法称为 "**$k$ 折交叉验证**" ($k$-fold cross validation)
- $k$ 最常用的取值为 10

与留出法, 类似将数据集 $D$ 划分为 $k$ 个子集同样存在多种划分方式. 为减小因样本划分不同而引入的差别, $k$ 折交叉验证通常要随机使用不同的划分重复 $p$ 次, 最终的评估结果是这 $p$ 次 $k$ 折交叉验证结果的均值.
- 常见的有 "10 次 10 折交叉验证"
```

```ad-note
title:留一法 (Leave-One-Out, LOO)
假定数据集 $D$ 中包含 $m$ 个样本, 若令 $k=m$, 则得到了交叉验证法的一个特例: **留一法**
- 留一法不受随机样本划分方式的影响
- 留一法中被实际评估的模型与期望评估的用 $D$ 训练出的模型很相似
- 留一法的评估结果往往被认为比较准确
- 在数据集较大时, 留一法的计算开销是难以忍受的
```
```ad-info
[[Chap 1. 绪论#^c516ab|NFL 定理]] 对模型评估方法同样适用.
```

## 自助法
```ad-question
- 留出法和交叉验证法中实际评估的模型所使用的训练集比 $D$ 小, 必然会引入一些因训练样本规模不同而导致的估计偏差
- 留一法的计算复杂度太高了

有没有什么办法可以减少训练样本规模不同造成的影响, 同时还能比较高效地进行实验估计呢?
```
自助法是一个比较好的解决方案
```ad-note
title:自助法 (bootstrapping)
- 直接以自助采样法 (bootstrap sampling) 为基础 \[Efron and Tibshirani, 1993\]
- 给定包含 $m$ 个样本的数据集 $D$, 对它进行采样产生数据集 $D'$
	- 每次随机从 $D$ 中挑选一个样本, 将其拷贝放入 $D'$, 然后再将该样本放回初始数据集 $D$ 中, 使得该样本在下次采样时仍可能被采到
	- 将上面的过程重复 $m$ 次, 就得到了包含 $m$ 个样本的数据集 $D'$
	- $D$ 中有一部分样本会在 $D'$ 中多次出现, 而另一部分不出现.
		- 样本在采样中始终不被采到的概率为: $\displaystyle\lim_{m\to\infty}\left(1-\frac{1}{m}\right)^m=\frac{1}{\mathrm{e}}\approx0.368$
- 将 $D'$ 作为训练集, $D-D'$ 作为测试集
	- 实际评估的模型与期望评估的模型都使用 $m$ 个训练样本
	- 仍有数据总量越 1/3 的, 没在训练集中出现的样本用于测试
- 这样的测试结果亦称 "**包外估计**" (out-of-bag estimate)
```
- 自助法在数据集较小, 难以有效划分训练/测试集时很有用
- 自助法能从初始数据集中产生多个不同的训练集, 对集成学习等方法有很大的好处
- 自助法产生的数据集改变了初始数据集的分布, 会引入估计偏差

```ad-info
在初始数据量够时, 一般采用留出法和交叉验证法
```

## 调参与最终模型
- 大多数学习算法都有些**参数** (parameter) 需要设定.
	- 参数配置不同, 学得模型的性能往往有显著差别
- **参数调节**/**调参** (parameter tuning): 在进行模型评估与选择时, 除了要对适用学习算法进行选择, 还需对算法参数进行设定
- 测试数据: 学得模型在实际使用中遇到的数据
- **验证集** (validation set): 模型评估与选择中用于评估测试的数据集

# 性能度量
```ad-note
title: 性能度量 (performance measure)
对学习器的泛化性能进行评估, 不仅需要有效可行的实验估计方法, 还需要有衡量模型泛化能力的评价标准, 这就是性能度量
```
在预测任务中, 给定样例集 $D=\{(\boldsymbol{x}_1, y_1),(\boldsymbol{x}_2, y_2),\cdots,(\boldsymbol{x}_m, y_m)\}$, 其中 $y_i$ 是 $\boldsymbol{x}_i$ 的真实标记. 要评估学习器 $f$ 的性能, 就要把学习器预测结果 $f(\boldsymbol{x})$ 与真实标记 $y$ 进行比较.


**回归任务**最常用的性能度量是均方误差
```ad-note
title: 均方误差 (mean square error)
$$
E(f;D)=\frac{1}{m}\sum_{i=1}^m(f(\boldsymbol{x}_i)-y_i)^2
$$
更一般地, 对于数据分布 $\mathcal{D}$ 和概率密度函数 $p(\cdot)$, 均方误差可描述为
$$
E(f;D)=\int_{\boldsymbol{x}\sim\mathcal{D}}(f(\boldsymbol{x}_i)-y_i)^2p(\boldsymbol{x})\mathrm{d}\boldsymbol{x}
$$
```

^b0298d

下面主要介绍**分类任务**中常用的性能度量
## 错误率与精度
```ad-note
title: 分类错误律
$$
E(f;D)=\frac{1}{m}\sum_{i=1}^m\mathbb{I}(f(\boldsymbol{x}_i\neq y_i))
$$
更一般地, 对于数据分布 $\mathcal{D}$ 和概率密度函数 $p(\cdot)$, 错误率可描述为
$$
E(f;D)=\int_{\boldsymbol{x}\sim\mathcal{D}}\mathbb{I}(f(\boldsymbol{x}_i)\neq y_i)p(\boldsymbol{x})\mathrm{d}\boldsymbol{x}
$$
```
```ad-note
title: 精度
$$
E(f;D)=\frac{1}{m}\sum_{i=1}^m\mathbb{I}(f(\boldsymbol{x}_i=y_i))=1-E(f;D)
$$
更一般地, 对于数据分布 $\mathcal{D}$ 和概率密度函数 $p(\cdot)$, 精度可描述为
$$
E(f;D)=\int_{\boldsymbol{x}\sim\mathcal{D}}\mathbb{I}(f(\boldsymbol{x}_i)=y_i)p(\boldsymbol{x})\mathrm{d}\boldsymbol{x}=1-E(f;D)
$$
```
## 查准率, 查全率与 F1
```ad-note
title: 混淆矩阵 (confusion matrix)
对于二分类问题, 可将样例根据其真实类别与学习器预测类别的组合划分为
- 真正例 (true positive, TP)
- 假正例 (false positive, FP)
- 真反例 (true negative, TN)
- 假反例 (false negative, FN)

$$
TP+FP+TN+FN = \text{样例总数}
$$


分类结果的混淆矩阵如下所示

~~~tx
| 真实情况 | 预测结果   ||
| ^^      | 正例 | 反例 |
|:-------:|:----:|:----:|
|   正例   |  TP  |  FN  |
|   反例   |  FP  |  TN  |
~~~
```
```ad-note
title: 查准率 (precision)/查全率 (recall)
查准率 $P$ 与查全率 $R$ 分别定义为
$$
P=\frac{TP}{{TP+FP}}\qquad R=\frac{TP}{{TP+FN}}
$$
```

查准率和查全率是一对矛盾的变量
- 一般来说, 查准率高时, 查全率往往偏低; 而查全率高时, 查准率往往偏低.
- 通常只有在一些简单的任务中, 才可能使查准率和查全率都很高.
  
```ad-note
title: P-R 曲线
我们可根据学习器的预测结果对样例进行排序, 排在前面的是学习器认为 "最可能" 是正例的样本, 排在最后的则是学习器认为 "最不可能" 是正例的样本. 按此顺序逐个把样本作为正例进行进行预测, 则每次可以计算出当前的查全率, 查准率. 以**查准率**为**纵轴**, **查全率**为**横轴**作图, 就得到了**查准率-查全率曲线**, 简称 "P-R 曲线", 显示该曲线的图为 "P-R 图"
```

^9359fb

- 如果一个学习器的 P-R 曲线被另一个学习器的曲线完全 "包住", 则可断言后者的性能优于前者
- 如果两个学习器的 P-R 曲线发生了交叉, 则难以一般性地断言孰优孰劣, 只能在具体的查准率或查全率条件下进行比较

然而很多情况下我们仍然需要对两个学习器进行比较
- 一个比较合理的判据是比较 P-R 曲线下面积的大小, 但这个值不好估算
- **平衡点** (Break-Even Point, BEP): "查准率 = 查全率" 时的取值
- F1 度量:  $$F1=\frac{2\times P\times R}{P+R}=\frac{2\times TP}{Total + TP - TN}$$

```ad-note
title: $F_\beta$ 度量
F1 度量的一般形式
$$
F_\beta=\frac{(1+\beta^2)\times P\times R}{(\beta^2\times P)+R}
$$
$\beta>0$ 度量了查全率对查准率的相对重要性
- $\beta=1$ 时退化为标准的 F1
- $\beta>1$ 时查全率有更大影响
- $\beta<1$ 时查准率有更大影响
```

有时我们有多个二分类混淆矩阵
- 多次训练/测试
- 在多个数据集上训练/测试
- 多分类任务, 每两两类别的组合都对应一个混淆矩阵
- …

我们希望在 $n$ 个二分类混淆矩阵上综合考察查准率与查全率

```ad-note
title: 宏查准率, 宏查全率, 宏 F1 (macro-P/R/F1)
先在各混淆矩阵上分别计算出查准率和查全率, 记为 $(P_1, R_1), (P_2, R_2),\cdots, (P_n, R_n)$, 再计算平均值, 得到
$$
\begin{gather}
	\text{macro-}P=\frac{1}{n}\sum_{i=1}^nP_i\\
	\text{macro-}R=\frac{1}{n}\sum_{i=1}^nR_i\\
	\text{macro-}F1=\frac{2\times \text{macro-}P\times \text{macro-}R}{\text{macro-}P+\text{macro-}R}
\end{gather}
$$
```
```ad-note
title: 微查准率, 微查全率, 微 F1 (micro-P/R/F1)
先将各混淆矩阵的对应元素进行平均, 得到 $\overline{TP}$, $\overline{FP}$, $\overline{TN}$, $\overline{FN}$, 再基于这些平均值计算出
$$
\begin{gather}
	\text{micro-}P=\frac{\overline{TP}}{{\overline{TP}+\overline{FP}}}\\
	\text{micro-}R=\frac{\overline{TP}}{{\overline{TP}+\overline{FN}}}\\
	\text{micro-}F1=\frac{2\times \text{micro-}P\times \text{micro-}R}{\text{micro-}P+\text{micro-}R}
\end{gather}
$$
```

## ROC 与 AUC
- **真正例率** (True Positive Rate, TPR): $\displaystyle TPR=\frac{TP}{{TP+FN}}$
- **假正例率** (False Positive Rate, FPR): $\displaystyle FPR=\frac{FP}{TN+FP}$ ^617c9b
- ROC 曲线的定义与[[#^9359fb|P-R 曲线]]类似, 其中纵轴为 TPR, 横轴为 FPR.
	- 利用有限个测试样例来绘制 ROC 图
		1. 给定 $m^+$ 个正例和 $m^-$ 个反例, 根据学习器预测结果进行排序
		2. 将分类阈值设为最大, 即把所有样例均预测为反例, 此时 $(x_0,y_0) = (FPR, TPR) = (0,0)$, 在 $(0,0)$ 处标记一个点
		3. 将分类阈值依次设为每个样例的预测值, 即依次将每个样例划分为正例
			1. 若为真正例, $(x_i, y_i)=\left(x_{i-1}, y_{i-1}+ \frac{1}{m^+}\right)$
			2. 若为假正例, $(x_i, y_i)=\left(x_{i-1}+ \frac{1}{m^-}, y_{i-1}\right)$
		3. 用线段连接相邻各点即可
	- 若 ROC 曲线为对角线, 则模型为 "随机猜测" 模型
	- 若 ROC 曲线过点 $(0, 1)$, 则模型将所有正例排在所有反例之前, 是"理想模型
- Area Under ROC Curve, AUC: ROC 曲线下的面积, 用于比较学习器之间的优劣, 一般有$$\text{AUC}=\frac{1}{2}\sum_{i=1}^{m-1}(x_{i+1}-x_i)(y_i+y_{i+1})$$其中 $(x_i, y_i)$ 为绘制 ROC 曲线时所作各点坐标
- 排序 "损失" (loss): 对每一对正/反例, 若正例的预测值小于反例, 则记一个 "罚分", 若相等, 则记 0.5 个 "罚分". 
	- 给定 $m^+$ 个正例和 $m^-$ 个反例, 令 $D^+$ 和 $D^-$ 分别表示正, 反例集合 $$\mathscr{l}_{rank}=\frac{1}{m^+m^-}\sum_{\boldsymbol{x^+}\in D^+}\sum_{\boldsymbol{x}\in D^-}\left(\mathbb{I}\left(f(\boldsymbol{x}^+)<f(\boldsymbol{x}^-)\right)+\frac{1}{2}\mathbb{I}\left(f(\boldsymbol{x}^+)=f(\boldsymbol{x}^-)\right)\right)$$
	- $\text{AUC}=1-\mathscr{l}_{rank}$

## 代价敏感错误率与代价曲线
为权衡不同类型错误所造成的损失, 可为错误赋予 "非均等代价" (unequal cost)

```ad-note
title: 代价矩阵 cost matrix
以二分类任务为例, 我们可根据任务的领域知识设定一个代价矩阵, 如下表, 其中 $cost_{ij}$ 表示将第 $i$ 类样本预测为第 $j$ 类样本的代价.

~~~tx
| 真实类别 | 预测类别                 ||
|   ^^    |   第 0 类   |   第 1 类   |
|:-------:|:-----------:|:-----------:|
| 第 0 类 |      0      | $cost_{01}$ |
| 第 1 类 | $cost_{10}$ |      0      |
~~~
- 一般来说, $cost_{ii}=0$
- 若将第 0 类判别为第 1 类所造成的损失更大, 则 $cost_{01} > cost_{10}$, 损失程度相差越大, 代价值的差别越大
```

```ad-info
前面的性能度量大多都隐式地假设了均等代价.
```

- 在非均等代价下, 我们希望最小化 "总体代价" (total cost).
- "代价敏感" (cost-sensitive) 错误率 $$ E(f;D;cost)=\frac{1}{m}\sum_{\boldsymbol{x}_i\in D} cost\left({f(\boldsymbol{x}_i), y_i}\right) $$

```ad-note
title: 代价曲线 cost curve
代价曲线可以直接反映出学习器的期望总体代价
- 横轴: 取值为 $[0,1]$ 的正例概率代价$$P(+)cost=\frac{p\times cost_{01}}{p\times cost_{01}+(1-p)\times cost_{10}}$$
	- $p$ 是样例为正例的概率
- 纵轴: 取值为 $[0,1]$ 的归一化代价 $$cost_{norm}=\frac{FNR\times p\times cost_{01}+FPR\times (1-p)\times cost_{10}}{p\times cost_{01}+(1-p)\times cost_{10}}$$
	- FPR: [[#^617c9b|假正例率]]
	- FNR: 假反例率, FNR = 1 - TPR
- 绘制
	1. ROC 曲线上的每一点对应了代价平面上的一条线段
		1. 设 ROC 曲线上点的坐标为 $(FPR,TPR)$, 则可相应计算出 $FNR$
		2. 在代价平面上绘制一条从 $(0,FPR)$ 到 $(1,FNR)$ 的线段, 线段下的面积即表示了该条件下的期望总体代价
	2. 如此将 ROC 曲线上的每一点转化为代价平面上的一条线段
	3. 取所有线段的下界, 围成的面积即为在所有条件下学习器的期望总体代价.
```

# 比较检验
统计[[数理统计#假设检验|假设检验]] (hypothesis test) 为学习器性能比较提供了重要的依据.
## 二项检验
```ad-tip
- 对单个学习器的性能的假设进行检验
- 适用于单次留出法
```

假定测试样本是从样本总体中独立采样而得, 那么泛化错误率为 $\epsilon$ 的学习器被测得错误率为 $\hat\epsilon$ 的概率为
$$
P(\hat\epsilon;\epsilon)=
\begin{pmatrix}
m \\ \hat\epsilon\times m
\end{pmatrix}
\epsilon^{\hat\epsilon\times m}(1-\epsilon)^{m-\hat\epsilon\times m}
$$
这符合二项 (binomial) 分布,  $P(\hat\epsilon;\epsilon)$ 在 $\epsilon=\hat\epsilon$ 时最大, $\lvert\epsilon-\hat\epsilon\rvert$ 增大时 $P(\hat\epsilon;\epsilon)$ 减小.

我们可用**二项检验** (binomial test) 来对假设 $\epsilon\le\epsilon_0$ 进行检验, 在 $\alpha$ 的显著度下 (即置信度为 $1-\alpha$), 记临界值
$$
\bar\epsilon=\min \epsilon\quad \mathrm{s.t.}\quad\sum_{i=\epsilon\times m+1}^m\begin{pmatrix}m \\ i\end{pmatrix}\epsilon_0^i(1-\epsilon_0)^{m-i}<\alpha
$$

- 若测试错误率 $\hat\epsilon$ 大于临界值 $\bar\epsilon$, 则在 $\alpha$ 的显著度下假设 $\epsilon\le\epsilon_0$ 不能被拒绝, 即能以 $1-\alpha$ 的置信度认为学习器的泛化错误率不大于 $\epsilon_0$
- 否则该假设可被拒绝, 即在 $\alpha$ 的显著度下可认为学习器的泛化错误率大于 $\epsilon_0$
## t 检验
```ad-tip
- 对单个学习器的性能的假设进行检验
- 适用于多次重复留出法或交叉验证法
```
假定我们得到了 $k$ 个测试错误率, $\hat\epsilon_1, \cdots, \hat\epsilon_k$, 则平均测试错误率 $\mu$ 和方差 $\sigma^2$ 为
$$
\begin{gather}
\mu = \frac{1}{k}\sum_{i = 1}^k\hat\epsilon_i\\
\sigma^2=\frac{1}{k - 1}\sum_{i = 1}^k (\hat\epsilon_i-\mu)^2
\end{gather}
$$
考虑到这 $k$ 个测试错误率可看作泛化误差率 $\epsilon_0$ 的独立采样, 则
$$
\tau_t=\frac{\sqrt{k}(\mu - \epsilon_0)}{\sigma}
$$
服从自由度为 $k - 1$ 的 [[数理统计#t 分布 Student 分布|t 分布]]. 对假设 $\mu=\epsilon_0$ 和显著度 $\alpha$
- 若 $\tau_t\in\left[t_{-\alpha/2},t_{\alpha/2}\right]$, 则不能拒绝假设 $\mu = \epsilon_0$, 即可认为泛化错误律为 $\epsilon_0$, 置信度为 $1-\alpha$
- 否则可拒绝该假设, 即在该显著度下可认为泛化错误律与 $\epsilon_0$ 显著不同.
## 交叉验证 t 检验
```ad-tip
- 对两个学习器的性能进行比较
- 适用于交叉验证法
```
对于两个学习器, 若我们使用 $k$ 折交叉验证法得到的测试错误率分别为 $\epsilon_1^A,\cdots,\epsilon_k^A$ 和 $\epsilon_1^B, \cdots, \epsilon_k^B$, 其中 $\epsilon_i^A$ 和 $\epsilon_i^B$ 是在相同的第 $i$ 折训练/测试集上得到的结果, 则可用 $k$ 折交叉验证**成对 $t$ 检验** (paired $t$-tests) 来进行比较检验. 
```ad-note
title: 成对 $t$ 检验
基本思想为[[数理统计#成对数据的假设检验]].
1. 对每对结果求差, $\Delta_i=\epsilon_i^A-\epsilon_i^B$
2. 计算出差值的均值 $\mu$ 和方差 $\sigma^2$
3. 有 $\tau_t=\displaystyle\frac{\sqrt{k}\mu}{\sigma}\sim t(k - 1)$
```
欲进行有效的假设检验, 一个重要的前提是测试错误率均为泛化错误率的独立采样. 然而, 通常情况下由于样本有限, 在使用交叉验证等实验估计方法时, 不同轮次的训练集会有一定程度的重叠, 这就使得测试错误率实际上并不独立, 会导致过高估计假设成立的概率. 为缓解这一问题, 可采用 **$5\times2$ 交叉验证法**
```ad-note
title: $5\times2$ 交叉验证法
1. 做 5 次 2 折交叉验证, 每次 2 折交叉验证之前随机将数据打乱, 使得 5 次交叉验证中的数据划分不重复.
2. 对两个学习器 A 和 B, 第 $i$ 次 2 折交叉验证将产生;两对测试错误率, 我们对它分别求差, 得到第 1 折上的差值 $\Delta_i^1$ 和第 2 折上的差值 $\Delta_i^2$
3. 对每次 2 折实验的结果都计算出其方差 $$\sigma_i^2=\left(\Delta_i^1-\frac{\Delta_i^1+\Delta_i^2}{2} \right)^2+\left(\Delta_i^2-\frac{\Delta_i^1+\Delta_i^2}{2} \right)^2$$
4. 有 $$\tau_t=\frac{\Delta_1^1}{\sqrt{0.2\sum_{i = 1}^{5}\sigma_i^2}}\sim t(5)$$
```
## McNemar 检验
```ad-tip
- 对两个学习器的性能进行比较
- 针对二分类问题
```
对于二分类问题, 使用留出法不仅可估计出学习器 $A$ 和 $B$ 的测试错误率, 还可获得两学习器分类结果的差别, 即两者都正确, 都错误, 一个正确另一个错误的样本数, 如下**列联表** (contingency table)

```tx
| 算法 B | 算法 A           ||
| ^^   |   正确   |   错误   |
|:----:|:--------:|:--------:|
| 正确 | $e_{00}$ | $e_{01}$ |
| 错误 | $e_{10}$ | $e_{11}$ |
```
若我们做的假设是两学习器性能相同, 则应有 $e_{01}=e_{10}$, 那么变量 $e_{01}-e_{10}$ 应当服从正态分布. McNemar 检验考虑变量
$$
\tau_{\chi^2}=\frac{(\lvert e_{01}-e_{10}\rvert-1)^2}{e_{01}+e_{10}}\sim \chi^2(1)
$$
```ad-info
$e_{01}+e_{10}$ 通常很小, 需考虑连续性校正, 因此分子中有 $-1$ 项.
```
## Friedman 检验 与 Nemenyi 后续检验
```ad-tip
- 在一组数据集上对多个算法进行比较
```
```ad-note
title: 基于算法排序的 Friedman 检验
假定我们用 $D_1$, $D_2$, $D_3$ 和 $D_4$ 四个数据集对算法 A, B, C 进行比较.
1. 使用留出法或交叉验证法得到每个算法在每个数据集上的测试结果
2. 在每个数据集上更具测试性能由好到坏排序, 并赋予序值 $1, 2, \cdots$, 若算法的测试性能相同, 则平分序值

例如, 在 $D_1$ 和 $D_3$ 上, A 最好, B 其次, C 最差, 而在 $D_2$ 上, A 最好, B 与 C 性能相同. 如下表, 其中最后一行通过对每一列的序值求平均, 得到平均序值

|  数据集  | 算法 A | 算法 B | 算法 C |
|:--------:|:------:|:------:|:------:|
|  $D_1$   |   1    |   2    |   3    |
|  $D_2$   |   1    |  2.5   |  2.5   |
|  $D_3$   |   1    |   2    |   3    |
|  $D_4$   |   1    |   2    |   3    |
| 平均序值 |   1    | 2.125  | 2.875  | 

3. 使用 Friedman 检验来判断这些算法是否性能相同. 若相同, 则它们的平均序值应当相同. 假定我们在 $N$ 个数据集上比较 $k$ 个算法, 令 $r_i$ 表示第 $i$ 个算法的平均序值. 为简化讨论, 暂不考虑平分序值的情况, 则 $r_1$ 的均值和方差分别为 $(k+1)/2$ 和 $(k^2-1)/12N$
4. 变量 $$\begin{aligned}\tau_{\chi^2}&=\frac{k-1}{k}\cdot \frac{12N}{k^2 - 1}\sum_{i = 1}^k\left(r_i - \frac{k + 1} {2}\right)^2\\&=\frac{12N}{k(k+1)}\left(\sum_{i=1}^kr_i^2-\frac{k(k+1)^2}{4}\right)\end{aligned}$$在 $k$ 和 $N$ 都较大时, 服从自由度为 $k-1$ 的 $\chi^2$ 分布
5. 上面的 *原始 Friedman 检验* 过于保守, 现在通常使用变量 $$\tau_F=\frac{(N-1)\tau_{\chi^2}}{N(k-1)-\tau_{\chi^2}}\sim F\left(k-1, \left(k-1\right)\left(N-1\right)\right)$$
```
若 *所有算法的性能相同* 这个假设被拒绝, 则说明算法的性能显著不同. 这时需进行**后续检验** (post-hoc test) 来进一步区分各算法. 常用的有 Nemenyi 后续检验.
```ad-note
title: Nemenyi 后续检验
Nemenyi 检验计算出平均序值差别的临界值域
$$
CD=q_\alpha\sqrt{\frac{k(k+1)}{6N}}
$$
若两个算法的平均序值只差超过了临界值域 $CD$, 则以相应的置信度拒绝 *两个算法性能相同* 这一假设.

下面是常用的 $q_\alpha$ 值
~~~tx
| $\alpha$ |                算法个数 $k$                                   |||||||||
|    ^^    |   2   |   3   |   4   |   5   |   6   |   7   |   8   |   9   | 10    |
|:--------:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:| ----- |
|   0.05   | 1.960 | 2.344 | 2.569 | 2.728 | 2.850 | 2.949 | 3.031 | 3.102 | 3.164 |
|   0.1    | 1.645 | 2.052 | 2.291 | 2.459 | 2.589 | 2.693 | 2.780 | 2.855 | 2.920 |
~~~

```
# 偏差与方差
对学习算法除了通过实验估计其泛化性能, 人们往往还希望了解它 "为什么" 具有这样的性能. **偏差-方差分解** (bias-variance decomposition) 是解释学习算法泛化性能的一种重要工具.

对测试样本 $\boldsymbol{x}$, 令 $y_D$ 为 $\boldsymbol{x}$ 在数据集中的标记, $y$ 为 $\boldsymbol{x}$ 的真实标记, $f(\boldsymbol{x};D)$ 为训练集 $D$ 上学得模型 $f$ 在 $\boldsymbol{x}$ 上的预测输出.

以回归任务为例,
- 期望预测 $\bar f(\boldsymbol{x})=\mathbb{E}_D[f(\boldsymbol{x}; D)]$
- 使用样本数相同的不同训练集产生的方差为 $\displaystyle var(\boldsymbol{x})=\mathbb{E}_D\left[\left(f(\boldsymbol{x};D)-\bar f(\boldsymbol{x})\right)^2\right]$
- 噪声 $\varepsilon^2=\mathbb{E}_D\left[(y_D-y)^2\right]$
- 期望输出与真实标记的差别称为偏差 (bias), $bias^2(\boldsymbol{x})=\left(\bar f(\boldsymbol{x})-y\right)^2$

经过数学运算对期望泛化误差进行分解, 有
$$
E(f;D)=\mathbb{E}_D\left[\left(f(\boldsymbol{x};D)-\bar f(\boldsymbol{x})\right)^2\right]+\mathbb{E}_D\left[\left(\bar f(\boldsymbol{x})-y\right)^2\right]+\mathbb{E}_D\left[(y_D-y)^2\right]
$$
即泛化误差可分解为偏差, 方差和噪声之和
$$
E(f;D)=bias^2(\boldsymbol{x})+var(\boldsymbol{x})+\varepsilon^2
$$

- 偏差度量了学习算法的期望预测与真实结果的偏离程度, 即刻画了学习算法本身的拟合能力
- 方差度量了同样大小的训练集的变动所导致的学习性能的变化, 即刻画了数据扰动造成的影响
- 噪声则表达了在当前任务上任何学习算法所能达到的期望泛化误差的下界, 即刻画了学习任务本身的难度.

```ad-attention
title: 偏差-方差窘境 bias-variance dilemma
一般来说, 偏差与方差是由冲突的
- 在训练不足时, 学习器的拟合能力不够强, 训练数据的扰动不足以使学习器产生显著变化, 此时**偏差**主导了泛化错误率
- 随着训练程度的加深, 学习器的拟合能力逐渐增强, 训练数据发生的扰动渐渐能被学习器学到, **方差**逐渐主导了泛化错误率
- 在训练充足后, 学习器的拟合能力已非常强, 训练集发生的轻微扰动都会导致学习器发生显著变化, 若训练数据自身的, 非全局的特性被学习器学到了, 则将发生过拟合
```