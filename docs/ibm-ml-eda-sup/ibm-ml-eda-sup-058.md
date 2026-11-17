# 058：线性回归实战演示（第二部分）📊



在本节课中，我们将继续学习线性回归的实际实现。我们将使用Scikit-learn库来构建一个线性回归模型，并对波士顿房价数据集进行预测。课程内容包括数据预处理、模型训练、评估以及预测。



---

## 导入必要的库


首先，我们需要导入实现线性回归所需的Scikit-learn模块。




```python
from sklearn.linear_model import LinearRegression
from sklearn.metrics import r2_score
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler, PolynomialFeatures
```



我们导入了以下核心模块：
*   `LinearRegression`：用于构建线性回归模型。
*   `r2_score`：用于计算R²分数，评估模型解释变异的能力。
*   `train_test_split`：用于将数据集分割为训练集和测试集。
*   `StandardScaler`：用于数据标准化，将所有特征缩放到相同尺度。
*   `PolynomialFeatures`：用于生成多项式特征（如二次项和交互项）。



---

## 初始化线性回归模型



接下来，我们初始化一个线性回归模型对象。



```python
lr = LinearRegression()
```




这里我们使用默认的超参数创建了一个`LinearRegression`对象，并将其赋值给变量`lr`。这个对象是一个预测器，在拟合数据后，可以使用`.predict()`方法进行预测。



---




## 准备数据



在训练模型之前，我们需要准备特征（X）和目标变量（y）。








```python
# 重新加载波士顿房价数据集，确保数据干净
boston_df = ... # 假设数据已加载
# 设置目标变量y为‘MEDV’（房价中位数）
y = boston_df['MEDV']
# 设置特征X为数据集中除‘MEDV’外的所有列
X = boston_df.drop('MEDV', axis=1)
```



*   `y` 是我们要预测的目标列。
*   `X` 是包含所有特征（预测变量）的数据框。


---




## 创建多项式特征

为了捕捉特征间的非线性关系，我们使用`PolynomialFeatures`来生成二次多项式特征。






```python
# 创建多项式特征转换器，设置degree=2，并排除偏置项（常数项）
pf = PolynomialFeatures(degree=2, include_bias=False)
# 将原始特征X转换为包含二次项和交互项的新特征矩阵
X_pf = pf.fit_transform(X)
```

*   `degree=2` 表示生成原始特征及其所有二阶组合（平方和两两交互）。
*   `include_bias=False` 表示不生成全为1的常数项列，因为线性回归模型会自行计算截距。
*   转换后，特征数量从13个大幅增加（具体数量为 `n_features + n_features^2`）。




---

## 分割数据集




为了评估模型在未见数据上的表现，我们需要将数据分为训练集和测试集。




```python
# 将多项式特征数据X_pf和目标y按70%/30%的比例分割
X_train, X_test, y_train, y_test = train_test_split(X_pf, y, test_size=0.3, random_state=42)
```

*   `test_size=0.3` 表示30%的数据用作测试集。
*   `random_state=42` 确保每次分割的结果一致，便于复现。




---



## 标准化特征数据




由于线性回归对特征尺度敏感，我们需要对训练集特征进行标准化。








```python
# 初始化标准化器
scaler = StandardScaler()
# 在训练集上拟合标准化器，并转换训练数据
X_train_s = scaler.fit_transform(X_train)
# 使用训练集拟合的标准化器转换测试集数据
X_test_s = scaler.transform(X_test)
```

**重要**：标准化必须在数据分割**之后**进行。我们使用训练集计算出的均值和标准差来转换**训练集和测试集**，以避免数据泄露。



---



## 转换目标变量



为了使目标变量更符合正态分布（线性回归的假设之一），我们对训练集的目标变量应用Box-Cox变换。



```python
# 假设boxcox_transform是之前定义好的Box-Cox变换函数
y_train_transformed, lambda_val = boxcox_transform(y_train)
```


*   该函数返回变换后的`y_train_transformed`和用于变换的参数`lambda_val`。
*   我们只对训练集进行此变换，并记录`lambda`值，以便后续对预测值进行逆变换。




---




## 训练线性回归模型



现在，我们使用标准化后的训练特征和变换后的训练目标来训练线性回归模型。


```python
# 使用训练数据拟合模型
lr.fit(X_train_s, y_train_transformed)
```


模型`lr`现在已学习到特征与（变换后的）目标变量之间的线性关系参数。


---


## 进行预测



模型训练完成后，我们使用它来对标准化后的测试集特征进行预测。



```python
# 对测试集进行预测
y_pred_transformed = lr.predict(X_test_s)
```




预测结果`y_pred_transformed`是基于变换后的目标变量尺度得到的。它的形状应与测试集样本数（例如152行）一致。


---



## 本节总结



本节课中，我们一起完成了线性回归模型构建的核心步骤：
1.  **导入库**：引入了Scikit-learn中的关键模块。
2.  **初始化模型**：创建了`LinearRegression`预测器。
3.  **数据预处理**：准备了特征和目标变量，并生成了多项式特征以捕捉非线性关系。
4.  **数据集分割**：将数据分为训练集和测试集。
5.  **特征标准化**：对训练集和测试集特征进行了标准化处理。
6.  **目标变量变换**：对训练集目标变量应用了Box-Cox变换。
7.  **模型训练**：使用处理后的训练数据拟合了线性回归模型。
8.  **模型预测**：使用拟合好的模型对测试集进行了预测。



![](img/dd2cad16dc77b1c7c8a964d9d61594a9_163.png)

在下节课中，我们将学习如何对预测结果进行逆变换，将其还原到原始房价尺度，并使用R²分数等指标来评估模型的最终性能。