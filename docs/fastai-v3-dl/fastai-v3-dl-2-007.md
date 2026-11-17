# 深度学习基础到稳定扩散模型：7：反向传播 🔄





在本节课中，我们将要学习神经网络的核心训练算法——反向传播。我们将从一个简单的多层感知机（MLP）开始，逐步推导并实现其前向传播和反向传播过程，最终训练一个能够识别手写数字的模型。

## 概述 📋

反向传播是训练神经网络的关键算法，它通过计算损失函数相对于网络参数的梯度，并使用梯度下降法来更新这些参数。本节课我们将从零开始，手动实现一个多层感知机的反向传播，并理解其背后的数学原理——链式法则。

---



## 神经网络基础回顾 🧠




上一节我们介绍了张量操作，本节中我们来看看神经网络的基本构成。


一个最简单的线性模型可以表示为：`y = w * x + b`。其中，`x`是输入（例如一个像素值），`w`是权重，`b`是偏置，`y`是输出（例如该像素属于数字“3”的概率）。




然而，单个线性模型表达能力有限。为了拟合更复杂的函数，我们可以组合多个“修正线性单元”（ReLU）。ReLU函数定义为：`f(x) = max(0, x)`。通过将多个ReLU“线段”相加，我们可以逼近任意形状的曲线。



当输入是多个像素（例如784个）时，我们就需要处理多维空间中的“平面”。原理是相同的：通过矩阵乘法组合多个输入，然后应用ReLU非线性激活函数，再将结果通过另一个线性层映射到输出空间（例如10个数字类别）。



以下是构建一个简单多层感知机（MLP）所需的步骤：
1.  定义网络结构：输入层、隐藏层（带ReLU激活）、输出层。
2.  初始化权重和偏置。
3.  实现前向传播函数。
4.  定义损失函数（如均方误差MSE或交叉熵损失）。
5.  实现反向传播以计算梯度。
6.  使用梯度下降更新参数。



---

## 实现前向传播 ➡️






首先，我们定义网络参数并实现前向传播过程。



















```python
import torch
import torch.nn.functional as F

# 定义网络尺寸
n, m, c = 50000, 784, 10  # 样本数， 像素数， 类别数
nh = 50  # 隐藏层神经元数量

# 初始化权重和偏置 (随机初始化权重，偏置初始为0)
w1 = torch.randn(m, nh) / m**0.5  # 第一层权重
b1 = torch.zeros(nh)              # 第一层偏置
w2 = torch.randn(nh, 1)           # 第二层权重 (简化版，单输出)
b2 = torch.zeros(1)               # 第二层偏置

# 线性层函数
def lin(x, w, b):
    return x @ w + b
# ReLU激活函数
def relu(x):
    return x.clamp_min(0.)
# 模型前向传播 (简化版，单输出)
def model(xb):
    l1 = lin(xb, w1, b1)  # 第一层线性变换
    l2 = relu(l1)         # ReLU激活
    l3 = lin(l2, w2, b2)  # 第二层线性变换
    return l3
# 计算均方误差损失 (MSE)
def mse(preds, targets):
    # 处理广播问题：确保preds是向量而非列向量
    diff = preds.squeeze() - targets.float()
    return (diff ** 2).mean()
```


我们使用验证集数据测试前向传播，并计算初始的MSE损失。由于权重是随机的，初始损失会很大。




---

## 理解梯度与链式法则 📈





为了优化网络，我们需要计算损失函数相对于每个权重（`w1`, `b1`, `w2`, `b2`）的梯度。梯度是一个多元函数的偏导数向量，它指向函数值增长最快的方向。





对于神经网络这种复合函数，我们使用**链式法则**来计算梯度。链式法则指出，对于函数 `z = f(y)` 和 `y = g(x)`，`z` 关于 `x` 的导数为：
`dz/dx = (dz/dy) * (dy/dx)`



在神经网络中，损失 `L` 是最终输出 `o` 的函数，`o` 又是前一层激活 `a` 的函数，依此类推。因此，计算 `dL/dw1` 需要将路径上所有中间导数相乘：
`dL/dw1 = (dL/do) * (do/da2) * (da2/dz2) * (dz2/da1) * (da1/dz1) * (dz1/dw1)`




