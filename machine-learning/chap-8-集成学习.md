#  个体与集成
```ad-def
集成学习 ensemble learning 通过构建并结合多个学习器来提升性能.
- 同质集成
- 异质集成

集成学习获得比单一学习器显著优越的泛化性能, 对**弱学习器**尤为明显.
```

```ad-note
title: 简单分析
考虑二分类问题, 假设基分类器的错误率为:
$$
P \left( h(\boldsymbol{x}) \neq f(\boldsymbol{x}) \right) = \epsilon
$$
假设集成通过简单投票法结合 $T$ 个分类器
$$
H(\boldsymbol{x}) = \operatorname{sign} \left( \sum_{i}^T h_{i}(\boldsymbol{x}) \right)
$$
假设基分类器的错误率相互独立, 令 $n(T)$ 表示 $T$ 个基分类器中对 $\boldsymbol{x}$ 预测正确的个数. 
- 若有超过半数的基分类器正确则分类就正确

$$
P \left( H(\boldsymbol{x}) \neq f(\boldsymbol{x}) \right) = P(n(T) \leq \lceil T / 2\rceil) = \sum_{k = 0}^{\lceil T / 2\rceil} 
\begin{pmatrix}
T  \\
k
\end{pmatrix}
(1 - \epsilon)^k \epsilon^{T - k}
$$
由 Hoeffding 不等式 
$$
\begin{align}
& P \left( \dfrac{n(T)}{T} \leq \mathbf{E} \left[ \dfrac{n(T)}{T} \right] - \delta \right) \leq \exp(-2\delta^2T) \\
& \implies P \left( n(T) \leq \mathbf{E} \left[ n(T)\right] - \delta T \right) \leq \exp(-2\delta^2T) \\
& \implies P(n(T) \leq k) \leq \exp \left( -2 \frac{(\mathbf{E}[n(T)] - k)^2}{T} \right) 
\end{align}
$$
可得
$$
\begin{align}
P \left( H(\boldsymbol{x}) \neq f(\boldsymbol{x}) \right) = P(n(T) \leq \lceil T / 2\rceil
\end{align}
$$
```