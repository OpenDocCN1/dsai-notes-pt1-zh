# 098：逻辑回归实验（第一部分）🔍

在本节课中，我们将学习逻辑回归以及误差度量的应用。我们将使用“带智能手机的人类活动识别”数据库，该数据库通过让研究参与者携带内置惯性传感器的智能手机四处走动而构建。我们的目标是，根据这些数据，判断参与者是在行走、上楼梯、下楼梯、坐着、站立还是躺着。这是一个分类问题，我们需要为数据贴上这六个类别之一的标签。





## 1. 导入库与设置工作目录 📂

首先，我们需要导入必要的库，这与我们之前的操作类似。我们还将运行 `os.chdir`（change directory 的缩写）来设置工作目录。我们将工作目录设置为当前文件夹内的一个名为 `data` 的子文件夹，这样可以永久更改路径，以便轻松访问该 `data` 文件夹内的任何文件。




以下是导入库和设置目录的代码：



```python
import pandas as pd
import numpy as np
import os
from sklearn.preprocessing import LabelEncoder
import matplotlib.pyplot as plt
# 更改工作目录到 'data' 文件夹
os.chdir('./data')
```



## 2. 数据导入与初步探索 📊


对于问题1，我们需要导入数据并执行以下操作：检查不同的数据类型、查看各列的值计数和数据类型、确定浮点数值是否需要缩放、分析每种活动的分布情况，最后将活动标签编码为整数，因为 Scikit-learn 模型无法处理字符串输入。



首先，我们使用 `pandas.read_csv` 读取数据文件。



```python
data = pd.read_csv('your_data_file.csv')
```




接着，我们查看各列的数据类型。运行 `data.dtypes` 可以获取每列的数据类型。



```python
data_types = data.dtypes
print(data_types)
```

为了查看不同数据类型的出现次数，我们可以像处理任何列一样对 `data.dtypes` 运行 `value_counts`。





```python
data_type_counts = data.dtypes.value_counts()
print(data_type_counts)
```


我们看到有561个浮点数列和1个对象列。这个对象列正是我们要预测的目标，它包含了“行走”、“上楼梯”、“躺着”等不同的字符串。

我们可以通过查看数据类型的最后五个值来确认这一点，会发现 `activity` 列就是那个对象类型。






```python
print(data.dtypes.tail())
```



然后，我们发现数据的值域已被缩放到最小值-1到最大值1之间。为了证明这一点，我们将检查所有列的最小值和最大值。


首先，我们查看所有列的最小值。我们选取除最后一列（目标列）外的所有行，然后对每列取最小值。

```python
min_values = data.iloc[:, :-1].min()
min_value_counts = min_values.value_counts()
print(min_value_counts)
```



我们看到所有列的最小值都是-1，共有561个-1。接着，我们对最大值进行同样的操作。



```python
max_values = data.iloc[:, :-1].max()
max_value_counts = max_values.value_counts()
print(max_value_counts)
```



我们发现所有列的最大值都被缩放到1，最小值被缩放到-1，中间有不同的值。



接下来，我们查看每种活动的分布情况。

```python
activity_counts = data['activity'].value_counts()
print(activity_counts)
```





我们看到结果变量（目标列）的分布相当均衡，每种活动在总行数中占据大致相等的比例。当我们看到这种平衡的数据集时，正如课程中讨论的，我们需要开始思考哪种误差度量最适合平衡数据集。这与我们在课程中讨论的白血病例子（99%健康，1%患病，非常不平衡）形成对比。


然后，如前所述，我们不能将字符串直接传入 Scikit-learn 模型（这里将是逻辑回归）。因此，我们必须将其编码为整数。我们将创建一个标签编码器对象。



首先从 `sklearn.preprocessing` 导入 `LabelEncoder`，然后像处理其他 Scikit-learn 对象一样实例化它。



```python
from sklearn.preprocessing import LabelEncoder
le = LabelEncoder()
```

接着，在我们数据集的 `activity` 列上调用这个刚创建的对象 `le` 的 `fit_transform` 方法。





```python
data['activity'] = le.fit_transform(data['activity'])
```

现在，我们的 `activity` 列已被更改为整数，如示例所示，范围从0到5，分别对应之前的六个不同类别。





