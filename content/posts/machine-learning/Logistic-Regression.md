+++
date = 2025-11-25
title = "逻辑回归(Logistic Regression)"
categories = ["学习笔记"]
tags = ["机器学习", "统计学", "分类算法"]
showHero = true
file = "Logistic-Regression.md"
+++

{{< katex >}}

逻辑回归是一种用于二分类问题的统计模型。它通过将输入特征的线性组合映射到0到1之间的概率值来进行分类。

# Sigmoid Function ( \(S\) 曲线函数 )

逻辑回归模型使用 Sigmoid 函数将线性组合的输入映射到0到1之间的概率值。Sigmoid 函数定义如下：

$$
\sigma(z) = \frac{1}{1 + e^{-z}} = \frac{e^{z}}{1 + e^{z}} = 1 - \frac{1}{1 + e^{z}} = 1 - \sigma(-z)
$$

函数的输出范围在0到1之间，适用于二分类问题，图像为 S 形曲线。

![Sigmoid 函数图像](/img/Logistic-Regression-0.png)

Sigmoid 函数的导数为：

$$
\sigma'(z) = \sigma(z)(1 - \sigma(z))
$$

图像为钟形曲线，表示在 z=0 处导数最大。

# 二维平面中的逻辑回归

在二维平面上进行二元分类时，逻辑回归模型找到的决策边界是一条直线，将数据点分为两类。

预测概率 \(h_\theta(x)\) 由以下公式给出（我们已知公式，就不从头推导了）：

$$
h_\theta(x) = \sigma(\theta^T x) = \frac{1}{1 + e^{-\theta^T x}}
$$

边界线表示当预测概率为 0.5 时的点集，即：

$$
h_\theta(x) = 0.5 \implies e^{-\theta^T x} = 1 \implies \theta^T x = 0
$$

其中，\(\theta\) 是模型参数向量，\(x\) 是输入特征向量。

矩阵形式下，转换为：

$$
\begin{bmatrix}
\theta_0 & \theta_1 & \theta_2
\end{bmatrix}
\begin{bmatrix}
1 \\ x_1 \\ x_2
\end{bmatrix} = 0
$$

修改为多项式形式，决策边界由以下方程定义：

$$
\theta_0 + \theta_1 x_1 + \theta_2 x_2 = 0
$$

其中，\(x_1\) 和 \(x_2\) 是输入特征，\(\theta_0\)、\(\theta_1\) 和 \(\theta_2\) 是模型参数。

该方程可以重写为（ \(x,y\) ）坐标形式：

$$
x_2 = -\frac{\theta_0}{\theta_2} - \frac{\theta_1}{\theta_2} x_1
$$

从几何角度来看，这条直线的斜率为 \(-\frac{\theta_1}{\theta_2}\)，截距为 \(-\frac{\theta_0}{\theta_2}\)。逻辑回归模型通过调整参数 \(\theta\) 来找到最佳的决策边界，以最大化分类的准确性。

![二维平面下的逻辑回归](/img/Logistic-Regression-1.png)

决策边界将二维平面分为两个区域：

- 当 \(\theta^T x \geq 0\) 时，预测类别为1；
- 当 \(\theta^T x < 0\) 时，预测类别为0。

模型参数 \(\theta\) 可以通过最小化损失函数或者最大化对数似然函数来估计。

# 损失函数和梯度

逻辑回归模型通常使用对数损失函数（Log Loss）来衡量模型预测与实际标签之间的差异。对于单个样本，损失函数定义为：

$$
L(h_\theta(x), y) = -[y \log(h_\theta(x)) + (1 - y) \log(1 - h_\theta(x))]
$$

其中，\(y\) 是实际标签，\(h_\theta(x)\) 是模型预测的概率，即 \(h_\theta(x) = \sigma(\theta^T x)\)。

该函数的梯度为：

$$
\nabla_\theta L(h_\theta(x), y) = (h_\theta(x) - y) x
$$

对于整个训练集，损失函数为：

$$
J(\theta) = -\frac{1}{m} \sum_{i=1}^{m} [y^{(i)} \log(h_\theta(x^{(i)})) + (1 - y^{(i)}) \log(1 - h_\theta(x^{(i)}))]
$$

其梯度为：

$$
\nabla_\theta J(\theta) = \frac{1}{m} \sum_{i=1}^{m} (h_\theta(x^{(i)}) - y^{(i)}) x^{(i)}
$$

求解该梯度可以使用梯度下降法或其他优化算法来更新模型参数 \(\theta\)，以最小化损失函数 \(J(\theta)\)。

## 对数似然函数

### 伯努利分布（Bernoulli Distribution）

逻辑回归模型假设输出变量 \(Y\) 服从伯努利分布。伯努利分布是一种离散概率分布，表示单次试验中只有两种可能结果的情况，通常记为成功（1）和失败（0）。其概率质量函数（PMF）定义如下：

