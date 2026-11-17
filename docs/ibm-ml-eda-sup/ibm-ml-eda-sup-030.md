# 030：特征工程实验解决方案（第一部分）🔧



在本节课中，我们将学习特征工程的核心实践。我们将使用Ames住房数据集，并围绕线性回归模型进行特征工程。这些技术同样适用于未来的许多机器学习模型。


## 数据加载与初步探索 📊


首先，我们需要导入必要的库并进行简单的探索性数据分析。




```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np
```


运行 `sns.set()` 是为了为所有Seaborn图表设置默认的输出样式。


接下来，指定我们将要使用的文件路径。这里我们使用一个TSV文件，它代表制表符分隔值文件。我们仍然可以使用 `read_csv` 函数，只需将分隔符参数设置为制表符 `\t`。



```python
df = pd.read_csv('path/to/your/file.tsv', sep='\t')
```



## 数据信息概览

运行 `df.info()` 方法将为我们提供三个重要信息。

以下是 `df.info()` 输出的核心信息：

1.  所有列的名称。
2.  每一列的非空值数量。例如，我们看到有198个非空值，考虑到总共有2930个条目，这意味着我们可能缺失了很多值。
3.  每一列的数据类型。我们可以看到整数、对象和浮点数类型。其中，“对象”类型通常指分类变量。


## 处理异常值


我们将查看特定列的直方图。作者指出该列存在一些异常值，我们通过快速查看来确认。可以看到，在4000以上的数据点频率很低，这正是作者提到的异常值所在区域。

因此，我们将过滤掉所有小于4000的值。

```python
df = df.loc[df['Gr Liv Area'] < 4000, :]
```



接着，我们想查看数据框的新形状。调用 `.shape` 属性会输出一个元组，第一个值是行数，第二个值是列数。


最后，最好保留一份数据的原始副本以备后用。使用 `df.copy()` 可以确保我们对 `df` 所做的更改不会影响初始副本。



```python
df_original = df.copy()
```




## 数据预览与无关特征剔除


快速查看前五行数据。


```python
df.head()
```


需要注意的是前两列 `Order` 和 `PID`。有多种方式可以检查它们，例如查看 `df[‘PID’].unique()` 的长度。我们会发现其长度与行数相同，这意味着该列的每一行都是唯一值。对于机器学习模型来说，这样的列无法提供任何预测价值，因为模型无法从全是唯一值的特征中学到任何模式。


因此，我们将删除这两列。使用 `.drop` 功能并指定要删除的列 `PID` 和 `Order`。必须设置 `axis=1`，因为默认情况下它会寻找行索引，而我们需要指定列索引。`inplace=True` 表示直接在原数据框上进行修改。

```python
df.drop(['PID', 'Order'], axis=1, inplace=True)
```




## 特征变换：处理偏斜数据 📈


现在开始进行一些特征变换。我们将对偏斜变量使用对数变换。



首先，需要提取出数值型变量。使用 `select_dtypes` 功能，传入参数 `‘number’`，这将把列过滤为仅包含浮点数和整数的数值型列。这在处理Pandas数据框时非常有用。



```python
numeric_cols = df.select_dtypes(include=['number']).columns
```



我们将偏斜限度设为一个变量，稍后会用到。我们需要寻找每个列中严重偏斜的分布。



正态分布的偏度为0。偏度值大于0表示右偏，小于0表示左偏。通常，如果偏度值在0.75以下，数据分布可能没有严重偏斜；如果高于0.75，则可能表明分布严重偏斜，应该进行某种变换。



为了查看实际的偏度值，我们运行 `df.skew()`。默认情况下，`.skew()` 会过滤到数值型列。`skew_vals` 将给出每个数值列的偏度值。


接下来，我们想过滤出偏度绝对值大于我们设定的偏斜限度的列。我们使用 `skew_vals[abs(skew_vals) > skew_limit]` 来实现。然后，使用 `sort_values(ascending=False)` 将这些值从大到小排序。


```python
skew_limit = 0.75
skew_vals = df.skew()
skew_cols = skew_vals[abs(skew_vals) > skew_limit].sort_values(ascending=False)
```



现在，我们得到了一个只包含偏度值高于0.75的序列，最偏斜的数据在顶部，偏斜程度相对较小的在底部。




## 可视化变换效果



现在快速查看一下经过变换和未变换的数据分布对比。我们将以 `SalePrice` 列为例。


使用 `plt.subplots()` 可以在一个图形中绘制两个坐标轴。坐标轴是图形的边界框，我们将在一个图形中放置两个边界框。`figsize` 参数指定图形大小为10x5。



```python
field = 'SalePrice'
fig, (ax_before, ax_after) = plt.subplots(1, 2, figsize=(10, 5))
```



首先，在第一个坐标轴 `ax_before` 上绘制原始 `SalePrice` 的直方图。这里使用Pandas内置的绘图功能，通过 `df[field].hist(ax=ax_before)` 实现。我们看到一个右偏的数据集，这与我们之前看到的约1.6的正偏度值相符。


接着，在第二个坐标轴 `ax_after` 上绘制变换后的数据。我们使用 `np.log1p` 函数对数据进行对数变换，并通过 `.apply` 方法应用。`np.log1p` 计算的是 `log(1 + x)`，能更好地处理零值。变换后，我们得到了一个更接近正态分布的数据。




```python
df[field].hist(ax=ax_before)
df[field].apply(np.log1p).hist(ax=ax_after)
```



我们可以分别为两个坐标轴设置标题和轴标签，并为整个图形设置一个总标题。



```python
ax_before.set(title='Original Distribution', ylabel='Frequency', xlabel=field)
ax_after.set(title='Log-Transformed Distribution', ylabel='Frequency', xlabel=field)
fig.suptitle(f'Field: {field}')
plt.show()
```



## 应用对数变换



最后，我们对所有偏度高于阈值的列（除了目标变量 `SalePrice`）应用对数变换。



我们遍历 `skew_cols.index`（即那些偏度高的列名）。对于每一列，如果列名不等于 `SalePrice`，则使用 `continue` 跳过；否则，将该列的值设置为对其取对数后的值。


```python
for col in skew_cols.index:
    if col == 'SalePrice':
        continue
    df[col] = np.log1p(df[col])
```


运行这段代码后，我们就完成了对所有高偏度列的对数变换。



## 本节总结



![](img/d55700b8861616a13f8e694d2a6c644e_125.png)

在本节课中，我们一起学习了特征工程实验的第一部分。我们首先加载并探索了Ames住房数据集，处理了异常值并移除了无关特征。接着，我们重点学习了如何处理数值型特征的偏斜分布：通过计算偏度识别出需要变换的列，并使用对数变换使其更接近正态分布，最后通过可视化对比了变换前后的效果。在下一节中，我们将继续探讨数据集中剩余的特征处理，并学习独热编码技术。