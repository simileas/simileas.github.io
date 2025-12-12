+++
date = '2025-11-22'
title = '线性回归'
categories = ['学习笔记']
tags = ['线性回归', '机器学习', '监督学习']
showHero = true
+++

{{< katex >}}

线性回归（Linear Regression）是一种用于预测连续数值的监督学习算法。它假设自变量（输入特征）与因变量（输出目标）之间存在线性关系。线性回归的目标是找到一条最佳拟合直线，使得预测值与实际值之间的误差最小。

线性回归的基本形式可以表示为：

$$
y = \omega_0 + \omega_1 x_1 + \omega_2 x_2 + \ldots + \omega_n x_n
$$

其中，\( y \) 是因变量，\( x_1, x_2, \ldots, x_n \) 是自变量，每个变量代表一个特征。\( \omega_0 \) 是截距，\( \omega_1, \omega_2, \ldots, \omega_n \) 是各自变量的系数。

线性回归的参数估计通常使用最小二乘法（Ordinary Least Squares, OLS），其目标是最小化预测值与实际值之间的平方误差和。损失函数可以表示为：

$$
J(\omega) = \frac{1}{2m} \sum_{i=1}^{m} (h_\omega(x^{(i)}) - y^{(i)})^2
$$

其中，\( m \) 是样本数量，\( h_\omega(x^{(i)}) \) 是模型的预测值，\( y^{(i)} \) 是实际值。\( x^{(i)} \) 表示第 \( i \) 个样本的特征向量。\( \omega \) 是参数向量，包含所有的系数和截距。

转换为矩阵形式后，损失函数可以表示为：

$$
J(\omega) = \frac{1}{2m} (X\omega - y)^T (X\omega - y)
$$

那么这个函数如何求导呢？矩阵求导的规则与标量求导类似，都遵循微积分基本原理，这里可以看作是乘法法则和链式法则的推广。

对矩阵的转置和乘法进行求导时，我们可以使用以下规则：

$$
\frac{\partial}{\partial \omega} (X\omega) = X^T
$$

$$
\frac{\partial}{\partial \omega} (X\omega - y)^T (X\omega - y) = 2X^T (X\omega - y)
$$

结合这些规则，我们可以对损失函数进行求导：

$$
\frac{\partial J(\omega)}{\partial \omega} = \frac{1}{m} X^T (X\omega - y)
$$

设置导数为零以找到最小值：

$$
X^T (X\omega - y) = 0
$$

解这个方程组可以得到线性回归模型的参数估计公式：

$$
\boldsymbol{\omega} = (\mathbf{X}^T \mathbf{X})^{-1} \mathbf{X}^T \mathbf{y}
$$

## R方（R-squared）

R方（R-squared）是评估线性回归模型性能的常用指标，表示模型解释的方差比例。R方的计算公式为：

$$
R^2 = 1 - \frac{SS_{res}}{SS_{tot}}
$$

这个公式也写成：

$$
R^2 = 1 - \frac{SSE}{SST}
$$

其中，$SS_{res}$ 是残差平方和，$SS_{tot}$ 是总平方和。R方的取值范围为0到1，值越接近1表示模型拟合效果越好。残差平方和（Residual Sum of Squares, RSS）表示预测值与实际值之间的差异，计算公式为：

$$
SS_{res} = \sum_{i=1}^{m} (y^{(i)} - h_\omega(x^{(i)}))^2
$$

总平方和（Total Sum of Squares, TSS）表示实际值与均值之间的差异，计算公式为：

残差平方和的公式与损失函数的公式部分一致，但在R方的计算中，残差平方和用于衡量模型的拟合效果，而损失函数用于参数优化过程中的误差最小化。

$$
SS_{tot} = \sum_{i=1}^{m} (y^{(i)} - \bar{y})^2
$$

其中，\(\bar{y}\) 是因变量的均值，衡量实际值的总变异性。
