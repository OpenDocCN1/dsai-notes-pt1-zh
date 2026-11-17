# 022：探索性数据分析（EDA）笔记本解决方案（选修部分）第2部分 📊


在本节课中，我们将继续学习探索性数据分析（EDA）的实践操作。我们将重点解决关于数据统计摘要、分组聚合以及基础数据可视化的几个问题。通过具体的代码示例，你将学会如何计算各类统计量、按类别分组分析数据，并绘制简单的图表。







---


## 问题三：计算基本统计量




上一节我们查看了数据的基本信息，本节中我们来看看如何计算数据的具体统计摘要。我们需要确定以下内容：
1.  每个物种的数量。
2.  每个花瓣（Petal）和萼片（Sepal）测量值的均值、中位数、分位数（如25%、50%、75%）以及范围（最大值与最小值之差）。





以下是解决步骤：


首先，我们使用 `value_counts()` 方法计算每个物种的数量。




```python
data['species'].value_counts()
```




接下来，我们使用 `describe()` 方法获取数据的描述性统计摘要，它包含了计数、均值、标准差和各分位数。





```python
stats_df = data.describe()
```




`describe()` 方法提供了中位数（标记为50%分位数），但没有直接提供“范围”。因此，我们需要手动计算每个特征的最大值与最小值之差，并将其添加到统计摘要中。




```python
# 计算每个特征的范围（最大值 - 最小值）
range_series = data.max() - data.min()
# 将‘范围’作为新的一行添加到 stats_df 中
stats_df.loc['range'] = range_series
```



现在，我们有了包含所有所需统计量的DataFrame。我们只需要筛选出特定的行：均值（mean）、25%分位数、50%分位数（即中位数）、75%分位数和范围（range）。





```python
# 定义我们需要的行索引
out_fields = ['mean', '25%', '50%', '75%', 'range']
# 使用 .loc 方法筛选出行
final_stats = stats_df.loc[out_fields]
```





最后，为了更清晰地表示，我们将“50%”这一行的索引重命名为“median”。



```python
final_stats = final_stats.rename(index={'50%': 'median'})
```




运行以上所有代码后，我们就得到了问题三所要求的所有答案。




---



## 问题四：按物种分组计算统计量



在掌握了整体数据的统计信息后，本节我们将学习如何按不同类别（物种）进行分组分析。问题要求我们为每个物种分别计算以下内容：
1.  每个测量特征（萼片长/宽、花瓣长/宽）的**均值**。
2.  每个测量特征的**中位数**。
3.  将均值和**中位数**显示在同一个表格中。



以下是实现方法：




首先，我们使用 `groupby()` 方法按‘species’列分组，然后计算均值。




```python
mean_by_species = data.groupby('species').mean()
```



同样地，我们可以计算中位数。




```python
median_by_species = data.groupby('species').median()
```



为了将均值和**中位数**合并到一张表中，我们可以使用 `agg()`（aggregate的缩写）方法，并传入一个包含所需聚合函数的列表。



```python
# 方法一：使用预定义的聚合函数字符串
stats_single_table = data.groupby('species').agg(['mean', 'median'])
```




`agg()` 方法的优势在于灵活性。例如，如果我们想计算所有值的乘积（这不是一个内置的聚合函数），可以传递一个自定义的NumPy函数。



```python
import numpy as np
# 方法二：传递自定义的聚合函数
custom_agg = data.groupby('species').agg([np.mean, np.median, np.prod])
```



更进一步，如果我们希望对不同的特征应用不同的聚合方式（例如，对大多数特征求均值和中位数，但对花瓣长度只求最大值），可以传递一个字典给 `agg()` 方法。









```python
# 首先，为所有数值列（排除‘species’）创建默认字典：均值和**中位数**
agg_dict = {col: ['mean', 'median'] for col in data.columns if col != 'species'}
# 然后，修改特定列的聚合方式，例如将‘petal_length’改为只求‘max’
agg_dict['petal_length'] = 'max'
# 应用自定义的聚合字典
final_custom_table = data.groupby('species').agg(agg_dict)
```



通过这种方式，我们得到了一个灵活的、按物种分组的统计摘要表。



---





## 问题五：绘制散点图



数据分析不仅限于数字，可视化同样重要。本节我们将学习如何使用 Matplotlib 创建基础图表。问题要求我们绘制萼片长度（sepal length）与萼片宽度（sepal width）的散点图，并为坐标轴和图表添加标签。




以下是绘制步骤：



首先，需要导入 Matplotlib 库，并设置让图表在 Jupyter Notebook 中内联显示。




```python
import matplotlib.pyplot as plt
%matplotlib inline
```




接着，我们创建一个图形坐标轴（axes）对象，并在其上绘制散点图。`scatter` 方法的参数分别是 x 轴数据（萼片长度）和 y 轴数据（萼片宽度）。



```python
fig, ax = plt.subplots() # 创建图形和坐标轴
ax.scatter(data['sepal_length'], data['sepal_width']) # 绘制散点图
```




最后，我们为坐标轴和整个图表添加标签与标题。




```python
ax.set_xlabel('Sepal Length (cm)')
ax.set_ylabel('Sepal Width (cm)')
ax.set_title('Sepal Length vs. Sepal Width')
plt.show() # 显示图形
```




运行代码后，你将看到一张清晰的散点图，直观地展示了萼片长度与宽度之间的关系。




---


## 总结





本节课中我们一起学习了探索性数据分析（EDA）的几个核心实践技能：
1.  **计算综合统计量**：使用 `describe()`、手动计算范围，并通过筛选和重命名整理出所需的统计摘要。
2.  **分组聚合分析**：利用 `groupby()` 结合 `mean()`、`median()` 或更强大的 `agg()` 方法，按类别计算统计量，并能灵活地为不同特征指定不同的聚合函数。
3.  **基础数据可视化**：使用 Matplotlib 绘制散点图，掌握创建图表、添加数据以及设置坐标轴标签和标题的基本流程。




![](img/32459848e4cc7a7ac8fcd4405b0d10a8_200.png)

这些技能是理解数据集分布、关系和模式的基础，对于后续的机器学习建模工作至关重要。在接下来的课程中，我们将继续深入其他可视化方法和更复杂的数据探索技术。