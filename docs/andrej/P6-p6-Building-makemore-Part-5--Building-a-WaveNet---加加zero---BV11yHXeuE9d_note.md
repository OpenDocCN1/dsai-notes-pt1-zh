# P6：构建 Makemore 第五部分：构建一个 WaveNet 🧠














在本节课中，我们将继续完善我们最喜爱的字符级语言模型。我们将从之前构建的多层感知器（MLP）架构出发，将其复杂化，以处理更长的输入序列，并构建一个更深、能逐步融合信息的模型。最终，我们将实现一个类似于 **WaveNet** 的层次化架构。








## 概述










在之前的课程中，我们构建了一个基于三层感知器的字符级语言模型。它接收三个先前的字符，并尝试预测序列中的第四个字符。本节课的目标是扩展这个模型，使其能够处理更长的上下文（例如8个字符），并通过一个更深的、树状层次化的结构来逐步融合信息，而不是一次性将所有信息压缩到一个隐藏层中。这种架构灵感来源于2016年的 **WaveNet** 论文，它本质上也是一个自回归模型，用于预测序列中的下一个元素。










## 从现有代码库开始







我们本节课的起点代码与第三部分结束时的代码非常相似。我们有一个处理好的数据集，包含约18.2万个“给定三个字符预测第四个字符”的示例。我们的代码已经模块化，包含了 `Linear`、`BatchNorm` 等层，其API设计模仿了PyTorch的 `torch.nn` 模块。






首先，我们注意到损失曲线因为批次大小太小而波动剧烈。我们通过将损失列表重塑为二维张量并计算行平均来平滑可视化。








```python
# 将损失列表重塑为二维张量以平滑曲线
losses = torch.tensor(losses).view(-1, 1000).mean(1)
```









## 重构模型：引入更多模块









上一节我们处理了数据可视化问题。本节中，我们将重构模型的前向传播逻辑，使其更加模块化和简洁。









我们之前将嵌入表（`C`）和展平（`view`）操作放在了层列表之外。为了使结构更统一，我们创建了 `Embedding` 和 `Flatten` 模块，并将它们加入到层的序列中。



















```python
class Embedding:
    def __init__(self, num_embeddings, embedding_dim):
        self.weight = torch.randn((num_embeddings, embedding_dim))
    def __call__(self, idx):
        return self.weight[idx]
class Flatten:
    def __call__(self, x):
        return x.view(x.shape[0], -1)
```




接着，我们引入 `Sequential` 容器来管理这些层，这进一步简化了代码。`Sequential` 接收一个层列表，并在前向传播中依次调用它们。









```python
class Sequential:
    def __init__(self, layers):
        self.layers = layers
    def __call__(self, x):
        for layer in self.layers:
            x = layer(x)
        return x
```







现在，我们的模型定义和前向传播变得非常清晰：









```python
model = Sequential([
    Embedding(vocab_size, n_embd),
    Flatten(),
    Linear(n_embd * block_size, n_hidden), BatchNorm(n_hidden), Tanh(),
    Linear(n_hidden, vocab_size)
])
logits = model(xb)
```





## 扩展上下文并引入层次化结构



上一节我们重构了模型代码。本节中，我们将扩展模型的上下文长度，并引入WaveNet的核心思想——层次化融合。







首先，我们将输入上下文长度从3增加到8。这立即带来了性能提升（验证损失从 ~2.10 降至 ~2.02），因为模型拥有了更多信息。




但是，简单地将8个字符的嵌入向量展平后送入一个线性层，意味着信息被过早地压缩了。我们想要的是像WaveNet那样，以树状结构逐步融合信息：先融合相邻的两个字符，再融合上一层的两个“组”，以此类推。




为了实现这一点，我们需要修改 `Flatten` 层。我们创建了一个新的 `FlattenConsecutive` 层，它可以将连续的 `n` 个元素拼接在一起，并增加一个“组”的维度。









```python
class FlattenConsecutive:
    def __init__(self, n):
        self.n = n
    def __call__(self, x):
        B, T, C = x.shape
        x = x.view(B, T // self.n, C * self.n)
        if x.shape[1] == 1:
            x = x.squeeze(1)
        return x
```







然后，我们重新设计模型架构。第一层 `FlattenConsecutive(2)` 将8个字符分成4组，每组2个字符的嵌入被拼接。随后的线性层只处理这“2个字符”的信息。之后，我们再次使用 `FlattenConsecutive(2)` 将4组合并为2组，以此类推，形成一个小型的层次化网络。







```python
model = Sequential([
    Embedding(vocab_size, n_embd),
    FlattenConsecutive(2), Linear(n_embd * 2, n_hidden), BatchNorm(n_hidden), Tanh(),
    FlattenConsecutive(2), Linear(n_hidden * 2, n_hidden), BatchNorm(n_hidden), Tanh(),
    FlattenConsecutive(2), Linear(n_hidden * 2, n_hidden), BatchNorm(n_hidden), Tanh(),
    Linear(n_hidden, vocab_size)
])
```








## 修复批归一化层






上一节我们构建了层次化模型。本节中，我们需要修复一个关键问题：`BatchNorm` 层对多维输入的处理。




我们原来的 `BatchNorm` 实现假设输入是二维的 `(batch_size, features)`。但在我们的新架构中，`FlattenConsecutive` 会产生三维输入 `(batch_size, groups, features)`。我们需要让 `BatchNorm` 在训练时，同时计算 `batch` 和 `groups` 维度上的均值和方差。







```python
class BatchNorm:
    def __call__(self, x):
        if self.training:
            dims = (0, 1) if x.ndim == 3 else (0)
            xmean = x.mean(dims, keepdim=True)
            xvar = x.var(dims, keepdim=True)
        # ... 后续标准化和更新运行统计量
```








修复这个Bug后，模型性能得到了小幅但稳定的提升。









## 实验结果与未来方向



通过增加模型容量（如嵌入维度和隐藏层大小），我们最终将验证损失降低到了 **1.993** 左右，成功跨过了2.0的界限。






本节课我们一起实现了一个简化的WaveNet风格架构。我们学习了如何：
1.  使用模块化构建块（如 `Sequential`）来组织复杂网络。
2.  通过 `FlattenConsecutive` 和线性层实现信息的层次化融合。
3.  调整 `BatchNorm` 以正确处理多维输入。








然而，我们实现的只是WaveNet思想的核心骨架。完整的WaveNet还包括**门控激活单元**、**残差连接**和**空洞因果卷积**（用于高效计算）。此外，我们缺乏一个系统的超参数搜索和实验框架，目前的优化更多是“猜测与检验”。







在未来的课程中，我们可以：
*   实现空洞卷积来高效地计算整个输入序列的输出。
*   添加残差连接以训练更深的网络。
*   建立实验管线，进行大规模的超参数优化。
*   探索循环神经网络（RNN/LSTM）和Transformer架构。









**挑战**：你可以尝试调整本课的模型（如各层通道数、嵌入维度），或者阅读WaveNet论文实现更复杂的层，看看能否击败 **1.993** 的验证损失记录。










![](img/c9be2506944533fbac16b5c8db99ea71_467.png)

---
**总结**：本节课中，我们从基础的MLP出发，逐步构建了一个层次化的、类似WaveNet的字符级语言模型。我们重构了代码使其更清晰，引入了层次化信息融合的概念，并修复了批归一化层的多维处理问题。虽然性能得到了提升，但这仅仅是探索现代深度神经网络架构的开始。