这个过程从最终损失开始，向后逐层传播梯度，因此被称为**反向传播**。



一个直观的理解方式是想象一组联动的齿轮。第一个齿轮（输入 `x`）的转动会带动中间齿轮（激活 `a`），最终带动最后一个齿轮（损失 `L`）。链式法则计算的就是 `x` 转动对 `L` 产生的总影响，等于每一级齿轮传动比的乘积。

---

## 手动实现反向传播 ⚙️



我们将为每个操作（线性层、ReLU、MSE损失）实现其反向传播函数，计算并存储梯度。










```python
def mse_grad(preds, targets):
    # MSE损失 L = mean((preds - targets)^2)
    # dL/d(preds) = 2 * (preds - targets) / n
    diff = preds.squeeze() - targets.float()
    return 2. * diff.unsqueeze(-1) / preds.shape[0]  # 梯度形状需与preds匹配
def lin_grad(input, output, weight, bias, output_grad):
    # 线性层 output = input @ weight + bias
    # dL/d(input) = output_grad @ weight.T
    # dL/d(weight) = input.T @ output_grad
    # dL/d(bias) = sum(output_grad, dim=0)
    input_grad = output_grad @ weight.T
    weight_grad = input.T @ output_grad
    bias_grad = output_grad.sum(0)
    return input_grad, weight_grad, bias_grad
def relu_grad(input, output_grad):
    # ReLU层 output = max(0, input)
    # d(output)/d(input) = 1 if input > 0 else 0
    # dL/d(input) = output_grad * (input > 0)
    return output_grad * (input > 0).float()
```







然后，我们按照计算图的反向顺序调用这些函数：








```python
# 前向传播
l1 = lin(xb, w1, b1)
l2 = relu(l1)
l3 = lin(l2, w2, b2)
loss = mse(l3, yb)
# 反向传播
l3_grad = mse_grad(l3, yb)          # 计算损失层梯度
l2_grad, w2_grad, b2_grad = lin_grad(l2, l3, w2, b2, l3_grad) # 计算第二层梯度
l1_grad = relu_grad(l1, l2_grad)    # 计算ReLU层梯度
_, w1_grad, b1_grad = lin_grad(xb, l1, w1, b1, l1_grad) # 计算第一层梯度
```






现在，`w1_grad`, `b1_grad`, `w2_grad`, `b2_grad` 就包含了损失相对于每个参数的梯度。我们可以用PyTorch的自动微分来验证我们手动计算的梯度是否正确。

---



## 模块化重构：创建Layer类 🧩



上面的代码比较冗长且重复。我们可以通过创建通用的 `Module` 类来重构，每个层（线性、ReLU、MSE）都继承自它。`Module` 类负责在正向传播时存储输入和输出，以便在反向传播时使用。











```python
class Module:
    def __call__(self, *args):
        self.args = args          # 存储输入参数
        self.out = self.forward(*args) # 计算并存储输出
        return self.out

    def forward(self): raise NotImplementedError
    def backward(self): raise NotImplementedError
class Relu(Module):
    def forward(self, inp):
        return inp.clamp_min(0.)
    def backward(self):
        inp, = self.args
        self.inp_grad = (self.out > 0).float() * self.out_grad
class Lin(Module):
    def __init__(self, w, b):
        self.w, self.b = w, b
    def forward(self, inp):
        return inp @ self.w + self.b
    def backward(self):
        inp, = self.args
        self.inp_grad = self.out_grad @ self.w.T
        self.w.grad = inp.T @ self.out_grad
        self.b.grad = self.out_grad.sum(0)
class Mse(Module):
    def forward(self, inp, targ):
        diff = inp.squeeze() - targ.float()
        return (diff ** 2).mean()
    def backward(self):
        inp, targ = self.args
        diff = inp.squeeze() - targ.float()
        self.inp_grad = (2. * diff.unsqueeze(-1)) / inp.shape[0]
```




然后，我们的模型可以简洁地定义为一系列层的组合，反向传播只需按逆序调用各层的 `.backward()` 方法。这种设计与PyTorch的 `nn.Module` 设计思想一致。


---






## 使用交叉熵损失改进模型 🎯






