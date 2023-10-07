# 线性代数
## 基础概念
### 范数
```ad-def
title: 向量范数
- $L_{1}$ 范数: $\lVert \boldsymbol{x}\rVert_{1} = \sum_{i = 1}^n$
```

```ad-def
title: 矩阵范数
```

```ad-def
title: 距离
```

## 矩阵分解算法
### 特征值分解
通过矩阵变换, 将矩阵的主要信息转化到对角线上, 主成分分析
$$
\boldsymbol{A} = \boldsymbol{P}\Delta\boldsymbol{P}^{-1}
$$
算法: 幂方法
### LU 分解
- **L**: 下三角矩阵
- **U**: 上三角矩阵

便于求矩阵的逆, 从而计算
$$
\boldsymbol{A}\boldsymbol{x} = \boldsymbol{b}
$$
的解

算法
### SVD 分解
奇异值
$$
\boldsymbol{A}_{m\times n} = \boldsymbol{U}_{m\times m}\boldsymbol{\Sigma}_{m\times n}\boldsymbol{V}^{\mathrm{T}}_{n\times n}
$$
其中
$$
\boldsymbol{U}^{\mathrm{T}}\boldsymbol{U} = \boldsymbol{I}, \boldsymbol{V}^{\mathrm{T}}\boldsymbol{V} = \boldsymbol{I}
$$

### Moore-Penrose 伪逆
$$
\boldsymbol{A} = \boldsymbol{U}\boldsymbol{\Sigma}\boldsymbol{V}^{\mathrm{T}}
$$
$$
\boldsymbol{A}^+ = \boldsymbol{V}\boldsymbol{\Sigma}^{-1}\boldsymbol{U}^{\mathrm{T}}
$$
# 多元微积分
## 基础概念

# 概率论与数理统计

# 其他