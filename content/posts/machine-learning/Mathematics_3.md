+++
title = "矩阵运算"
file = "Mathematics_3.md"
date = "2025-12-02"
tags = ["机器学习", "数学基础"]
categories = ["学习笔记"]
+++

{{< katex >}}

最近学数学有一点点模糊的认识：很多计算方法的定义是为了方便我们进行基本运算的组合和推广。比如不同对象的加减乘除，很多运算都遵循交换律、结合律、分配律等基本规则。比如矩阵、向量、复数的运算，都是基本运算的推广和组合。

有一些运算存在单位元和逆元的概念。单位元是指在某种运算下不改变其他元素的特殊元素，比如加法的单位元是0，乘法的单位元是1。逆元是指在某种运算下与某个元素结合后得到单位元的元素，比如对于加法来说，元素 \( a \) 的逆元是 \( -a \)，因为 \( a + (-a) = 0 \)；对于乘法来说，非零元素 \( a \) 的逆元是 \( \frac{1}{a} \)，因为 \( a \times \frac{1}{a} = 1 \)。对于矩阵来说，单位矩阵是乘法的单位元，而逆矩阵是乘法的逆元。

# 矩阵乘法的运算规则及其来源

关于矩阵的一些基本运算定义：

- 矩阵的转置（Transpose）：将矩阵的行和列进行交换，记作 \( \mathbf{X}^T \)。
- 矩阵的逆（Inverse）：对于一个方阵 \( \mathbf{A} \)，如果存在一个矩阵 \( \mathbf{B} \) 使得 \( \mathbf{A} \mathbf{B} = \mathbf{B} \mathbf{A} = \mathbf{I} \)（单位矩阵），则称 \( \mathbf{B} \) 为 \( \mathbf{A} \) 的逆矩阵，记作 \( \mathbf{A}^{-1} \)。
- 单位矩阵（Identity Matrix）：一个方阵，其对角线元素为1，其他元素为0，记作 \( \mathbf{I} \)。
- 矩阵乘法（Matrix Multiplication）：矩阵乘法的运算规则要求前一个矩阵的列数必须等于后一个矩阵的行数，结果矩阵的值通过对应元素的乘积求和得到。用公式表示为：
$$
(\mathbf{A} \mathbf{B})_{ij} = \sum_{k} a_{ik} b_{kj}
$$
其中 \( a_{ik} \) 是矩阵 \( \mathbf{A} \) 的第 \( i \) 行第 \( k \) 列元素，\( b_{kj} \) 是矩阵 \( \mathbf{B} \) 的第 \( k \) 行第 \( j \) 列元素。

那么为什么这样规定矩阵乘法的运算规则呢？

矩阵乘法的运算规则源于线性代数中的线性变换和向量空间的概念。矩阵可以看作是对向量进行线性变换的工具，而矩阵乘法则对应于连续应用多个线性变换。

向量空间是由向量组成的集合，向量可以通过线性组合进行操作，例如加法和数乘。向量加法：

$$
\mathbf{u} + \mathbf{v} = \begin{bmatrix} u_1 + v_1 \\ u_2 + v_2 \\ \vdots \\ u_n + v_n \end{bmatrix}
$$

数乘：

$$
c \mathbf{u} = \begin{bmatrix} c u_1 \\ c u_2 \\ \vdots \\ c u_n \end{bmatrix}
$$

线性变换是将一个向量映射到另一个向量的函数，满足以下两个性质：

1. 加法封闭性：对于任意向量 \( \mathbf{u} \) 和 \( \mathbf{v} \)，线性变换满足 \( T(\mathbf{u} + \mathbf{v}) = T(\mathbf{u}) + T(\mathbf{v}) \)。
2. 数乘封闭性：对于任意向量 \( \mathbf{u} \) 和标量 \( c \)，线性变换满足 \( T(c \mathbf{u}) = c T(\mathbf{u}) \)。

到这里我们理解了向量空间和线性变换的概念，举例一个简单的线性变换：

$$
T\begin{bmatrix} x_1 \\ x_2 \end{bmatrix} = \begin{bmatrix} 2x_1 + 3x_2 \\ 4x_1 + 5x_2 \end{bmatrix}
$$

这个变换是线性的，因为它满足加法封闭性和数乘封闭性：

$$
T\begin{bmatrix} x_1 + y_1 \\ x_2 + y_2 \end{bmatrix} = T\begin{bmatrix} x_1 \\ x_2 \end{bmatrix} + T\begin{bmatrix} y_1 \\ y_2 \end{bmatrix}
$$

$$
T\begin{bmatrix} c x_1 \\ c x_2 \end{bmatrix} = c T\begin{bmatrix} x_1 \\ x_2 \end{bmatrix}
$$

将这个线性变换表示为矩阵形式：

$$
\begin{bmatrix} 2 & 3 \\ 4 & 5 \\ 5 & 6 \end{bmatrix} \begin{bmatrix} x_1 \\ x_2 \end{bmatrix} = \begin{bmatrix} 2x_1 + 3x_2 \\ 4x_1 + 5x_2 \\ 5x_1 + 6x_2 \end{bmatrix}
$$

可以看出，矩阵乘法的定义确保了线性变换的性质得以保持。

这种计算推广到更高维度，矩阵其实是多个线性变换的组合，例如：

