+++
title = "机器学习实践（三）"
date = "2025-12-08"
draft = false
tags = ["机器学习", "特征工程"]
categories = ["学习笔记"]
+++

学习过程枯燥无趣，唯有实践才能让人在昏昏欲睡的状态中找到一丝乐趣。

根据前两篇的内容，处理一下数据。删除 PassengerId、Name、Ticket、Cabin 四列，剩余特征中，Pclass、Sex、Embarked 为类别型特征，Age、SibSp、Parch、Fare 为数值型特征。

然后进行数值处理：

- One-Hot 编码：Pclass、Sex、Embarked

```python
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import OneHotEncoder, StandardScaler, MinMaxScaler

# 不显示告警
import warnings
warnings.filterwarnings('ignore')

data = pd.read_csv('titanic.csv')
# 删除不必要的列
data = data.drop(columns=['PassengerId', 'Name', 'Ticket', 'Cabin'])
# Embarked 列缺失值填充为众数
mode_embarked = data['Embarked'].mode()[0]
print(f'Embarked 列众数为: {mode_embarked}\n')
data['Embarked'].fillna(mode_embarked, inplace=True)

# 编码
categorical_features = ['Pclass', 'Sex', 'Embarked']
encoder = OneHotEncoder(sparse=False)
encoded_categorical = encoder.fit_transform(data[categorical_features])
encoded_categorical_df = pd.DataFrame(encoded_categorical, columns=encoder.get_feature_names_out(categorical_features))
data = pd.concat([data.drop(columns=categorical_features), encoded_categorical_df], axis=1)
```

接下来使用随机森林补全 Age 和 Embarked 的缺失值：

```python
from sklearn.ensemble import RandomForestRegressor, RandomForestClassifier
# 补全 Age 缺失值
age_data = data[['Age', 'SibSp', 'Parch', 'Fare'] + list(encoded_categorical_df.columns)]
age_train = age_data[age_data['Age'].notnull()]
age_test = age_data[age_data['Age'].isnull()]
rf_regressor = RandomForestRegressor(n_estimators=100, random_state=42)
rf_regressor.fit(age_train.drop(columns=['Age']), age_train['Age'])
predicted_ages = rf_regressor.predict(age_test.drop(columns=['Age']))
print(f'补全 Age 缺失值，共补全 {len(predicted_ages)} 个缺失值。\n')
data.loc[data['Age'].isnull(), 'Age'] = predicted_ages
```

将特征进行标准化处理：

```python
# 标准化数值特征
numerical_features = ['Age', 'SibSp', 'Parch', 'Fare']
scaler = StandardScaler()
data[numerical_features] = scaler.fit_transform(data[numerical_features])
```

那么如何判断特征需要标准化还是归一化呢？

一般来说：

- 标准化（Standardization）：适用于大多数机器学习算法，尤其是那些基于距离的算法（如 KNN、SVM）和线性模型（如线性回归、逻辑回归）。标准化将数据转换为均值为 0、标准差为 1 的分布，适合处理具有不同尺度的特征。
- 归一化（Normalization）：适用于需要将数据缩放到特定范围（通常是 [0, 1]）的算法，如神经网络和深度学习模型。归一化有助于加快收敛速度，尤其是在使用梯度下降优化时。

# 逻辑回归模型训练与评估

Base Line 模型是指在没有进行任何复杂处理或优化的情况下，使用最简单的方法建立的模型。它通常作为评估其他更复杂模型性能的基准。

在这里，我们使用逻辑回归作为 Base Line 模型。逻辑回归有以下几个超参数：

- solver：优化算法，常用的有 'liblinear'（适用于小数据集）、'lbfgs'（适用于大数据集）等。
- penalty：正则化类型，常用的有 'l2'（岭回归）和 'l1'（Lasso 回归）。
- C：正则化强度的倒数，值越小表示正则化强度越大，也就越难拟合，默认值为 1.0。
- max_iter：最大迭代次数，默认值为 100。
- tol：优化算法的容忍度，默认值为 1e-4。优化算法在迭代过程中，如果损失函数的变化小于 tol，则停止迭代。

```python
X_train, X_test, y_train, y_test = train_test_split(data.drop(columns=['Survived']), data['Survived'], test_size=0.2, random_state=42)

from sklearn.linear_model import LogisticRegression
model = LogisticRegression(solver='liblinear', penalty='l2', C=1.0, max_iter=100)
model.fit(X_train, y_train)
train_accuracy = model.score(X_train, y_train)
test_accuracy = model.score(X_test, y_test)
print(f'训练集准确率: {train_accuracy:.4f}')
print(f'测试集准确率: {test_accuracy:.4f}')
```