之前我们使用了均方误差（MSE）损失，但对于分类问题，交叉熵损失是更标准且效果更好的选择。交叉熵损失与Softmax激活函数通常结合使用。




Softmax函数将神经网络的原始输出（logits）转换为概率分布：
`softmax(x_i) = exp(x_i) / sum(exp(x_j)) for j in all classes`









为了避免数值计算不稳定（指数运算可能产生极大值），我们使用 **Log-Softmax Trick**：
`log_softmax(x_i) = x_i - log(sum(exp(x_j - max(x)))) - max(x)`
这样在计算对数时更加稳定。


对于分类问题，目标标签通常是整数形式（如“3”）。交叉熵损失可以高效地计算为：
`loss = -log(softmax_preds[target])` 的均值。


PyTorch中，`F.cross_entropy` 函数已经集成了 `log_softmax` 和 `negative log likelihood loss` 的计算。



```python
def log_softmax(x):
    # 使用log-sum-exp技巧实现稳定的log_softmax
    m = x.max(-1, keepdim=True).values
    exp_x = (x - m).exp()
    log_sum_exp = exp_x.sum(-1, keepdim=True).log()
    return x - m - log_sum_exp

def cross_entropy_loss(preds, targets):
    # preds: [batch_size, n_classes]
    # targets: [batch_size] (整数标签)
    log_probs = log_softmax(preds)
    # 使用高级索引获取每个目标类别对应的对数概率
    nll = -log_probs[range(targets.shape[0]), targets].mean()
    return nll
```


---



## 实现训练循环 🔁


现在，我们将所有部分组合起来，实现一个完整的训练循环。我们使用小批量梯度下降（Mini-batch SGD）。

以下是训练循环的关键步骤：
1.  将数据划分为小批量（batch）。
2.  对每个批量执行前向传播，计算预测和损失。
3.  执行反向传播，计算梯度。
4.  根据梯度和学习率更新模型参数。
5.  重复多个周期（epoch）。











```python
lr = 0.1  # 学习率
epochs = 20
bs = 64   # 批量大小
for epoch in range(epochs):
    for i in range(0, n, bs):
        # 1. 获取小批量数据
        xb, yb = x_train[i:i+bs], y_train[i:i+bs]

        # 2. 前向传播 (使用改进后的模型，输出10个类别)
        # ... (假设 model() 现在输出 [batch_size, 10])
        preds = model(xb)
        loss = cross_entropy_loss(preds, yb)

        # 3. 反向传播
        loss.backward()  # 如果使用我们的Module类，需要手动调用各层.backward()

        # 4. 更新参数 (梯度下降)
        with torch.no_grad():
            for p in [w1, b1, w2, b2]:
                p -= lr * p.grad
                p.grad.zero_()  # 清零梯度，为下一轮准备

    # 每个epoch后可以计算并打印训练集准确率
    train_acc = (model(x_train).argmax(1) == y_train).float().mean()
    print(f"Epoch {epoch}, Loss: {loss.item():.4f}, Acc: {train_acc:.4f}")
```





经过几个周期的训练，我们的手写数字识别模型在训练集上的准确率可以显著提升。



---







## 总结 🎓



本节课中我们一起学习了神经网络训练的核心——反向传播算法。


我们从一个简单的多层感知机出发，回顾了神经网络的基础。然后，我们深入探讨了梯度的概念和链式法则，这是理解反向传播的数学基础。接着，我们手动实现了线性层、ReLU激活层和损失层的反向传播计算。



为了使代码更清晰和可复用，我们引入了模块化设计，创建了通用的 `Module` 类以及具体的层类。我们还改进了损失函数，用更适合分类任务的交叉熵损失替代了均方误差损失，并解释了其实现细节和数值稳定技巧（Log-Softmax Trick）。









最后，我们将所有组件整合到一个完整的训练循环中，使用小批量梯度下降成功训练了一个手写数字识别模型。



![](img/5149affbe02aaa27d202cb5dce5b7d13_308.png)

通过本节课的学习，你应该对神经网络的内部运作机制，特别是梯度如何从损失函数向后传播并用于更新权重，有了扎实的理解。这是理解更复杂深度学习模型的重要基石。