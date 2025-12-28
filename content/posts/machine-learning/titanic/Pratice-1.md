+++
title = "机器学习实践（一）"
date = "2025-12-04"
draft = false
tags = ["机器学习", "特征工程"]
categories = ["学习笔记"]
+++

{{< katex >}}

在学习了机器学习的基本算法以及数据预处理和特征工程的相关知识后，我们通过经典的机器学习案例，对现实数据进行一些处理和分析，让我们机器学习工程有一个基本的认识。

# 泰坦尼克生存数据集描述

数据集包含以下主要特征：

- **PassengerId**: 乘客的唯一标识符
- **Survived**: 是否幸存（0 = 否，1 = 是）
- **Pclass**: 船舱等级（1 = 头等舱，2 = 二等舱，3 = 三等舱）
- **Name**: 乘客姓名
- **Sex**: 性别
- **Age**: 年龄
- **SibSp**: 兄弟姐妹 / 配偶数量，Sibling/Spouse aboard the Titanic
- **Parch**: 父母 / 子女数量，Parent/Children aboard the Titanic
- **Ticket**: 船票号码
- **Fare**: 票价
- **Cabin**: 船舱号码
- **Embarked**: 登船港口（C = Cherbourg（瑟堡），Q = Queenstown（昆士敦），S = Southampton（南安普顿））

其中，目标变量是 **Survived**，表示乘客是否幸存。

PassengerId 是每个乘客的唯一标识符，不应作为特征用于模型训练。
Name 特征通常也不会用于模型训练，因为它们包含大量唯一值，且与幸存与否的关系不大。
Ticket 和 Cabin 特征也常常被忽略，前者因为其复杂性和多样性，后者因为缺失值较多。
其中 Sex 和 Embarked 是分类变量，需要进行适当的编码处理（如独热编码）以便用于机器学习模型。

剩余的数值特征如 Pclass、Age、SibSp、Parch 和 Fare 可以直接用于模型训练，但可能需要进行归一化或标准化处理。

# 成人收入数据集描述

数据集有 15 个特征，主要包括：

- **age**: 年龄
- **workclass**: 工作类别
- **fnlwgt**: 最终权重
- **education**: 教育水平
- **education-num**: 教育年限
- **marital-status**: 婚姻状况
- **occupation**: 职业
- **relationship**: 家庭关系
- **race**: 种族
- **gender**: 性别
- **capital-gain**: 理财收益
- **capital-loss**: 理财损失
- **hours-per-week**: 每周工作小时数
- **native-country**: 原籍国

目标变量是 **income**，表示收入是否超过 5 万美元（<=50K 或 >50K）。

# 数据查看

一般机器学习工作的第一步都是使用 Pandas 库加载并查看数据集，以下几个方法：

- head(): 查看前几行数据
- info(): 查看数据的基本信息，包括数据类型和缺失值情况
- describe(): 查看数值特征的统计信息，包括均值、标准差、最小值、最大值等

# 数据预处理

在处理数据之前，需要回答以下几个问题：

1. 数据是否存在异常值或缺失值？
2. 特征是否需要进行转换或编码？
3. 是否需要创建新的特征？
4. 特征分布有什么特点？是否需要标准化或归一化？

## 检测和处理异常值

使用箱线图（Box Plot）来检测异常值，箱线图可以帮助我们识别数据中的极端值。

以泰坦尼克数据集中的 Fare（票价）特征为例，下面的代码展示了如何绘制箱线图来检测异常值：

```python
import matplotlib.pyplot as plt
import seaborn as sns

data = pd.read_csv('titanic.csv')
plt.figure(figsize=(5, 3))
sns.boxplot(x=data['Fare'])
plt.show()
```

箱形图的上限和下限可以通过以下公式计算：

- 上限 = Q3 + 1.5 * IQR
- 下限 = Q1 - 1.5 * IQR

其中，Q1 和 Q3 分别是第一四分位数和第三四分位数，IQR 是四分位距（IQR = Q3 - Q1）。

Z-score 方法可以用于检测异常值，计算公式如下：

$$
z = \frac{(X - \mu)}{\sigma}
$$

其中，\(X\) 是数据点，\(\mu\) 是均值，\(\sigma\) 是标准差。通常，Z-score 大于 3 或小于 -3 的数据点被视为异常值。具体说就是数据点距离均值超过三个标准差。

计算 Z-score 并检测异常值的示例代码：

