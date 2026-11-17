# 166：LDA算法流程与代码实现 🧮




在本节课中，我们将学习线性判别分析（LDA）算法的具体实现流程，并通过代码演示其核心计算步骤。LDA是一种有监督的降维方法，其目标是找到能够最大化类间距离、最小化类内距离的特征投影方向。



---

## 导入必要的库

首先，我们需要导入实现LDA所需的Python库。


```python
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis
from sklearn import datasets
import numpy as np
from scipy import linalg
```



## 加载数据




我们使用鸢尾花数据集进行演示，该数据集包含特征`X`和对应的类别标签`y`。


```python
iris = datasets.load_iris(return_X_y=True)
X, y = iris
print(X[:5])
print(y[:5])
```




## 使用sklearn的LDA进行降维


我们可以直接使用`sklearn`中的`LinearDiscriminantAnalysis`类来快速实现降维。


```python
lda = LinearDiscriminantAnalysis(solver='eigen')
X_lda = lda.fit_transform(X, y)
print(X_lda[:5])
```

与PCA不同，LDA的`fit`方法需要同时传入特征`X`和标签`y`。参数`n_components`用于指定降维后的特征数量，但其值不能超过`min(n_features, n_classes - 1)`。



## 手动实现LDA算法流程




上一节我们使用了现成的库，本节中我们来看看LDA算法背后的具体计算步骤，以加深理解。



### 1. 计算总的散度矩阵



总的散度矩阵（Total Scatter Matrix）描述了所有数据的总体离散程度，可以用协方差矩阵来表示。


```python
# 去中心化
X_centered = X - X.mean(axis=0)
# 计算总的散度矩阵 St
St = np.cov(X_centered.T, bias=1)
```


### 2. 计算类内的散度矩阵





类内散度矩阵（Within-class Scatter Matrix）衡量了同一类别内部数据的离散程度。我们需要对每个类别分别计算其协方差矩阵，然后求和。



以下是计算类内散度矩阵`Sw`的步骤：


```python
# 初始化Sw为零矩阵
Sw = np.zeros((X.shape[1], X.shape[1]), dtype=np.float64)
# 遍历每个类别
for i in range(3):  # 鸢尾花数据有3个类别
    # 获取属于当前类别的所有样本点
    X_class = X[y == i]
    # 计算当前类别的协方差矩阵（散度矩阵）
    Sw_class = np.cov(X_class.T, bias=1)
    # 累加到总的类内散度矩阵中
    Sw += Sw_class
```



### 3. 计算类间的散度矩阵

类间散度矩阵（Between-class Scatter Matrix）描述了不同类别中心之间的离散程度。它可以通过总的散度矩阵减去类内散度矩阵得到。


```python
Sb = St - Sw
```



### 4. 计算特征值和特征向量





LDA的核心是求解广义特征值问题：**Sb * w = λ * Sw * w**。我们需要找到能使类间散度最大、类内散度最小的投影方向`w`，即对应的特征向量。




```python
eig_vals, eig_vecs = linalg.eigh(Sb, Sw)
```



### 5. 筛选特征向量



我们需要根据特征值的大小对特征向量进行排序，并选择最大的`k`个特征值对应的特征向量，其中`k`是我们希望降维到的维度（`n_components`）。





```python
# 对特征值进行从大到小排序，并获取对应的索引
sorted_index = np.argsort(eig_vals)[::-1]
# 根据排序后的索引，对特征向量进行重排
sorted_eig_vecs = eig_vecs[:, sorted_index]
# 选择前k个特征向量（例如k=2）
k = 2
selected_eig_vecs = sorted_eig_vecs[:, :k]
```





### 6. 进行矩阵运算得到降维结果




最后，将原始数据投影到筛选出的特征向量所张成的子空间上，即可得到降维后的数据。





```python
X_lda_manual = X_centered.dot(selected_eig_vecs)
print(X_lda_manual[:5])
```




---




## 总结



本节课中我们一起学习了LDA算法的完整流程。我们首先使用`sklearn`库快速实现了LDA降维，然后手动实现了其核心计算步骤，包括：





1.  计算总的散度矩阵 **St**。
2.  计算类内的散度矩阵 **Sw**。
3.  计算类间的散度矩阵 **Sb**。
4.  求解广义特征值问题，得到特征值和特征向量。
5.  根据特征值筛选特征向量。
6.  将数据投影到新的特征子空间，完成降维。




![](img/0fcaad10095c3ab2d10d9840c3d3f388_162.png)

LDA通过最大化类间散度与类内散度的比值来寻找最佳投影方向，是一种非常有效的有监督降维方法。理解其计算过程有助于我们更好地应用和调优该算法。