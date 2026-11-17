# 18：使用 scikit-learn 第二部分


在本节课中，我们将要学习如何处理文本数据、进行模型评估与参数调优、深入了解线性模型与决策树，并简要介绍异常检测与高级聚类方法。

---

## 文本数据处理 📝

上一节我们介绍了分类数据和连续型特征。本节中我们来看看如何处理长文本数据。


文本数据（如电子邮件、产品评论）与数值数据有很大不同。文本是字符串，长度不一，且长度本身通常不直接反映内容。我们需要一种方法将字符串数据转换为数值表示。

### 词袋模型

基本思想是将文本分解为单词（或标记），然后统计每个单词在文档中出现的频率。

以下是实现步骤：
1.  **分词**：将文本按空格分割并转换为小写，得到标准化单词（标记）。
2.  **构建词汇表**：收集训练数据中所有独特的单词。
3.  **编码**：对于每个文档，创建一个长度等于词汇表大小的向量，记录每个单词的出现次数。


由于向量中大部分元素为零，我们使用稀疏矩阵表示以节省空间。这种方法称为**词袋模型**，因为它忽略了单词的顺序。



在 scikit-learn 中，可以使用 `CountVectorizer` 实现：






```python
from sklearn.feature_extraction.text import CountVectorizer
X = ["Some say the world will end in fire",
     "Some say in ice"]
vectorizer = CountVectorizer()
X_bow = vectorizer.fit_transform(X)
```


### TF-IDF 编码













词袋模型的一个变体是 **TF-IDF** 编码。它通过降低常见单词的权重来改进表示。












*   **TF** 代表词频。
*   **IDF** 代表逆文档频率，用于惩罚在所有文档中频繁出现的单词。

其核心思想是：一个单词在越多的文档中出现，其重要性就越低。公式可以简化为对计数进行重新缩放和归一化。


```python
from sklearn.feature_extraction.text import TfidfVectorizer
vectorizer = TfidfVectorizer()
X_tfidf = vectorizer.fit_transform(X)
```


### N-gram 特征

为了捕捉单词顺序信息，我们可以使用 **N-gram**。N-gram 是文本中连续的 N 个单词序列。


*   **1-gram**：单个单词。
*   **2-gram**：相邻的单词对。
*   **3-gram**：相邻的三个单词。


使用 `CountVectorizer` 或 `TfidfVectorizer` 的 `ngram_range` 参数可以轻松提取 N-gram 特征。


```python
# 提取 1-gram 和 2-gram
vectorizer = CountVectorizer(ngram_range=(1, 2))
X_ngram = vectorizer.fit_transform(X)
```

### 字符 N-gram

对于短文本或名称，**字符 N-gram** 可能更有效。它将文本视为字符序列，并提取连续的 N 个字符片段。

```python
vectorizer = CountVectorizer(analyzer='char', ngram_range=(2, 2))
X_char_ngram = vectorizer.fit_transform(X)
```


---





## 文本分类实战：垃圾短信检测 📱


现在，让我们应用所学知识解决一个实际问题：垃圾短信检测。

我们使用一个包含正常短信和垃圾短信的数据集。以下是处理流程：


1.  **加载数据**：数据 `X` 是短信文本列表，`y` 是标签列表。
2.  **划分数据集**：将数据分为训练集和测试集。
3.  **特征提取**：使用 `CountVectorizer` 将文本转换为数值特征。
4.  **训练模型**：由于特征维度可能很高，线性模型（如逻辑回归）通常效果很好。
5.  **评估模型**：在测试集上评估模型性能。

```python
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.feature_extraction.text import CountVectorizer

# 假设 X_text, y 已加载
X_train, X_test, y_train, y_test = train_test_split(X_text, y, stratify=y)
vectorizer = CountVectorizer()
X_train_bow = vectorizer.fit_transform(X_train)
X_test_bow = vectorizer.transform(X_test)

model = LogisticRegression()
model.fit(X_train_bow, y_train)
print("Test accuracy:", model.score(X_test_bow, y_test))
```

我们可以通过检查模型系数（每个单词的权重）来理解哪些单词对预测垃圾短信最重要。


---

## 模型评估与交叉验证 📊


之前我们使用单一的训练/测试分割来评估模型，但这可能不稳定且浪费数据。**K折交叉验证** 是一种更稳健的评估方法。

### K折交叉验证原理

其思想是将数据随机分成 K 个大小相似的子集（称为“折”）。然后进行 K 次训练和评估：
*   第1次：使用折1作为测试集，其余折作为训练集。
*   第2次：使用折2作为测试集，其余折作为训练集。
*   ...
*   第K次：使用折K作为测试集，其余折作为训练集。

最终，我们得到 K 个性能评估分数，可以计算其均值和标准差，从而更可靠地估计模型性能。

