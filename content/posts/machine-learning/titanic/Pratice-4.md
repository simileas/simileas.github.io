+++
title = "机器学习实践（四）"
date = "2025-12-10"
draft = true
tags = ["机器学习", "特征工程"]
categories = ["学习笔记"]
+++

进阶的机器学习实践离不开对数据的深入理解和特征工程的有效应用。下面介绍一些常用的特征工程技术和实践方法，帮助提升模型的性能。

**1. 更深入的特征工程**

将连续变量进行分箱处理，或者对类别变量进行独热编码（One-Hot Encoding）等方法，可以帮助模型更好地理解数据的分布和关系。

识别文本特征中的规律，如关键词提取、情感分析等，可以为模型提供更多有价值的信息。

**2. Bad Case 研究**

通过分析模型在某些样本上的错误预测，找出模型的不足之处。可以通过可视化工具或统计分析来识别这些“坏案例”，并针对性地改进特征或模型。

**3. 集成学习**

集成学习通过结合多个模型的预测结果，通常能提升整体的预测性能。常见的集成方法包括 Bagging、Boosting 和 Stacking。

# 深入的理解特征

对数据有八卦之心还是很重要的，很多 insight 都藏在数据里。

数据集中的人名、船舱号等信息，可能包含有用的线索。比如，某些姓氏可能与社会地位相关，从而影响生存率。

将名字中的 Title （如 Mr., Mrs., Miss 等）提取出来，作为新的特征，可以帮助模型更好地理解乘客的社会角色和生存概率：

```python
import pandas as pd
data = pd.read_csv('titanic.csv')

# 将 Title 提取出来
data['Title'] = data['Name'].str.extract(' ([A-Za-z]+)\.', expand=False)
data['Title'].value_counts()
```

将 Title 进行分组，合并一些称谓：

```python
data['Title'] = data['Title'].replace(['Lady', 'Countess', 'Capt', 'Col', 'Don', 'Dr', 'Major', 'Rev', 'Sir', 'Jonkheer', 'Dona', 'Mme', 'Mlle', 'Ms'], 'Rare')
data['Title'] = data['Title'].map({"Mr": 1, "Miss": 2, "Mrs": 3, "Master": 4, "Rare": 5})
data['Title'] = data['Title'].fillna(-100)
```

# Bad Case 研究

有女性死亡的家庭和有男性生还的家庭，都是 Bad Case 的典型代表。 通过分析这些 Bad Case，可以发现模型在某些特征上的不足，从而进行针对性的改进。

标注出 Bad Case：

```python
data['FamilySize'] = data['SibSp'] + data['Parch'] + 1
data['IsAlone'] = 1  # 初始化为独自一人
data['IsAlone'].loc[data['FamilySize'] > 1] = 0

# 标注 Bad Case
data['BadCase'] = 0
data.loc[(data['Sex'] == 'female') & (data['Survived'] == 0), 'BadCase'] = 1
data.loc[(data['Sex'] == 'male') & (data['Survived'] == 1), 'BadCase'] = 1
```

# 集成学习

集成学习通过结合多个模型的预测结果，通常能提升整体的预测性能。下面是一个使用随机森林和梯度提升树进行集成学习的示例：

```python
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier, VotingClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

X = data.drop(columns=['Survived', 'BadCase'])
y = data['Survived']

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

rf_clf = RandomForestClassifier(n_estimators=100, random_state=42)
gb_clf = GradientBoostingClassifier(n_estimators=100, random_state=42)
voting_clf = VotingClassifier(estimators=[('rf', rf_clf), ('gb', gb_clf)], voting='soft')
voting_clf.fit(X_train, y_train)
y_pred = voting_clf.predict(X_test)
accuracy = accuracy_score(y_test, y_pred)
print(f'集成模型准确率: {accuracy:.4f}')
```
