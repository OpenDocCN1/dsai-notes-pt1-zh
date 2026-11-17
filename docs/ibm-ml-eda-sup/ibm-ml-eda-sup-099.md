# 099：逻辑回归实验（第二部分）📊








在本节课中，我们将继续逻辑回归实验，重点学习如何将数据拆分为训练集和测试集，以及如何拟合和比较带有不同正则化项（L1和L2）的逻辑回归模型。我们将使用分层抽样来确保数据分布的一致性，并通过交叉验证来寻找最优的超参数。




---





## 数据拆分与分层抽样 📂





上一节我们完成了数据预处理。本节中，我们来看看如何将数据拆分为训练集和测试集。






为了确保训练集和测试集中预测类别的比例保持一致，我们使用Scikit-learn的`StratifiedShuffleSplit`方法进行分层抽样。无论使用何种拆分方法，之后都应比较训练集和测试集中类别的比例。





以下是使用`StratifiedShuffleSplit`拆分数据的步骤：




1.  从`sklearn.model_selection`导入`StratifiedShuffleSplit`。
2.  创建`StratifiedShuffleSplit`对象，并设置参数：
    *   `n_splits=1`：表示只进行一次拆分，生成一组训练集和测试集。
    *   `test_size=0.3`：表示30%的数据作为测试集（保留集），70%作为训练集。
    *   `random_state=42`：设置随机种子以确保结果可复现。
3.  调用`split`方法，传入特征数据`X`和目标变量`y`（即`data.activity`）。
4.  使用生成的索引从原始数据集中提取出对应的训练集和测试集。














```python
from sklearn.model_selection import StratifiedShuffleSplit
# 创建分层抽样拆分器
sss = StratifiedShuffleSplit(n_splits=1, test_size=0.3, random_state=42)
# 生成训练和测试索引
for train_index, test_index in sss.split(X, y):
    X_train, X_test = X.iloc[train_index], X.iloc[test_index]
    y_train, y_test = y.iloc[train_index], y.iloc[test_index]
```





拆分完成后，我们可以检查`y_train`和`y_test`中各类别的比例是否相似。






```python
print(y_train.value_counts(normalize=True))
print(y_test.value_counts(normalize=True))
```





---



## 拟合逻辑回归模型 🔧








现在，我们开始拟合逻辑回归模型。首先，我们拟合一个没有正则化的基础模型，然后使用交叉验证来寻找L1和L2正则化的最优超参数。



### 1. 无正则化逻辑回归





我们使用`solver='liblinear'`来拟合一个多分类逻辑回归模型。`liblinear`求解器适用于“一对多”（OvR）策略，即为每个类别训练一个二分类器（例如，“站立” vs “非站立”）。








```python
from sklearn.linear_model import LogisticRegression
# 拟合无正则化的逻辑回归模型
lr = LogisticRegression(solver='liblinear').fit(X_train, y_train)
```






### 2. 带正则化的逻辑回归（使用交叉验证）




接下来，我们使用`LogisticRegressionCV`进行交叉验证，以自动寻找L1和L2正则化的最佳强度参数`C`（`C`是正则化强度λ的倒数，`C`值越小，正则化越强）。

















```python
from sklearn.linear_model import LogisticRegressionCV
# 拟合带L1正则化的逻辑回归模型（交叉验证选择最佳C值）
lr_l1 = LogisticRegressionCV(Cs=10, cv=4, penalty='l1', solver='liblinear').fit(X_train, y_train)
# 拟合带L2正则化的逻辑回归模型
lr_l2 = LogisticRegressionCV(Cs=10, cv=4, penalty='l2', solver='liblinear').fit(X_train, y_train)
```




**注意**：L1正则化的拟合过程可能较慢，因为它会尝试将许多特征的系数压缩至零（特征选择）。L2正则化通常计算更快。






---



## 比较模型系数 📈





由于我们采用了“一对多”策略，每个模型（LR、L1、L2）都会为6个类别分别生成一组系数。每组系数有561个值（对应561个特征），描述了该特征对“特定类别 vs 其他所有类别”的对数几率的影响。




我们需要将这些系数整理并可视化，以便比较。





以下是提取和整理所有模型系数的代码步骤：





1.  创建一个空列表来存储系数DataFrame。
2.  遍历模型标签和模型对象。
3.  从每个模型中提取系数。
4.  为系数DataFrame创建多层列索引（第一层是模型类型，第二层是类别标签）。
5.  将每个模型的系数DataFrame添加到列表中。
6.  最后，将所有DataFrame沿列方向拼接起来。






















```python
import pandas as pd
coefficients = []
coef_labels = ['LR', 'L1', 'L2']
coef_models = [lr, lr_l1, lr_l2]
for label, model in zip(coef_labels, coef_models):
    # 获取模型系数
    coef_values = model.coef_
    # 创建多层列索引
    multi_index = pd.MultiIndex.from_product([[label], range(coef_values.shape[0])], names=['model', 'class'])
    # 将系数转换为DataFrame并转置，使每个类别为一列
    coef_df = pd.DataFrame(coef_values.T, columns=multi_index)
    coefficients.append(coef_df)
# 沿列方向拼接所有系数
all_coefficients = pd.concat(coefficients, axis=1)
```




### 可视化系数





我们将所有模型的系数绘制出来，以直观比较不同正则化方式对系数大小（绝对值）的影响。





















```python
import matplotlib.pyplot as plt
fig, axes = plt.subplots(nrows=3, ncols=2, figsize=(12, 10))
axes = axes.flatten()  # 将轴数组展平为一维，便于循环
for i, ax in enumerate(axes):
    # 为每个子图选择对应类别（0-5）在所有模型下的系数
    coef_data = all_coefficients.xs(i, level=1, axis=1)
    # 绘制散点图
    ax.scatter(range(len(coef_data)), coef_data['LR'], s=2, label='LR', marker='o')
    ax.scatter(range(len(coef_data)), coef_data['L1'], s=2, label='L1', marker='o')
    ax.scatter(range(len(coef_data)), coef_data['L2'], s=2, label='L2', marker='o')
    # 只在第一个子图添加图例
    if i == 0:
        ax.legend(loc=4)
    ax.set_title(f'Coefficients for Class {i} vs Rest')
plt.tight_layout()
plt.show()
```






通过散点图，我们可以观察到：
*   **无正则化（LR）**：系数值可能分布很广，有些会非常大。
*   **L1正则化**：倾向于产生稀疏解，许多系数被压缩为零（在图上表现为集中在0点附近）。
*   **L2正则化**：倾向于将系数整体缩小，但很少完全为零（系数分布更集中，但范围比LR小）。





---





## 总结 ✨




本节课中我们一起学习了逻辑回归实验的关键步骤：






1.  **数据拆分**：使用`StratifiedShuffleSplit`进行分层抽样，确保训练集和测试集的类别分布一致。
2.  **模型拟合**：
    *   拟合了无正则化的基础逻辑回归模型。
    *   使用`LogisticRegressionCV`通过交叉验证拟合了带L1和L2正则化的模型，并自动选择了最佳的正则化强度`C`。
3.  **系数分析与比较**：提取并整理了“一对多”策略下每个模型、每个类别的系数。通过可视化发现，L1正则化能产生稀疏模型（特征选择），而L2正则化则使所有系数均匀缩小。





![](img/916255a354808a4764b2e500da6f91ce_262.png)

在下一节中，我们将使用训练好的模型进行预测，并计算每个预测类别的概率，以评估模型的性能。