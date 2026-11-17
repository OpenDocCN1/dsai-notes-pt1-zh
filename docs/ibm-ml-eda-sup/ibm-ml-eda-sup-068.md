# 068：交叉验证演示第一部分 📊





在本节课中，我们将学习如何将多个数据处理步骤链接在一起，使用Scikit-learn的`Pipeline`功能来加速机器学习工作流。我们将讨论如何使用`KFold`对象将数据分割成多个“折”，并学习如何使用`cross_val_predict`和`GridSearchCV`来评估模型在每一折上的表现。





---




## 概述与库导入



首先，我们导入本教程所需的库。除了常用的`pandas`和`numpy`，我们还需要从Scikit-learn中导入一些特定模块。


```python
import pandas as pd
import numpy as np
from sklearn.model_selection import KFold, cross_val_predict
from sklearn.linear_model import LinearRegression, Lasso, Ridge
from sklearn.metrics import r2_score
from sklearn.pipeline import Pipeline
```



从`model_selection`模块，我们导入了`KFold`和`cross_val_predict`。从`linear_model`模块，我们不仅导入了`LinearRegression`，还引入了`Lasso`和`Ridge`回归。这些是带有防止过拟合机制的线性回归变体。从`metrics`模块导入了`r2_score`（即R平方分数），并从`pipeline`模块导入了`Pipeline`函数。



---




## 加载数据集




本次使用的数据集存储在一个`pickle`文件中。`pickle`允许我们保存和轻松检索Python对象。








```python
with open(‘boston_housing.pkl‘, ‘rb‘) as f:
    boston_dict = pickle.load(f)
boston_data = boston_dict[‘dataframe‘]
boston_description = boston_dict[‘description‘]
```




打开`pickle`文件后，我们发现它是一个字典，包含`‘dataframe‘`和`‘description‘`两个键。我们将`pandas`数据框提取为`boston_data`，将描述信息提取为`boston_description`。


查看数据框的前五行，可以看到我们的目标是预测`‘median value‘`（房价中位数），并使用其他所有特征来帮助预测。




---


## 数据准备与KFold分割



我们的目标是如何仅使用当前数据集来预测未来的值。为此，我们将使用`KFold`将数据分割成三个不同的训练集和测试集。





以下是实现步骤的代码：





首先，分离特征`X`和目标变量`y`。




```python
X = boston_data.drop(‘median value‘, axis=1)
y = boston_data[‘median value‘]
```



接着，初始化`KFold`对象。我们需要传入几个参数：
*   `shuffle=True`：在分割前打乱数据顺序，避免按原始顺序分割。
*   `random_state`：设置随机种子以确保结果可复现。
*   `n_splits=3`：指定将数据分割成3折。





```python
kf = KFold(shuffle=True, random_state=42, n_splits=3)
```



`kf.split(X)`会生成一个生成器对象，可以将其视为一个列表。列表中的每个元素都是一个元组`(train_index, test_index)`，分别包含了训练集和测试集的索引。


让我们查看一下第一个分割的索引情况：





```python
for train_index, test_index in kf.split(X):
    print(‘First 10 train indices:‘, train_index[:10])
    print(‘First 10 test indices:‘, test_index[:10])
    print(‘Train length:‘, len(train_index), ‘Test length:‘, len(test_index))
    print(‘---‘)
```




运行代码后，我们会看到测试集索引的长度大约是总数据集长度的三分之一（例如170），而训练集索引的长度则是剩余的三分之二。训练集的索引在不同折之间可以重叠，但测试集的索引是互斥的，确保了每次评估的都是不同的数据子集。



---





## 手动循环实现交叉验证




现在，我们想看看模型在每一折训练和测试集上的得分。我们将手动循环遍历每一折，进行训练和评估。




以下是实现过程：




首先，创建一个空列表来存储每次的得分，并初始化线性回归模型。


```python
scores = []
lr = LinearRegression()
```





然后，使用`for`循环遍历`kf.split(X)`生成的每一组`(train_index, test_index)`。





```python
for train_index, test_index in kf.split(X):
    # 根据索引分割数据
    X_train = X.iloc[train_index]
    X_test = X.iloc[test_index]
    y_train = y.iloc[train_index]
    y_test = y.iloc[test_index]

    # 在训练集上拟合模型
    lr.fit(X_train, y_train)

    # 在测试集上进行预测
    y_pred = lr.predict(X_test)

    # 计算R²分数并存储
    score = r2_score(y_test, y_pred)
    scores.append(score)
```



在循环内部：
1.  我们根据当前折的索引，从`X`和`y`中提取出对应的训练集和测试集。
2.  使用训练集`(X_train, y_train)`来拟合线性回归模型。
3.  使用拟合好的模型对测试集`X_test`进行预测，得到`y_pred`。
4.  将预测值`y_pred`与真实值`y_test`进行比较，计算R²分数。
5.  将每次计算得到的分数添加到`scores`列表中。




运行这段代码，将会输出三个不同的R²分数，分别对应三个不同的测试集。这些分数可能差异较大，这凸显了进行多次交叉验证的重要性。在完整的交叉验证流程中，我们通常会取这些分数的平均值作为模型性能的最终估计。



---





## 总结




本节课中，我们一起学习了交叉验证的基本实现。我们首先使用`KFold`对象将波士顿房价数据集分割成了三个互斥的测试折。然后，我们通过手动编写循环，在每一折上训练线性回归模型并评估其R²分数，直观地展示了模型性能如何随测试集的不同而变化。



![](img/29f79d9ce2580b5b47cbeffbe75c489f_167.png)

在下一节中，我们将在此基础上继续深入，引入数据标准化步骤，并使用Scikit-learn内置的`cross_val_predict`功能来更高效地进行交叉验证预测。