$$
\begin{bmatrix} a_{11} & a_{12} & a_{13} \\ a_{21} & a_{22} & a_{23} \end{bmatrix} \begin{bmatrix} x_{11} & x_{12} \\ x_{21} & x_{22} \\ x_{31} & x_{32} \end{bmatrix} = \begin{bmatrix} a_{11}x_{11} + a_{12}x_{21} + a_{13}x_{31} & a_{11}x_{12} + a_{12}x_{22} + a_{13}x_{32} \\ a_{21}x_{11} + a_{22}x_{21} + a_{23}x_{31} & a_{21}x_{12} + a_{22}x_{22} + a_{23}x_{32} \end{bmatrix}
$$

用语言描述就是：矩阵的每一行与另一个矩阵的每一列进行点积运算，得到结果矩阵的对应元素。行数和列数的匹配确保了每个线性变换都能正确应用到输入向量上，从而实现了线性变换的连续应用。

矩阵乘法不满足交换律（即 \( \mathbf{A} \mathbf{B} \neq \mathbf{B} \mathbf{A} \)），这是因为线性变换的顺序会影响最终结果。不同的线性变换顺序可能会导致不同的输出，因此矩阵乘法的非交换性反映了线性变换的本质特性。

# 点乘

向量点积（Dot Product）是线性代数中的一种基本运算，用于计算两个向量之间的相似度或投影。对于两个同维度的向量 \( \mathbf{a} \) 和 \( \mathbf{b} \)，它们的点积定义为：

$$
\mathbf{a} \cdot \mathbf{b} = \sum_{i=1}^{n} a_i b_i
$$

向量转置（Transpose）是将一个列向量转换为行向量，或将一个行向量转换为列向量的操作。对于一个向量 \( \mathbf{a} \)：

$$
\mathbf{a} = \begin{bmatrix} a_1 \\ a_2 \\ \vdots \\ a_n \end{bmatrix} \quad \Rightarrow \quad \mathbf{a}^T = \begin{bmatrix} a_1 & a_2 & \ldots & a_n \end{bmatrix}
$$

向量转置后，与原向量进行点积运算，可以得到一个标量值：

$$
\mathbf{a}^T \mathbf{a} = \sum_{i=1}^{n} a_i^2
$$

现在我们有两个向量 \( \mathbf{a} \) 和 \( \mathbf{b} \)，想实现 \( \mathbf{a} - \mathbf{b} \) 的平方和，可以通过以下方式表示：

$$
(\mathbf{a} - \mathbf{b})^T (\mathbf{a} - \mathbf{b}) = \sum_{i=1}^{n} (a_i - b_i)^2
$$

线性回归的损失函数通常表示为预测值与真实值之间的平方误差和：

$$
L(\boldsymbol{\omega}) = \sum_{i=1}^{m} (y_i - \hat{y}_i)^2
$$

转换为矩阵形式后，可以表示为：

$$
L(\boldsymbol{\omega}) = (\mathbf{y} - \mathbf{X} \boldsymbol{\omega})^T (\mathbf{y} - \mathbf{X} \boldsymbol{\omega})
$$

# 矩阵求导

我们有这样一个函数：

$$
f(\mathbf{x}) = \mathbf{x}^T \mathbf{A} \mathbf{x}
$$

其中 \( \mathbf{x} \) 是一个列向量，\( \mathbf{A} \) 是一个矩阵。我们想求这个函数对向量 \( \mathbf{x} \) 的导数。

首先，我们可以将函数展开：

$$
f(\mathbf{x}) = \sum_{i=1}^{n} \sum_{j=1}^{n} x_i a_{ij} x_j
$$

接下来，我们对 \( f(\mathbf{x}) \) 关于 \( x_k \) 求偏导数：

$$
\frac{\partial f}{\partial x_k} = \sum_{j=1}^{n} a_{kj} x_j + \sum_{i=1}^{n} x_i a_{ik}
$$

将所有偏导数组合成一个向量，我们得到：

$$
\nabla_{\mathbf{x}} f(\mathbf{x}) = \mathbf{A} \mathbf{x} + \mathbf{A}^T \mathbf{x}
$$

如果矩阵 \( \mathbf{A} \) 是对称矩阵（即 \( \mathbf{A} = \mathbf{A}^T \)），那么导数可以简化为：

$$
\nabla_{\mathbf{x}} f(\mathbf{x}) = 2 \mathbf{A} \mathbf{x}
$$

这是因为对称矩阵的性质使得两个项相等，从而合并为一个项的两倍。

这里其实就可以引出Jacobi矩阵的概念。Jacobi矩阵是一个由多变量函数的偏导数组成的矩阵，广泛应用于数值分析和优化问题中。对于一个向量值函数 \( \mathbf{f} : \mathbb{R}^n \to \mathbb{R}^m \)，其 Jacobi 矩阵定义为：

\[
J(\mathbf{f}) = \begin{bmatrix}
\frac{\partial f_1}{\partial x_1} & \frac{\partial f_1}{\partial x_2} & \cdots & \frac{\partial f_1}{\partial x_n} \\
\frac{\partial f_2}{\partial x_1} & \frac{\partial f_2}{\partial x_2} & \cdots & \frac{\partial f_2}{\partial x_n} \\
\vdots & \vdots & \ddots & \vdots \\
\frac{\partial f_m}{\partial x_1} & \frac{\partial f_m}{\partial x_2} & \cdots & \frac{\partial f_m}{\partial x_n}
\end{bmatrix}
\]

后续有很多地方也会用到雅可比矩阵的概念。

# 总结

矩阵乘法的运算规则源于线性代数中的线性变换和向量空间的概念。矩阵乘法对应于连续应用多个线性变换，确保了线性变换的性质得以保持。