## 生成结果

使用模型处理测试集，生成提交结果：

```python
test_data = pd.read_csv('test.csv')

# 进行与训练集相同的数据处理
test_data = test_data.drop(columns=['PassengerId', 'Name', 'Ticket', 'Cabin'])
test_data['Embarked'].fillna(mode_embarked, inplace=True)
encoded_test_categorical = encoder.transform(test_data[categorical_features])
encoded_test_categorical_df = pd.DataFrame(encoded_test_categorical, columns=encoder.get_feature_names_out(categorical_features))
test_data = pd.concat([test_data.drop(columns=categorical_features), encoded_test_categorical_df], axis=1)

# 补全 Age 缺失值
age_test_data = test_data[['Age', 'SibSp', 'Parch', 'Fare'] + list(encoded_test_categorical_df.columns)]
age_test_missing = age_test_data[age_test_data['Age'].isnull()]
predicted_test_ages = rf_regressor.predict(age_test_missing.drop(columns=['Age']))
test_data.loc[test_data['Age'].isnull(), 'Age'] = predicted_test_ages

# 补全 Fare 缺失值
test_data['Fare'].fillna(data['Fare'].mean(), inplace=True)

# 标准化数值特征
test_data[numerical_features] = scaler.transform(test_data[numerical_features])

# 预测
predictions = model.predict(test_data)
submission = pd.DataFrame({'PassengerId': pd.read_csv('test.csv')['PassengerId'], 'Survived': predictions})
submission.to_csv('submission.csv', index=False)
print("提交文件 submission.csv 已生成。")
```

# coef

在使用线性模型（如线性回归或逻辑回归）时，`coef_` 属性表示模型的系数（权重）。这些系数反映了每个特征对预测结果的影响程度。通过 coef_ 属性，我们可以了解哪些特征对模型的预测结果贡献较大，可以帮助我们进行特征选择和解释模型行为。

系数的正负表示特征与目标变量之间的关系方向，系数的绝对值表示特征的重要性。

```python
coefficients = model.coef_

# 输出每个特征的系数的表格
feature_names = X_train.columns
coef_df = pd.DataFrame({'Feature': feature_names, 'Coefficient': coefficients[0], 'Absolute Value': abs(coefficients[0])})
print("特征系数：")
print(coef_df)
```

这里可以看到，SibSp（兄弟姐妹/配偶数量）和 Parch（父母/子女数量）的系数较小，说明它们对生存预测的影响较小；尝试删除这些特征，会发现模型的准确率下降，说明这些特征虽然系数小，但仍然有一定的贡献。

# 学习曲线

学习曲线（Learning Curve）是用来评估机器学习模型性能随训练样本数量变化的图形表示。它通常显示训练集和验证集的误差或准确率随训练样本数量增加而变化的趋势。

学习曲线的横轴表示训练样本的数量，纵轴表示模型的性能指标（如误差率或准确率）。通过观察学习曲线，可以帮助我们理解模型的学习能力、是否过拟合或欠拟合，以及是否需要更多的数据来提升模型性能。

如果训练误差和验证误差都很高，说明模型欠拟合，可能需要更复杂的模型或更多的特征。如果训练误差很低但验证误差很高，说明模型过拟合，可能需要正则化或更多的数据。

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.model_selection import learning_curve

train_sizes, train_scores, test_scores = learning_curve(
    model, X_train, y_train, cv=5, scoring='准确率',
    train_sizes=np.linspace(0.1, 1.0, 10), n_jobs=-1
)

train_scores_mean = np.mean(train_scores, axis=1)
test_scores_mean = np.mean(test_scores, axis=1)

plt.figure()
plt.plot(train_sizes, train_scores_mean, 'o-', color='r', label='训练准确率')
plt.plot(train_sizes, test_scores_mean, 'o-', color='g', label='验证准确率')
plt.title('学习曲线')
plt.xlabel('训练样本数量')
plt.ylabel('准确率')
plt.legend(loc='best')
plt.grid()
plt.show()
```

# 总结

对于基础模型的调用，到这里基本讲完，后续介绍更复杂的特征处理方法和更复杂的模型。
