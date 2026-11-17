# 141：Stacking算法 🧱




在本节课中，我们将要学习最后一种集成方法——Stacking（堆叠）。我们将了解其核心思想、工作原理、实现细节以及需要注意的事项。

## 概述


Stacking是一种集成学习技术，它通过组合多个不同类型的基础学习器（Base Learners）的预测结果，并将这些结果作为新的特征输入到一个最终的元分类器（Meta-Classifier）中，从而产生最终的预测。这种方法不局限于决策树，可以融合多种算法的优势。





## Stacking的核心思想 🧠



上一节我们介绍了基于决策树的集成方法，本节中我们来看看一种更灵活的集成策略。



Stacking的核心思想是：首先使用训练集拟合多个不同的基础学习算法，然后将这些学习器的预测结果（或得分）作为一个新的训练集。这些新特征被称为元特征（Meta-Features）。最后，将这些元特征输入到一个最终的元分类器中，以得出单一的最终预测。



你可以将最后这个聚合步骤，类比为我们之前在其他集成方法中整合不同“投票”的过程。



## Stacking与Bagging的异同



在某种程度上，Stacking与Bagging非常相似，但它不需要进行自助采样（Bootstrap），也不局限于使用决策树作为基础学习器。



相反，我们训练多个**不同的算法**。这可以被看作是在数据集上测试多种不同的假设，因此我们不会被决策树或其他单一模型所需的假设所限制。


这些算法的输出（即预测结果）成为了新的特征，将被输入到最终的聚合任务，即那个最终的分类器中。




## Stacking的工作流程 🔄


以下是Stacking方法的标准工作流程：




1.  **训练基础学习器**：使用原始训练数据训练多个不同的基础模型（例如逻辑回归、支持向量机、随机森林）。
2.  **生成元特征**：使用这些训练好的基础模型对训练集进行预测，将每个模型的预测结果（对于分类可以是类别标签或概率，对于回归是数值）作为新的特征。
3.  **训练元分类器**：将上一步生成的新特征（元特征）作为输入，原始训练集的标签作为输出，训练一个最终的元分类器。
4.  **进行预测**：对于新数据，先让所有基础学习器进行预测，生成元特征，再输入给元分类器得到最终预测。


用伪代码可以简要描述为：










```python
# 第一步：训练基础学习器
base_learners = [LogisticRegression(), SVM(), RandomForest()]
for learner in base_learners:
    learner.fit(X_train, y_train)
# 第二步：生成元特征（新训练集）
X_meta_train = []
for learner in base_learners:
    predictions = learner.predict(X_train)  # 或 predict_proba
    X_meta_train.append(predictions)
X_meta_train = stack_features(X_meta_train) # 将预测结果堆叠成新特征
# 第三步：训练元分类器
meta_classifier = LogisticRegression() # 元分类器也可以是其他模型
meta_classifier.fit(X_meta_train, y_train)
# 第四步：对新样本预测
X_meta_test = generate_meta_features(base_learners, X_test)
final_prediction = meta_classifier.predict(X_meta_test)
```



## 元分类器的选择


那么，这个最终的元分类器可以是什么样子呢？



一个选择是进行**多数投票**（Majority Vote）或**加权投票**（Weighted Vote），就像我们在其他集成方法中所做的那样。




但需要注意的是，最终的元分类器不一定只是一个投票器，它本身可以是一个完整的模型。我们可以在最后一步运行逻辑回归、线性回归、支持向量机或随机森林等，将每个基础学习器的输出作为这个最终学习器的输入。


## 重要注意事项 ⚠️



为了优化元学习步骤的参数，特别是优化每个基础学习器的参数，我们需要谨慎并科学地设计我们的方法。



这意味着我们不仅需要为最终的分类方法准备留出集（Holdout Set）和测试集，为了正确学习每个基础学习器的参数，我们也需要确保为它们准备相应的留出集。我们不能只为最终分类方法准备留出集。

同时，我们需要意识到，这类模型可能会很快变得相当复杂。通常，更高的复杂度意味着我们更有可能过拟合。因此，我们必须注意这一点，并确保所做的任何工作都能在数据之外很好地泛化。重申一遍，我们需要确保有适当的留出集。





## 在Scikit-learn中的实践 🛠️



我们如何在实践中实现Stacking呢？


首先，我们需要导入相应的类。对于投票式的集成，可以使用 `VotingClassifier`。



以下是使用 `VotingClassifier` 的关键步骤：


```python
from sklearn.ensemble import VotingClassifier
# 创建VotingClassifier实例，传入已训练好的基础模型列表
# estimators 应是一个包含（名称，模型实例）的列表
vc = VotingClassifier(estimators=[('lr', logistic_regression_model),
                                   ('svm', svm_model),
                                   ('rf', random_forest_model)],
                       voting='soft') # 指定投票方式
# 拟合模型
vc.fit(X_train, y_train)
# 进行预测
predictions = vc.predict(X_test)
```



另一个需要注意的超参数是投票方式（`voting`）。
*   **硬投票**：直接采用每个基础学习器预测的类别标签，进行多数表决。
*   **软投票**：采用每个基础学习器预测的类别概率，计算平均概率，并将概率最高的类别作为最终预测。这允许那些对预测非常确定的模型拥有更大的权重。



对于回归问题，可以使用 `VotingRegressor`，其用法类似。


此外，Scikit-learn还提供了更灵活的 `StackingClassifier`。它的工作方式类似，但允许我们指定一个除了投票之外的不同最终分类器（元分类器）来得出最终预测。




```python
from sklearn.ensemble import StackingClassifier
# final_estimator 参数指定元分类器，默认为LogisticRegression
stack_clf = StackingClassifier(estimators=[('lr', logistic_regression_model),
                                            ('svm', svm_model),
                                            ('rf', random_forest_model)],
                                final_estimator=RandomForestClassifier())
stack_clf.fit(X_train, y_train)
```



## 本节总结 📝



本节课中我们一起学习了：
1.  **Stacking算法**：一种通过组合异构基础学习器的预测结果，并将其作为新特征训练元分类器的集成方法。
2.  **工作流程**：包括训练基础学习器、生成元特征、训练元分类器及最终预测。
3.  **关键要点**：Stacking比Bagging更灵活，不限于决策树；需要注意使用独立的留出集来训练基础学习器和元分类器以防止过拟合；元分类器可以是投票器，也可以是任何其他机器学习模型。
4.  **实践工具**：介绍了如何使用Scikit-learn中的 `VotingClassifier`、`VotingRegressor` 和更通用的 `StackingClassifier` 来实现Stacking。

## 扩展阅读 📚



XGBoost是另一个非常流行的提升（Boosting）算法，它不属于Scikit-learn库，但拥有自己的独立库。它本质上是梯度提升，但加入了一些并行化处理，这可以大大加快模型拟合的速度。在处理大规模数据或复杂模型时，值得考虑使用XGBoost。



![](img/10a175b3e613e881653c8cfca09f4450_115.png)

好了，关于Stacking算法的理论部分就到这里，接下来我们将进入实践环节，在Notebook中具体实现它。