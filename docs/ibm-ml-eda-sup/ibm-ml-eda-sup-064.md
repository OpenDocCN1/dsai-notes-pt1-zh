# 064：训练集与测试集分割实验（第三部分）📊


在本节课中，我们将学习如何对数据集进行训练集和测试集的分割，并比较两种不同特征处理方式（原始数值特征与独热编码特征）在模型上的表现差异。我们将使用线性回归模型，并通过均方误差来评估模型性能。




---



## 概述


我们将处理两个数据集：一个是原始的数值型特征数据集，另一个是经过独热编码处理后的数据集。我们将使用 `train_test_split` 函数对这两个数据集进行相同的分割，然后分别训练线性回归模型，并比较它们在训练集和测试集上的表现，以评估模型是否过拟合。



---


## 导入必要的库




首先，我们需要导入进行数据分割和模型训练所需的库。



```python
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error
import pandas as pd
```



---


## 定义目标变量



我们首先定义目标变量列的名称，以便后续从数据中分离特征和目标。


```python
y_col = 'SalePrice'
```




---


## 准备特征列

接下来，我们需要从数据集中提取特征列。以下是提取特征列的步骤：



```python
feature_cols = [x for x in data.columns if x != y_col]
```





这段代码会生成一个列表，包含除目标变量 `SalePrice` 之外的所有列名。


---



## 分离特征和目标



使用上面定义的特征列列表，我们可以将数据集分离为特征（X）和目标（y）。

```python
X_data = data[feature_cols]
y_data = data[y_col]
```




---



## 使用 `train_test_split` 分割数据




`train_test_split` 函数用于将数据集分割为训练集和测试集。以下是该函数的使用方法：


```python
X_train, X_test, y_train, y_test = train_test_split(X_data, y_data, test_size=0.3, random_state=42)
```



**参数说明：**
*   `X_data`: 特征数据。
*   `y_data`: 目标数据。
*   `test_size`: 测试集所占比例，这里设置为 30%。
*   `random_state`: 随机种子，确保每次分割结果一致。



函数返回四个部分：训练集特征、测试集特征、训练集目标、测试集目标。



---



## 对独热编码数据执行相同操作



现在，我们对经过独热编码处理的数据集 `data_ohc` 执行相同的分割操作。






```python
feature_cols_ohc = [x for x in data_ohc.columns if x != y_col]
X_data_ohc = data_ohc[feature_cols_ohc]
y_data_ohc = data_ohc[y_col]
X_train_ohc, X_test_ohc, y_train_ohc, y_test_ohc = train_test_split(X_data_ohc, y_data_ohc, test_size=0.3, random_state=42)
```


---




## 验证数据分割一致性


为了确保两个数据集的分割方式完全相同，我们可以检查它们训练集的索引是否一致。


```python
(X_train_ohc.index == X_train.index).all()
```




如果返回 `True`，则说明两个数据集的训练集包含了相同的样本，分割是一致的。

---




## 训练模型并评估性能


上一节我们准备好了数据，本节我们将使用线性回归模型进行训练和预测。我们将分别对原始数据集和独热编码数据集进行建模，并计算均方误差。

首先，初始化线性回归模型和一个用于存储误差的列表。




```python
lr = LinearRegression()
error_df = []
```




### 在原始数据上训练和评估

以下是使用原始数据训练模型并评估的步骤：













```python
# 1. 训练模型
lr.fit(X_train, y_train)
# 2. 在训练集上预测
y_train_pred = lr.predict(X_train)
# 3. 在测试集上预测
y_test_pred = lr.predict(X_test)
# 4. 计算并存储误差
error_df.append(pd.Series({'train': mean_squared_error(y_train, y_train_pred),
                           'test': mean_squared_error(y_test, y_test_pred)},
                          name='no_encoding'))
```



### 在独热编码数据上训练和评估

接下来，对独热编码数据重复上述过程：












```python
# 1. 训练模型
lr.fit(X_train_ohc, y_train_ohc)
# 2. 在训练集上预测
y_train_pred_ohc = lr.predict(X_train_ohc)
# 3. 在测试集上预测
y_test_pred_ohc = lr.predict(X_test_ohc)
# 4. 计算并存储误差
error_df.append(pd.Series({'train': mean_squared_error(y_train_ohc, y_train_pred_ohc),
                           'test': mean_squared_error(y_test_ohc, y_test_pred_ohc)},
                          name='one_hot_encoding'))
```



---



## 比较模型性能



最后，我们将存储的误差结果合并成一个数据框，以便直观比较。

```python
error_df = pd.concat(error_df, axis=1)
error_df
```


观察结果，我们可能会发现：
*   在**训练集**上，独热编码模型的误差可能更低。
*   但在**测试集**上，原始数据模型的误差可能更低，泛化能力更好。




这种训练集误差低而测试集误差高的情况，通常是模型**过拟合**的标志。独热编码引入了大量特征，增加了模型复杂度，使其过于“贴合”训练数据，从而在新数据上表现不佳。

---




## 总结

本节课中，我们一起学习了：
1.  如何使用 `train_test_split` 分割数据集。
2.  如何为原始数据和独热编码数据分别准备特征和目标变量。
3.  如何训练线性回归模型并计算均方误差来评估性能。
4.  通过比较两种特征处理方式的误差，理解了模型复杂度和过拟合之间的关系：特征并非越多越好，过于复杂的模型可能在训练集上表现优异，但在未知数据上泛化能力会下降。




![](img/788a54d5e2caf7951b0d074f24ad1c8f_172.png)

在接下来的课程中，我们将探讨数据缩放以及如何可视化预测结果与实际值。