在 scikit-learn 中，可以使用 `cross_val_score` 函数轻松实现：

```python
from sklearn.model_selection import cross_val_score
from sklearn.neighbors import KNeighborsClassifier

scores = cross_val_score(KNeighborsClassifier(), X, y, cv=5)
print("CV scores:", scores)
print("Mean CV accuracy:", scores.mean())
```

### 分层交叉验证




对于分类问题，为了确保每个折中各类别的比例与原始数据集一致，应使用**分层K折交叉验证**，这是 `cross_val_score` 在分类任务中的默认行为。

---

## 模型复杂度与参数调优 ⚙️














模型可能**欠拟合**（过于简单）或**过拟合**（过于复杂）。我们需要找到最佳平衡点。






### 验证曲线



验证曲线展示了模型在训练集和验证集上的性能如何随某个超参数（如 KNN 中的邻居数 `n_neighbors`）变化。这有助于我们可视化欠拟合和过拟合的区域。



### 网格搜索


当模型有多个超参数时，我们可以使用 **网格搜索** 来系统地寻找最佳参数组合。`GridSearchCV` 类可以自动进行交叉验证并选择最佳参数。

以下是使用网格搜索为 K近邻分类器寻找最佳 `n_neighbors` 的示例：











```python
from sklearn.model_selection import GridSearchCV
from sklearn.neighbors import KNeighborsClassifier
param_grid = {'n_neighbors': [1, 3, 5, 10, 50]}
grid_search = GridSearchCV(KNeighborsClassifier(), param_grid, cv=5)
grid_search.fit(X_train, y_train)

print("Best parameters:", grid_search.best_params_)
print("Best cross-validation score:", grid_search.best_score_)
print("Test set score:", grid_search.score(X_test, y_test))
```

**重要提示**：使用网格搜索找到最佳参数后，在最终报告模型性能时，应在**一个独立的、未参与参数搜索的测试集**上进行评估，以避免乐观偏差。






---

## 深入线性模型与决策树 🌲



### 线性模型与正则化



线性模型（如线性回归、逻辑回归）通过特征的线性组合进行预测。为了防止过拟合，可以引入正则化项：
*   **岭回归**：使用 L2 正则化，使系数趋向于零但通常不为零。
*   **Lasso回归**：使用 L1 正则化，可以使某些系数**恰好为零**，从而实现特征选择。
*   **弹性网络**：结合 L1 和 L2 正则化。

正则化强度由参数 `alpha`（回归）或 `C`（分类，`C` 是 `alpha` 的倒数）控制。

### 决策树与随机森林










**决策树**通过一系列“是/否”问题（基于特征阈值）对数据进行递归划分。它们非常灵活，但容易过拟合。





**随机森林**是解决过拟合问题的强大方法。它构建许多决策树，每棵树在训练数据的自助采样子集上训练，并且在每个节点分裂时只考虑特征的随机子集。最终的预测是所有树预测的平均（回归）或投票结果（分类）。




随机森林的优点包括：
*   通常具有很高的准确性。
*   对特征缩放不敏感。
*   可以提供特征重要性度量。



```python
from sklearn.ensemble import RandomForestClassifier
rf = RandomForestClassifier(n_estimators=100, max_depth=5)
rf.fit(X_train, y_train)
print("Feature importances:", rf.feature_importances_)
```







---








## 异常检测与高级聚类 🎯


### 异常检测

异常检测旨在识别与大多数数据显著不同的样本。常用方法包括：
*   **孤立森林**：基于随机划分，异常点容易被“孤立”。
*   **一类SVM**：在只有正常数据的情况下，学习一个边界。
*   **局部离群因子**：基于邻居密度的比较。





### 高级聚类方法






除了 K-Means，还有其他聚类算法：
*   **DBSCAN**：基于密度进行聚类，可以发现任意形状的簇，并能识别噪声点。它需要设定邻域半径 `eps` 和最小样本数 `min_samples`。
*   **层次聚类**：通过连续合并或分割簇来构建树状结构（树状图）。有单连接、全连接、Ward 等方法。


---








## 总结





本节课中我们一起学习了：
1.  **文本处理**：使用词袋模型、TF-IDF 和 N-gram 将文本转换为特征。
2.  **模型评估**：使用 K折交叉验证获得更可靠的性能估计。
3.  **参数调优**：使用网格搜索寻找模型最佳超参数。
4.  **核心模型**：深入了解了线性模型的正则化以及决策树/随机森林的工作原理。
5.  **进阶主题**：简要介绍了异常检测的常用方法和 DBSCAN 等高级聚类算法。

![](img/92aed0107e5ed9540c8a768811cdbac0_267.png)

这些工具和方法构成了使用 scikit-learn 进行机器学习实践的核心部分。