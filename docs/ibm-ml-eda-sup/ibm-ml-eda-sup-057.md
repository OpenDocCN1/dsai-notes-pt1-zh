# 057：线性回归演示（第一部分）📊



## 概述

在本节课中，我们将学习线性回归的基本设置，并演示如何通过数据变换使目标变量更符合正态分布，从而提升回归模型的效果。我们将使用Scikit-learn库运行一个简单的线性回归，并展示如何应用及逆变换变换后的数据。



---



## 导入必要库

首先，我们需要导入一些在整个演示中都会用到的库。我们已经熟悉了NumPy、Pandas和Matplotlib。

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
```




我们还需要从本课程专用的辅助Python文件中导入一些函数。这个文件包含了一些特定的工具函数。



```python
from helper import plot_exponential_data, plot_square_normal_data, boston_dataframe
```


`boston_dataframe`函数将输出一个Pandas DataFrame，我们将用它来进行回归分析。


---


## 加载并查看数据

现在，我们使用`boston_dataframe`函数加载波士顿房价数据集，并将其赋值给一个变量。



```python
boston_data = boston_dataframe()
```


我们可以使用`.head()`方法查看数据的前15行，以了解数据结构。


```python
print(boston_data.head(15))
```


我们试图预测的目标变量是最后一列`MEDV`，它代表特定区域房屋的中位价值。其余列是用于预测这个结果变量的不同特征。


---



## 评估目标变量的正态性


使目标变量呈正态分布通常能带来更好的模型结果。但请注意，线性回归并不要求结果变量本身必须正态分布，它要求的是**残差**（误差）正态分布。调整目标变量有助于满足这个前提。



我们可以通过两种方式判断目标变量是否正态分布：一是直观地查看数据直方图，二是进行统计检验。


首先，我们进行可视化检查。我们提取目标列`MEDV`，并使用Pandas内置的绘图功能绘制直方图。

```python
boston_data[‘MEDV’].hist()
plt.show()
```



从直方图右侧的长尾来看，数据可能不是正态分布的。


我们也可以从`scipy.stats`导入`normaltest`函数（也称为D’Agostino的K²检验）进行统计检验。这是一个用于检验分布是否正态的假设检验。


该检验会输出一个p值。p值越高，分布越接近正态。极低的p值意味着我们拒绝原假设（即数据是正态分布的）。



让我们对`MEDV`列运行正态性检验。


```python
from scipy.stats import normaltest
stat, p_value = normaltest(boston_data[‘MEDV’])
print(‘P-value:’, p_value)
```


我们得到的p值非常小（例如1.7e-20），远低于常见的显著性水平（如0.05）。因此，我们拒绝数据正态分布的原假设。



---


## 应用数据变换



既然原始数据不是正态分布，我们可以尝试一些常见的变换来使其更接近正态分布，这有助于满足线性回归对残差正态性的要求。


我们将尝试三种变换：
1.  对数变换
2.  平方根变换
3.  Box-Cox变换



### 1. 对数变换

对数变换适用于处理右偏（有长尾）的数据。理想情况下，变换后的数据直方图应更接近钟形曲线。



我们使用NumPy的`log`函数对`MEDV`列进行变换。


```python
log_medv = np.log(boston_data[‘MEDV’])
log_medv.hist()
plt.show()
```


变换后的直方图长尾有所改善，但可能仍不完美。我们再次运行正态性检验。

```python
stat, p_value = normaltest(log_medv)
print(‘P-value after log transform:’, p_value)
```




p值可能仍然很低，我们可能仍需拒绝正态分布的原假设。

### 2. 平方根变换


平方根变换是另一种处理非正态数据的方法。它特别适用于处理方差随均值增大的数据。

我们计算`MEDV`列的平方根。





```python
sqrt_medv = np.sqrt(boston_data[‘MEDV’])
sqrt_medv.hist()
plt.show()
```


绘制直方图并再次进行正态性检验。p值可能仍然表明数据不是完美的正态分布。



### 3. Box-Cox变换


Box-Cox变换是一种参数化的变换方法，它会自动寻找最佳的变换参数λ，使数据尽可能接近正态分布。其公式为：


`y(λ) = (y^λ - 1) / λ`， (当 λ ≠ 0)




你可以将其视为平方根变换（λ=0.5）和对数变换（λ趋近于0时）的推广。


我们从`scipy.stats`导入`boxcox`函数。

```python
from scipy.stats import boxcox
bc_result, fitted_lambda = boxcox(boston_data[‘MEDV’])
```

`boxcox`函数返回两个结果：第一个是变换后的数组，第二个是最优的λ值。



```python
print(‘Fitted lambda:’, fitted_lambda)
```

让我们查看变换后的直方图。



```python
pd.Series(bc_result).hist()
plt.show()
```

现在，数据看起来更接近正态分布了。运行正态性检验。



```python
stat, p_value = normaltest(bc_result)
print(‘P-value after Box-Cox transform:’, p_value)
```


此时的p值很可能高于0.05的阈值。因此，我们**不拒绝**数据正态分布的原假设，可以认为变换后的数据已足够接近正态分布。


---

## 总结



本节课中，我们一起学习了线性回归演示的第一部分。我们首先加载并查看了波士顿房价数据集，然后学习了如何评估目标变量的正态性。接着，我们探索了三种数据变换方法（对数变换、平方根变换和Box-Cox变换）来使目标变量更符合正态分布，并通过统计检验验证了Box-Cox变换的有效性。


![](img/0f5f8241b7900b44bbb6c570197d8c0d_123.png)

在下一节课中，我们将继续使用这个笔记本，学习如何用变换后的变量进行回归建模，并应用逆变换将预测值转换回原始尺度。