## 3. 计算特征间相关性 🔗

对于问题2，我们需要计算自变量之间的相关性，创建不同相关性值的直方图，并识别出彼此高度相关（无论是正相关还是负相关）的特征。





我将把这个过程分解为几个步骤。首先，我们定义特征列，即除最后一列（目标列）外的所有列。



```python
feature_columns = data.columns[:-1]
```



为了获取相关性矩阵，我们只需指定这些特征列并调用 `.corr()` 方法。这将输出一个 Pandas DataFrame，即相关性矩阵。




```python
corr_matrix = data[feature_columns].corr()
```


我们注意到，相关性矩阵中 x 和 y 的相关性与 y 和 x 的相关性相同，并且对角线上的值都是1（与自身的完全相关）。因此，矩阵的下三角部分（包括对角线）没有提供新信息。我们最终需要移除这些值。


首先，我们使用 `np.tril_indices_from` 来获取下三角矩阵（包括对角线）的所有索引。



```python
indices = np.tril_indices_from(corr_matrix)
```



一个高效的处理方法是确保我们操作的是 NumPy 数组。我们将当前的 Pandas DataFrame 转换为数组。




```python
corr_array = corr_matrix.values
```



然后，对于我们刚刚定义的那些索引，将它们设置为 `np.nan`（空值）。


```python
corr_array[indices] = np.nan
```




在将下三角区域置空后，我们将其转换回 DataFrame，并设置列名和索引为原始值。



```python
corr_df = pd.DataFrame(corr_array, columns=corr_matrix.columns, index=corr_matrix.index)
```


接着，我们“堆叠”这个 DataFrame。`.stack()` 操作会将其转换为一个多级索引的 Series，其中每一行对应一对特征及其相关性值。




```python
corr_series = corr_df.stack()
```


然后，我们重置索引，使其成为一个规整的 DataFrame。


```python
corr_values = corr_series.reset_index()
```





我们将列重命名为 `feature1`、`feature2` 和 `correlation`。

```python
corr_values.columns = ['feature1', 'feature2', 'correlation']
```




我们还会创建一个新列 `abs_correlation`，它是相关性的绝对值，因为我们只关心相关性的强度，而不关心正负。


```python
corr_values['abs_correlation'] = corr_values['correlation'].abs()
```




## 4. 可视化与分析相关性分布 📈



现在，我们可以开始绘制不同相关性值的分布，以查看各列之间高相关性与低相关性的分布情况。


我们设置绘图轴，数据是 `corr_values` DataFrame 中的 `abs_correlation` 列，然后调用直方图函数，设置50个箱子。





```python
plt.figure(figsize=(10, 6))
plt.hist(corr_values['abs_correlation'], bins=50)
plt.xlabel('Absolute Correlation')
plt.ylabel('Frequency')
plt.title('Distribution of Absolute Correlations Between Features')
plt.show()
```


运行后，我们看到大多数情况下相关性基本为零，在接近1的高度相关区域则少得多。整体分布相对均匀，但在低值区域有所下降。




然后，我们查看高度相关的值。我们按相关性对值进行排序（降序）。

```python
sorted_corr = corr_values.sort_values(by='correlation', ascending=False)
```





接着，我们使用 `.query()` 方法过滤出绝对值大于0.8的相关性。


```python
high_corr = sorted_corr.query('abs_correlation > 0.8')
print(high_corr.head())
```

我们可以查看 `high_corr` 的大小，并与原始 `corr_values` DataFrame 的大小进行比较。原始相关性值非常多，因为它是所有交叉相关的组合。当我们看到如此高度相关的特征时，可能需要进行一些特征工程或特征选择，这将在课程后续部分进一步讨论。






本节课中，我们一起学习了逻辑回归实验的第一部分，包括数据导入、探索性分析、标签编码以及特征相关性的计算与可视化。在下一个视频中，我们将继续在笔记本中讨论训练集-测试集划分。



![](img/44b9d00c4609e8d52e8498aeed0bcf81_190.png)

---
**总结**：本节课我们完成了逻辑回归实验的数据准备阶段。我们导入了库与数据，探索了数据类型与分布，将字符串标签编码为整数，并深入分析了特征间的相关性，为后续的模型训练与评估奠定了基础。