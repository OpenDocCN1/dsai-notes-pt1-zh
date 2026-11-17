# 129：决策树回归与可视化 🌳






在本节课中，我们将学习如何使用决策树进行回归任务，并探索如何将训练好的决策树模型可视化。我们将从分类问题转向回归问题，预测葡萄酒的残糖量这一连续值。


## 从分类到回归 🔄







上一节我们介绍了决策树在分类问题上的应用。本节中，我们将看看如何使用决策树进行回归预测。





我们将使用之前定义的分层随机划分索引来创建训练集和测试集。




```python
# 使用分层随机划分索引创建训练集和测试集
X_train = data.iloc[train_idx][feature_columns]
y_train = data.iloc[train_idx]['residual sugar']
X_test = data.iloc[test_idx][feature_columns]
y_test = data.iloc[test_idx]['residual sugar']
```



## 网格搜索与交叉验证 🎯





我们将使用网格搜索配合交叉验证，来寻找在留出集上表现最佳的超参数组合。





由于我们进行的是回归任务，预测的是连续值，因此不能使用F1分数或召回率等分类指标。我们将使用**均方误差**来衡量预测误差。




均方误差公式为：
**MSE = (1/n) * Σ(y_i - ŷ_i)^2**





以下是实施步骤：



1.  首先，我们需要定义特征列。特征列是数据中除“残糖量”之外的所有列。
2.  然后，我们导入`DecisionTreeRegressor`。它与分类器的主要区别在于，`y`变量必须是连续值。
3.  我们首先创建一个决策树回归器，并在训练集上拟合。这样做是为了查看在不进行任何剪枝或正则化的情况下，树的最大深度，以便为网格搜索设置参数范围。
4.  我们设置参数网格，例如`max_depth`从1到上一步得到的最大深度，步长为2。
5.  我们将这些参数传入`GridSearchCV`。这里的关键点是，由于网格搜索默认是最大化一个分数，而我们需要最小化均方误差，因此我们将评分标准设置为`neg_mean_squared_error`。
6.  拟合网格搜索对象后，我们可以得到最佳估计器，并查看其节点数和最大深度。
























```python
from sklearn.tree import DecisionTreeRegressor
from sklearn.model_selection import GridSearchCV
# 创建回归器并拟合，以查看最大深度
dt_temp = DecisionTreeRegressor(random_state=42)
dt_temp.fit(X_train, y_train)
max_depth_unpruned = dt_temp.tree_.max_depth
# 定义参数网格
param_grid = {
    'max_depth': range(1, max_depth_unpruned+1, 2),
    'max_features': ['sqrt', 'log2', None]
}
# 创建网格搜索对象，使用负均方误差作为评分
grid_search = GridSearchCV(estimator=DecisionTreeRegressor(random_state=42),
                           param_grid=param_grid,
                           scoring='neg_mean_squared_error',
                           cv=5,
                           n_jobs=-1)
grid_search.fit(X_train, y_train)
best_estimator = grid_search.best_estimator_
print(f"最佳模型节点数: {best_estimator.tree_.node_count}")
print(f"最佳模型最大深度: {best_estimator.tree_.max_depth}")
```



## 评估回归性能 📊






现在，我们来评估模型在训练集和测试集上的性能。




1.  使用最佳模型对训练集和测试集进行预测。
2.  计算训练集和测试集的均方误差。
3.  将结果整理成`DataFrame`以便查看。





我们通常会发现，模型在训练集上的误差远小于在测试集上的误差。这是因为决策树回归器在训练集上通过不断分割来获取均值，容易过拟合。但请记住，我们通过网格搜索根据留出集调整了模型，这是使用网格搜索的关键。
















