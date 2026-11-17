# 072：使用网格搜索优化模型 🧠


在本节课中，我们将学习如何使用网格搜索（Grid Search）来简化并自动化超参数调优过程。我们将把之前学到的K折交叉验证和超参数循环整合到一个简洁的步骤中。






上一节我们介绍了如何使用循环手动测试不同的超参数组合。本节中，我们将使用 `GridSearchCV` 来自动完成这一过程。




## 导入网格搜索工具


首先，我们需要从 `sklearn` 中导入必要的模块。




```python
from sklearn.model_selection import GridSearchCV
```

## 构建模型管道





我们将使用一个管道（Pipeline）来组合数据预处理和模型训练步骤。这个管道包含三个步骤：多项式特征生成、数据标准化和岭回归。








```python
# 假设已经导入了 PolynomialFeatures, StandardScaler, Ridge
from sklearn.preprocessing import PolynomialFeatures, StandardScaler
from sklearn.linear_model import Ridge
from sklearn.pipeline import Pipeline
# 创建管道
pipeline = Pipeline([
    ('poly', PolynomialFeatures()),
    ('scaler', StandardScaler()),
    ('ridge', Ridge())
])
```


## 定义超参数网格


接下来，我们需要定义要搜索的超参数范围。这些是我们手动选择的参数，而不是模型从数据中学习到的。





以下是我们要测试的超参数：
*   **多项式阶数 (`degree`)**：控制模型的复杂度。更高的阶数意味着更复杂的模型。
*   **岭回归正则化强度 (`alpha`)**：控制模型的正则化程度。更高的 `alpha` 值意味着更强的正则化，模型更简单。




我们将测试3个不同的 `degree` 值和30个不同的 `alpha` 值，总共形成90种组合。

```python
# 定义参数网格
param_grid = {
    'poly__degree': [1, 2, 3],          # 测试1阶、2阶、3阶多项式
    'ridge__alpha': [4.0, 10.0, 20.0]   # 测试不同的alpha值，示例中简化了范围
}
```
> **注意**：在参数字典中，键的命名格式为 `[步骤名称]__[参数名]`。这告诉 `GridSearchCV` 将哪个参数分配给管道中的哪个步骤。




## 配置与运行网格搜索




现在，我们初始化 `GridSearchCV` 对象。我们需要传入之前定义的估计器（管道）和参数字典。同时，我们还需要指定交叉验证的策略，例如使用之前创建的 `KFolds` 对象。







```python
# 初始化GridSearchCV对象
grid_search = GridSearchCV(
    estimator=pipeline,      # 我们的模型管道
    param_grid=param_grid,   # 要搜索的参数网格
    cv=5                     # 使用5折交叉验证，也可以传入KFolds对象
)
# 在数据上拟合网格搜索
grid_search.fit(X_train, y_train)
```
拟合过程会遍历所有90种超参数组合，并对每一种进行K折交叉验证，以评估其在验证集上的性能。

## 获取最佳模型与结果





拟合完成后，我们可以轻松地获取最佳超参数组合及其对应的分数。


```python
# 打印最佳分数和最佳参数
print(f"最佳交叉验证分数: {grid_search.best_score_:.2f}")
print(f"最佳参数组合: {grid_search.best_params_}")
```
输出可能类似于：
```
最佳交叉验证分数: 0.85
最佳参数组合: {'poly__degree': 2, 'ridge__alpha': 4.0}
```




## 使用最佳模型进行预测




`GridSearchCV` 的一个关键优势是，在完成交叉验证并确定最佳超参数后，它会用这些最佳超参数在整个训练数据集上重新训练一个最终模型。这个模型可以直接用于预测。






```python
# 使用最佳模型进行预测（在训练集上，用于演示）
y_pred = grid_search.predict(X_train)
# 计算最终模型在训练集上的R²分数（通常表现会很好）
from sklearn.metrics import r2_score
final_score = r2_score(y_train, y_pred)
print(f"最终模型在训练集上的R²分数: {final_score:.2f}")
```




## 深入分析结果




我们可以进一步查看网格搜索的详细结果，例如每一组参数在每一折上的得分。



```python
# 将交叉验证结果转换为DataFrame以便查看
import pandas as pd
results_df = pd.DataFrame(grid_search.cv_results_)
print(results_df[['params', 'mean_test_score', 'std_test_score']].head())
```
我们还可以访问最佳估计器，并提取其内部信息，例如岭回归的系数。






```python
# 获取最佳估计器（即用最佳参数在整个数据集上训练好的管道）
best_estimator = grid_search.best_estimator_
# 从管道中提取岭回归步骤，并查看系数
ridge_model = best_estimator.named_steps['ridge']
print(f"最佳模型的系数: {ridge_model.coef_}")
```

## 课程总结 🎯






本节课中我们一起学习了机器学习工作流中自动化超参数调优的核心工具。



1.  **回顾基础**：我们首先回顾了K折交叉验证的基本原理。
2.  **简化评估**：接着，我们学习了使用 `cross_val_predict` 来简洁地获取模型在每一折上的预测结果和分数。
3.  **自动化调优**：最后，我们重点掌握了如何使用 `GridSearchCV` 来自动化超参数搜索过程。它能够：
    *   遍历我们指定的所有超参数组合（例如，多项式阶数 `[1, 2, 3]` 和正则化参数 `alpha` 的范围）。
    *   对每一种组合执行K折交叉验证，并计算平均性能。
    *   自动选择在验证集上表现最佳的超参数组合。
    *   使用这组最佳超参数，在整个训练数据集上重新训练一个最终模型，该模型已准备好用于对新数据进行预测。



![](img/a5e43002aa9c0fd66e62a6780c2c5320_118.png)

通过 `GridSearchCV`，我们将繁琐的手动循环和评估过程封装成了一个高效、简洁的步骤，这是构建高性能、泛化能力强模型的关键环节。