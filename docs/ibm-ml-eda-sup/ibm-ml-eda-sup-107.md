# 107：K最近邻数据预处理实战 🧹





在本节课中，我们将学习如何为K最近邻（KNN）模型准备数据。核心任务是对不同类型的数据进行编码和缩放，以确保模型能够正确、无偏地处理它们。



上一节我们介绍了如何识别数据集中不同类型的特征。本节中，我们来看看如何具体地对这些特征进行预处理。


## 数据预处理步骤概述




我们的数据包含不同类型的特征：有序分类特征、无序分类特征、数值特征和二元特征。由于机器学习模型无法直接处理字符串，我们需要将它们全部转换为数值，并对数值特征进行缩放，以避免因量纲不同而引入偏差。



以下是数据预处理的三个主要步骤：
1.  **对分类特征进行编码**：将字符串转换为数值。
2.  **对数值特征进行缩放**：将所有数值特征调整到相近的尺度。
3.  **保存预处理后的数据**：方便后续直接使用，无需重复处理步骤。



## 步骤一：导入编码工具


首先，我们需要导入必要的编码工具。



```python
from sklearn.preprocessing import LabelBinarizer, LabelEncoder, OrdinalEncoder
```




我们将主要使用 `LabelBinarizer` 和 `LabelEncoder`，并初始化它们。



```python
LB = LabelBinarizer()
LE = LabelEncoder()
```



**注意**：我们在这里对有序特征使用 `LabelEncoder`，是因为我们的有序特征值（如合同类型、满意度等级）已经是按字母顺序排列的。如果顺序不是我们想要的，则应使用 `OrdinalEncoder` 并手动指定顺序。




## 步骤二：编码有序特征



接下来，我们对有序分类变量进行编码。有序变量包括 `contract`（合同类型）、`satisfaction`（满意度）和 `month`（月份分组）。


以下是具体操作：对每个有序变量列，我们使用 `LabelEncoder` 的 `fit_transform` 方法进行原地替换。




```python
df['month'] = LE.fit_transform(df['month'])
df['satisfaction'] = LE.fit_transform(df['satisfaction'])
df['contract'] = LE.fit_transform(df['contract'])
```



编码后，原本的字符串类别（例如月份的五个分组）被替换为整数（0, 1, 2, 3, 4）。我们可以检查唯一值的数量来确认。




```python
print(df[['month', 'satisfaction', 'contract']].nunique())
```

## 步骤三：编码二元特征





对于只有两个取值的二元特征，我们使用 `LabelBinarizer` 将其转换为0和1。这本质上是对二元变量的独热编码。

```python
for col in binary_variables:
    df[col] = LB.fit_transform(df[col])
```

## 步骤四：编码无序分类特征





最后，对于剩下的无序分类特征（既不是有序也不是二元的），我们需要进行完整的独热编码。我们使用Pandas的 `get_dummies` 函数。




```python
df = pd.get_dummies(df, columns=categorical_variables, drop_first=True)
```




这里设置 `drop_first=True` 是为了避免多重共线性。例如，如果一个特征有6个类别，独热编码后会生成6列。但其中一列的信息可以通过其他5列推导出来（如果其他5列都是0，则第6列为1）。因此，我们丢弃第一列以消除冗余信息。



## 步骤五：缩放数值特征



完成编码后，所有特征都变成了数值。但它们的尺度差异很大（例如“每月流量(GB)”约20，“月费用”约64）。为了不让KNN模型被大数值特征主导，我们需要进行缩放。



我们使用 `MinMaxScaler` 将数值特征（包括已编码的有序特征）缩放到0到1之间。对于已经是0和1的二元或独热编码特征，则无需再次缩放。



```python
from sklearn.preprocessing import MinMaxScaler
MM = MinMaxScaler()
df[ordinal_and_numeric_columns] = MM.fit_transform(df[ordinal_and_numeric_columns])
```



缩放后，再次使用 `df.describe()` 查看数据，可以看到所有指定列的数值范围都大致在0到1之间，处于相同的尺度上。




## 步骤六：保存预处理数据


所有预处理步骤完成后，我们将处理好的数据框保存为CSV文件，以便在后续建模时直接加载使用，无需重新运行预处理代码。




```python
output_file = 'preprocessed_data.csv'
df.to_csv(output_file, index=False)
```


---




![](img/6eeeac20ac3d52d82c3850278d81aad6_106.png)

本节课中我们一起学习了为KNN模型准备数据的完整流程。我们首先根据数据类型对特征进行分类，然后分别对**有序特征**、**二元特征**和**无序分类特征**进行了编码，接着使用 `MinMaxScaler` 对**数值特征**进行了缩放，最后保存了预处理结果。下一节，我们将使用这些处理好的数据，结合训练集/测试集划分，开始构建KNN模型来预测客户流失。