$$
P(Y = y) = p^y (1 - p)^{1 - y}
$$

其中，\(y \in \{0, 1\}\)，\(p\) 是成功的概率，\(0 \leq p \leq 1\)。

举例来说，如果我们有一个硬币，掷出正面（成功）的概率为 \(p\)，那么掷出反面（失败）的概率就是 \(1 - p\)。因此，伯努利分布非常适合描述只有两种可能结果的随机事件。二分类问题中，概率分布不一定是均匀分布，比如，某个类别的样本可能远多于另一个类别。

### 似然函数（Likelihood Function）

似然函数是一个用于估计模型参数的函数，表示在给定参数下，观察到的数据出现的概率。对于逻辑回归模型，假设我们有 \(m\) 个独立同分布的样本 \((x^{(i)}, y^{(i)})\)，其中 \(y^{(i)} \in \{0, 1\}\)。则似然函数定义为：

$$
L(\theta) = \prod_{i=1}^{m} P(Y = y^{(i)} | x^{(i)}; \theta)
$$

公式中，\(P(Y = y^{(i)} | x^{(i)}; \theta)\) 是在给定参数 \(\theta\) 和输入特征 \(x^{(i)}\) 下，输出变量 \(Y\) 取值为 \(y^{(i)}\) 的概率。根据伯努利分布的定义，我们可以将其展开为：

$$
L(\theta) = \prod_{i=1}^{m} [h_\theta(x^{(i)})]^{y^{(i)}} [1 - h_\theta(x^{(i)})]^{1 - y^{(i)}}
$$

其中，\(h_\theta(x^{(i)}) = \sigma(\theta^T x^{(i)})\) 是逻辑回归模型的假设函数，表示在给定参数 \(\theta\) 和输入特征 \(x^{(i)}\) 下，输出变量 \(Y\) 取值为1的概率。

为了简化计算，通常使用对数似然函数（Log-Likelihood Function），也就是对似然函数取自然对数：

$$
\ell(\theta) = \log L(\theta) = \sum_{i=1}^{m} [y^{(i)} \log(h_\theta(x^{(i)})) + (1 - y^{(i)}) \log(1 - h_\theta(x^{(i)}))]
$$

这样，连乘操作被转换为加法操作，便于计算和优化。

此函数为什么能够描述模型的性能：

- 分类问题中，目标是最大化正确分类的概率。对数似然函数衡量了模型在给定参数 \(\theta\) 下对训练数据的拟合程度。在此函数中：
  - 当 \(y^{(i)} = 1\) 时，贡献项为 \(\log(h_\theta(x^{(i)}))\)，如果模型预测概率接近1，该项值较大。
  - 当 \(y^{(i)} = 0\) 时，贡献项为 \(\log(1 - h_\theta(x^{(i)}))\)，如果模型预测概率接近0，该项值较大。
- 逻辑回归是一个二分类模型，结果越接近实际标签，贡献值越大。因此，最大化对数似然函数等价于提高模型的分类准确性。

# 简单实例

假设我们有一个简单的数据集，用于预测某人是否会购买某产品，基于他们的年龄和收入。数据集如下：

| 年龄 (Age) | 收入 (Income) | 购买 (Purchased) |
|------------|----------------|------------------|
| 22         | 15000          | 0                |
| 25         | 29000          | 0                |
| 47         | 48000          | 1                |
| 52         | 60000          | 1                |

我们可以使用逻辑回归模型来预测购买概率。假设模型参数为：

$$
\theta = \begin{bmatrix} -4 \\ 0.05 \\ 0.0001 \end{bmatrix}
$$

对于一个新样本，年龄为30岁，收入为40000，我们可以计算其购买概率：

$$
x = \begin{bmatrix} 1 \\ 30 \\ 40000 \end{bmatrix}
$$

计算线性组合：

$$
z = \theta^T x = -4 + 0.05 \times 30 + 0.0001 \times 40000 = -4 + 1.5 + 4 = 1.5
$$

应用 Sigmoid 函数：

$$
h_\theta(x) = \sigma(1.5) = \frac{1}{1 + e^{-1.5}} \approx 0.8176
$$

因此，该模型预测该用户购买该产品的概率约为81.76%。

# 参考资料

- [Logistic Regression - Wikipedia](https://en.wikipedia.org/wiki/Logistic_regression)
- [Scikit-learn Logistic Regression Documentation](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html)
- [Understanding the Sigmoid Function](https://towardsdatascience.com/understanding-the-sigmoid-function-9b3f7f4d3f2d)
- [Machine Learning Mastery: Logistic Regression](https://machinelearningmastery.com/logistic-regression-for-machine-learning/)
- [Coursera: Machine Learning by Andrew Ng](https://www.coursera.org/learn/machine-learning)