```python
from sklearn.metrics import mean_squared_error
import pandas as pd
# 进行预测
y_train_pred = grid_search.predict(X_train)
y_test_pred = grid_search.predict(X_test)
# 计算均方误差
mse_train = mean_squared_error(y_train, y_train_pred)
mse_test = mean_squared_error(y_test, y_test_pred)
# 将结果放入DataFrame
mse_results = pd.DataFrame({'MSE': [mse_train, mse_test]},
                           index=['Train', 'Test']).T
print(mse_results)
```






## 可视化预测结果 📈




我们可以绘制实际值与预测值的散点图来直观评估回归效果。




如果预测完全准确，所有点都应落在对角线上。我们的点越接近这条对角线，说明预测越准确。




























```python
import matplotlib.pyplot as plt
import pandas as pd
# 创建图形
fig, ax = plt.subplots(figsize=(8, 8))
# 将实际值和预测值放入DataFrame并绘图
results_df = pd.DataFrame({'Actual': y_test, 'Predicted': y_test_pred})
results_df.plot(kind='scatter', x='Actual', y='Predicted', ax=ax, marker='o', linestyle='')
# 设置坐标轴标签和范围
ax.set_xlabel('Actual Residual Sugar')
ax.set_ylabel('Predicted Residual Sugar')
ax.set_xlim([y_test.min(), y_test.max()])
ax.set_ylim([y_test_pred.min(), y_test_pred.max()])
# 添加对角线参考线
ax.plot([y_test.min(), y_test.max()], [y_test.min(), y_test.max()], 'k--', lw=2)
plt.show()
```






## 决策树可视化 🌲





决策树的强大之处在于其工作逻辑类似于人类决策。我们可以深入模型内部，查看计算机是如何通过一系列“如果...那么...”的规则得出结论的。




为了实现可视化，我们需要安装`graphviz`和`pydotplus`库。请确保按照文档说明安装所有依赖项。


以下是可视化决策树的步骤：





1.  导入必要的库：`StringIO`用于将文件转储为字符串，`IPython.display.Image`用于在Jupyter笔记本中显示图像，`sklearn.tree.export_graphviz`用于导出决策树模型。
2.  创建一个`StringIO`对象来存储图形数据。
3.  使用`export_graphviz`函数将决策树模型导出到该字符串对象中。参数`filled=True`会用颜色填充节点，颜色代表该节点中的多数类别。
4.  使用`pydotplus`解析这个字符串并生成图形对象。
5.  将图形保存为PNG图片文件，并使用`Image`显示它。



我们可以对比未剪枝的树和经过剪枝的树。通过设置较小的`max_depth`（例如3），可以得到一个清晰易懂的小型决策树图，帮助我们理解模型是如何做出每一步分割的。
























```python
from sklearn.tree import export_graphviz
import pydotplus
from IPython.display import Image
from io import StringIO
# 1. 创建一个StringIO对象
dot_data = StringIO()
# 2. 导出决策树到dot_data
# 假设 `dt_model` 是您训练好的决策树模型（例如 best_estimator）
# 假设 `feature_columns` 是特征名称列表
export_graphviz(dt_model, out_file=dot_data,
                filled=True, rounded=True,
                feature_names=feature_columns,
                class_names=['Low Sugar', 'High Sugar'], # 对于分类问题
                special_characters=True)
# 3. 使用pydotplus生成图形
graph = pydotplus.graph_from_dot_data(dot_data.getvalue())
# 4. 保存并显示图形
graph.write_png('decision_tree.png')
Image(filename='decision_tree.png')
```





## 总结 🎓



本节课中我们一起学习了决策树在回归任务中的应用。我们完成了从数据准备、使用网格搜索进行超参数调优、用均方误差评估模型性能，到绘制预测结果散点图的全过程。最后，我们还探索了如何将决策树模型可视化，这有助于我们理解模型的内在逻辑和决策路径。决策树的这种可解释性是其相对于许多“黑箱”模型的一大优势。鼓励大家尝试改变`max_depth`等参数，并观察可视化结果的变化，以加深理解。







![](img/93f83eff173f0af9c4c615eee22f1dfa_256.png)

下节课再见！