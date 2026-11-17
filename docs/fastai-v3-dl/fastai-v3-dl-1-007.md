# 深度学习实践课程：7：神经网络内部机制与多目标模型





## 概述

在本节课中，我们将深入探讨神经网络的内部机制，特别是如何通过梯度累积技术训练更大的模型，以及如何构建能够同时预测多个目标的多目标模型。我们还将学习协同过滤的基本原理，并了解如何从零开始构建一个推荐系统模型。

---

## 梯度累积：突破GPU内存限制 🚀

上一节我们介绍了如何通过数据增强和模型集成来提升竞赛成绩。本节中，我们来看看一个简单但强大的技巧——梯度累积，它允许我们在有限的GPU内存下训练更大的模型。

### 梯度累积原理

梯度累积的核心思想是：我们不需要在每个小批量数据后都更新模型权重，而是可以累积多个小批量的梯度，然后一次性更新。

以下是梯度累积的关键代码实现：

```python
count = 0
for x, y in data_loader:
    loss = calculate_loss(x, y)
    loss.backward()
    count += len(x)
    if count >= effective_batch_size:
        coefficients -= gradients * learning_rate
        coefficients.grad.zero_()
        count = 0
```

### 梯度累积的优势

以下是梯度累积的主要优势：

*   **突破内存限制**：通过使用更小的实际批量大小，减少单次前向传播和反向传播的内存占用。
*   **保持训练动态**：累积梯度后更新权重，模拟了使用更大批量大小的训练效果，避免了因批量大小变化而需要重新调整学习率等问题。
*   **成本效益**：无需购买昂贵的大内存GPU，通过软件技巧即可训练大模型。

### 实践应用

在FastAI中，使用梯度累积非常简单，只需在创建`Learner`时添加`GradientAccumulation`回调，并指定累积步数即可。

---



## 多目标模型：同时预测疾病和品种 🌾

现在，让我们将注意力转向模型的输出层。我们将构建一个能够同时预测水稻疾病类型和品种的多目标模型。

### 数据准备








首先，我们需要创建一个包含两个因变量的数据加载器。使用FastAI的`DataBlock`可以轻松实现：

```python
dblock = DataBlock(
    blocks=(ImageBlock, CategoryBlock, CategoryBlock),
    get_items=get_image_files,
    get_y=[parent_label, get_variety],
    splitter=RandomSplitter(valid_pct=0.2),
    item_tfms=Resize(192),
    batch_tfms=aug_transforms(size=128)
)
```


### 模型架构与损失函数

多目标模型的关键在于其输出层和损失函数的设计。

**输出层**：模型需要输出20个激活值，其中前10个对应10种疾病的概率，后10个对应10个品种的概率。





**损失函数**：我们需要为每个目标分别计算损失，然后求和作为总损失。

以下是自定义损失函数和评估指标的代码：






```python
def disease_loss(preds, targs):
    disease_preds = preds[:, :10]  # 前10列为疾病预测
    disease_targs = targs[0]       # 第一个目标是疾病标签
    return F.cross_entropy(disease_preds, disease_targs)
def variety_loss(preds, targs):
    variety_preds = preds[:, 10:]  # 后10列为品种预测
    variety_targs = targs[1]       # 第二个目标是品种标签
    return F.cross_entropy(variety_preds, variety_targs)

def combined_loss(preds, targs):
    return disease_loss(preds, targs) + variety_loss(preds, targs)
```

### 交叉熵损失详解

对于分类任务，我们使用交叉熵损失。它衡量的是模型预测的概率分布与真实标签的“距离”。


**Softmax函数**：首先将模型的原始输出转换为概率分布。
公式：`softmax(z_i) = exp(z_i) / Σ_j exp(z_j)`


**交叉熵计算**：对于真实类别`k`，交叉熵损失为 `-log(p_k)`，其中`p_k`是模型预测该类别属于`k`的概率。







**多目标训练的优势**：有时，让模型同时学习相关的辅助任务（如预测品种），反而能提升其主要任务（如预测疾病）的性能，因为模型可以学习到更有泛化能力的特征表示。



---





## 协同过滤：从零构建推荐系统 🎬


最后，我们探索深度学习的第四个主要应用领域——协同过滤，它是推荐系统的核心。






### 问题定义


我们使用MovieLens数据集，其中包含用户对电影的评分。我们的目标是预测用户对未观看电影的评分，从而进行推荐。


### 潜在因子模型

协同过滤的核心思想是“矩阵补全”。我们假设存在一些“潜在因子”（如电影风格、用户偏好），用户对电影的评分可以通过用户因子向量和电影因子向量的点积来近似。

**嵌入层（Embedding）**：在深度学习中，我们使用“嵌入层”来学习这些因子。嵌入层本质上是一个查找表，将用户ID或电影ID映射到一个低维向量。




```python
# 用户嵌入矩阵：n_users x n_factors
# 电影嵌入矩阵：n_movies x n_factors
user_factors = torch.randn(n_users, n_factors)
movie_factors = torch.randn(n_movies, n_factors)
```


### 从零构建模型


我们可以从零开始构建一个点积协同过滤模型：

```python
class DotProduct(Module):
    def __init__(self, n_users, n_movies, n_factors):
        self.user_factors = Embedding(n_users, n_factors)
        self.movie_factors = Embedding(n_movies, n_factors)
        
    def forward(self, x):
        users = self.user_factors(x[:,0])  # 查找用户因子
        movies = self.movie_factors(x[:,1]) # 查找电影因子
        return (users * movies).sum(dim=1)  # 点积作为预测评分
```

### 模型改进



基础点积模型可以进一步改进：




1.  **偏置项**：添加用户偏置（反映用户打分严苛程度）和电影偏置（反映电影普遍受欢迎程度）。
2.  **激活函数**：使用Sigmoid函数将输出限制在评分范围内（如0-5分）。
3.  **权重衰减**：在损失函数中添加L2正则化项（权重衰减），防止过拟合。这通过惩罚过大的权重，鼓励模型学习更简单、泛化更好的模式。



---

## 总结

本节课中我们一起学习了三个核心内容：

1.  **梯度累积**：一种通过累积多个小批量的梯度来模拟大批量训练的技术，有效解决了GPU内存不足的问题，是训练大模型的实用技巧。
2.  **多目标模型**：学习了如何构建能够同时预测多个目标的神经网络，深入理解了输出层设计、交叉熵损失函数，并认识到多任务学习有时能提升主任务性能。
3.  **协同过滤**：从零开始探索了推荐系统的基本原理，理解了潜在因子模型和嵌入层的概念，并实践了如何构建、训练以及改进一个协同过滤模型。

![](img/9790e5fe5cae4197004209600cae6f7e_111.png)

这些知识将帮助你更灵活地设计模型架构，优化训练过程，并解决更复杂的实际问题。