```python
from scipy import stats
import numpy as np

data['Fare_zscore'] = np.abs(stats.zscore(data['Fare']))
outliers = data[data['Fare_zscore'] > 3]
print("异常值数量：", len(outliers))
```

通过 IRQ 计算并检测异常值的示例代码：

```python
Q1 = data['Fare'].quantile(0.25)
Q3 = data['Fare'].quantile(0.75)
IQR = Q3 - Q1
lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR
outliers_iqr = data[(data['Fare'] < lower_bound) | (data['Fare'] > upper_bound)]
print("异常值数量（IQR 方法）：", len(outliers_iqr))
```

通过以上方法，可以看出 Fare 特征中存在一些异常值。票价中位数是 14.4542，均值是 32.2042，标准差是 49.6934，说明有些票价远大于合理的区间。还有一些乘客的票价为 0，这也是异常值的一种表现。

## 处理缺失值

处理缺失值的方法包括删除缺失值、填充缺失值（如使用均值、中位数、众数或插值法）等。

从 info 方法可以看出，年龄、船舱和登船港口存在缺失值。可以使用以下方法处理缺失值：

- 对于年龄（Age）特征，可以使用中位数进行填充，因为年龄的分布可能存在偏态。
- 对于船舱（Cabin）特征，可以选择删除该特征，因为缺失值较多且对模型影响不大。
- 对于登船港口（Embarked）特征，可以使用众数进行填充。

## 特征编码

对于分类变量（如性别和登船港口），可以使用独热编码（One-Hot Encoding）或标签编码（Label Encoding）进行转换。

独热编码的意思是将分类变量转换为多个二进制特征：

- 比如 Sex 特征有两个类别： male 和 female，可以创建两个新特征：
    - Sex_male：如果性别为 male，则值为 1，否则为 0
    - Sex_female：如果性别为 female，则值为 1，否则为 0
- 如果是 Embarked 特征有三个类别： C、Q 和 S，可以创建三个新特征：
    - Embarked_C
    - Embarked_Q
    - Embarked_S

标签编码是将每个类别映射为一个整数值。例如：

- Sex 特征：
    - male -> 0
    - female -> 1
- Embarked 特征：
    - C -> 0
    - Q -> 1
    - S -> 2

## 新增特征

可以根据现有特征创建新的特征，以增强模型的表达能力。在本案例中，可以创建以下新特征：

- 家庭规模（FamilySize）：通过将 SibSp 和 Parch 相加再加 1（表示乘客本人）来计算家庭规模。

这个新特征可以帮助模型更好地理解家庭对生存率的影响。

## 特征值分布

特征的正态和偏态分布会影响模型的表现。可以使用直方图（Histogram）、密度图（Density Plot）或 Q-Q 图（Quantile-Quantile Plot）来查看特征的分布情况。

密度图可以理解为是直方图的平滑版本，更加直观地展示数据的分布情况。将两个特征的密度图进行对比，可以观察它们的分布差异。

Q-Q 图用于比较数据分布与正态分布的差异，如果数据点大致沿对角线分布，则说明数据接近正态分布。

也可以使用偏度（Skewness）和峰度（Kurtosis）来量化分布的形态：

- 偏度衡量分布的对称性，偏度为 0 表示完全对称，正偏度表示右尾较长，负偏度表示左尾较长。
- 峰度衡量分布的尖锐程度，峰度大于 3 表示分布较尖锐，峰度小于 3 表示分布较平坦。

### 特征标准化和归一化

对于数值特征（如年龄和票价），可以使用标准化（Standardization）或归一化（Normalization）进行处理。

标准化是将特征转换为均值为 0，标准差为 1 的分布，计算公式如下：

$$
X_{std} = \frac{(X - \mu)}{\sigma}
$$

标准化可以帮助模型更快收敛，尤其是对于基于梯度下降的算法。

归一化是将特征缩放到一个固定范围（通常是 0 到 1），计算公式如下：

$$
X_{norm} = \frac{(X - X_{min})}{(X_{max} - X_{min})}
$$

归一化适用于需要度量距离的算法，如 KNN 和 K-means。

选择标准化还是归一化取决于具体的模型和数据分布情况，在本案例中，可以对 Age 和 Fare 特征进行标准化处理。

# 总结

通过对单个特征的详细分析和处理，回答一些关键问题，更好的理解数据；并根据一些人类常识，对数据进行合理的预处理，可以提高后续学习模型的效果。
