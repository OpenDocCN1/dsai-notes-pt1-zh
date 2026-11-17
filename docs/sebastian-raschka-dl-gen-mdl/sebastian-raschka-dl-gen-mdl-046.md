# 046：使用PyTorch训练ADALINE 🚀



在本节课中，我们将学习如何使用PyTorch的自动微分功能来训练ADALINE模型。我们将从手动实现梯度计算开始，逐步过渡到使用PyTorch提供的更自动化的方法，最终展示PyTorch的标准训练流程。通过对比不同实现，你将理解自动微分的便利性及其在深度学习中的核心作用。




## 概述与准备工作




首先，我们导入必要的库并准备数据集。我们将使用熟悉的Iris数据集，以便将注意力集中在代码实现上。




```python
import torch
import torch.nn.functional as F
from torch.autograd import grad
```



以下是加载和预处理Iris数据集的代码，这与我们之前课程中的步骤完全相同。


```python
# 假设 load_iris_dataset 是一个返回 (X_train, y_train, X_test, y_test) 的函数
X_train, y_train, X_test, y_test = load_iris_dataset()
# 将数据转换为PyTorch张量
X_train = torch.tensor(X_train, dtype=torch.float32)
y_train = torch.tensor(y_train, dtype=torch.float32).view(-1, 1)
X_test = torch.tensor(X_test, dtype=torch.float32)
y_test = torch.tensor(y_test, dtype=torch.float32).view(-1, 1)
```



## 方法一：手动计算梯度





上一节我们准备好了数据，本节中我们首先回顾上周实现的手动计算梯度的ADALINE版本。这个实现的核心在于我们根据推导出的公式，自己编写代码计算权重和偏置的梯度。

以下是ADALINE模型的手动实现，包括初始化、前向传播和反向传播。





```python
class AdalineManual:
    def __init__(self, num_features):
        self.num_features = num_features
        self.weights = torch.zeros(num_features, 1, requires_grad=False)
        self.bias = torch.zeros(1, requires_grad=False)

    def forward(self, x):
        net_input = torch.matmul(x, self.weights) + self.bias
        activation = net_input  # 恒等激活函数
        return activation

    def backward(self, x, y, y_pred):
        # 手动计算梯度公式: - (y - y_pred) * x
        grad_weights = -torch.matmul(x.t(), (y - y_pred))
        grad_bias = -torch.sum(y - y_pred)
        return grad_weights, grad_bias

    def predict(self, x):
        # 阈值函数用于预测
        net_input = self.forward(x)
        return torch.where(net_input >= 0.0, 1, -1)
```





训练循环负责迭代数据、计算损失和更新参数。以下是训练函数的关键部分。

```python
def train_manual(model, X, y, epochs=20, lr=0.01, batch_size=32):
    cost_list = []
    for epoch in range(epochs):
        # 打乱数据并创建小批量
        indices = torch.randperm(X.size(0))
        X_shuffled = X[indices]
        y_shuffled = y[indices]

        for i in range(0, X.size(0), batch_size):
            X_batch = X_shuffled[i:i+batch_size]
            y_batch = y_shuffled[i:i+batch_size]

            # 前向传播
            y_pred = model.forward(X_batch)
            # 计算损失（均方误差）
            loss = torch.mean((y_batch - y_pred) ** 2)
            cost_list.append(loss.item())

            # 反向传播：手动计算梯度
            grad_w, grad_b = model.backward(X_batch, y_batch, y_pred)
            # 更新参数
            model.weights += -lr * grad_w
            model.bias += -lr * grad_b

        print(f'Epoch: {epoch+1:03d} | Loss: {loss.item():.4f}')
    return cost_list
```



运行手动实现的模型，我们可以看到损失函数随着训练轮次下降。




```python
model_manual = AdalineManual(num_features=X_train.shape[1])
costs_manual = train_manual(model_manual, X_train, y_train)
```




## 方法二：使用 `torch.autograd.grad` 进行自动微分


在手动实现中，我们亲自推导并编码了梯度公式。本节中我们来看看如何利用PyTorch的 `torch.autograd.grad` 函数来自动计算梯度，从而减少错误并提高效率。







这个版本的ADALINE模型结构与手动版本几乎相同，主要区别在于反向传播中梯度的计算方式。

```python
class AdalineAutoGrad:
    def __init__(self, num_features):
        self.num_features = num_features
        self.weights = torch.zeros(num_features, 1, requires_grad=True)
        self.bias = torch.zeros(1, requires_grad=True)

    def forward(self, x):
        net_input = torch.matmul(x, self.weights) + self.bias
        activation = net_input
        return activation

    def predict(self, x):
        net_input = self.forward(x)
        return torch.where(net_input >= 0.0, 1, -1)
```






以下是使用 `grad` 函数进行训练的关键步骤。注意，我们不再需要手动推导梯度公式。

```python
def train_autograd(model, X, y, epochs=20, lr=0.01, batch_size=32):
    cost_list = []
    for epoch in range(epochs):
        indices = torch.randperm(X.size(0))
        X_shuffled = X[indices]
        y_shuffled = y[indices]

        for i in range(0, X.size(0), batch_size):
            X_batch = X_shuffled[i:i+batch_size]
            y_batch = y_shuffled[i:i+batch_size]

            # 前向传播
            y_pred = model.forward(X_batch)
            loss = torch.mean((y_batch - y_pred) ** 2)
            cost_list.append(loss.item())

            # 使用 autograd.grad 自动计算梯度
            grad_w = grad(loss, model.weights, retain_graph=True)[0]
            grad_b = grad(loss, model.bias)[0] # 计算偏置梯度后图会被释放

            # 更新参数（添加负梯度）
            model.weights.data += -lr * grad_w
            model.bias.data += -lr * grad_b

        # 使用 torch.no_grad() 上下文管理器避免在评估时构建计算图
        with torch.no_grad():
            print(f'Epoch: {epoch+1:03d} | Loss: {loss.item():.4f}')
    return cost_list
```



运行这个版本，得到的损失曲线和准确率应与手动版本完全一致，这验证了自动微分计算的正确性。

```python
model_autograd = AdalineAutoGrad(num_features=X_train.shape[1])
costs_autograd = train_autograd(model_autograd, X_train, y_train)
```


## 方法三：使用PyTorch标准API（`backward` 和优化器）

我们已经看到了如何使用 `grad` 函数。本节中我们将探索PyTorch最常用、最自动化的流程：使用 `backward()` 方法进行反向传播，并结合优化器（如SGD）来更新参数。

这个实现利用了PyTorch的 `nn.Linear` 层和 `nn.MSELoss` 损失函数，使得模型定义更加简洁。




```python
import torch.nn as nn

class AdalineTorchAPI(nn.Module):
    def __init__(self, num_features):
        super(AdalineTorchAPI, self).__init__()
        self.linear = nn.Linear(num_features, 1, bias=True)
        # 为了与之前对比，将权重初始化为0
        self.linear.weight.data.zero_()
        self.linear.bias.data.zero_()

    def forward(self, x):
        net_input = self.linear(x)
        # 恒等激活函数
        activations = net_input
        return activations

    def predict(self, x):
        net_input = self.forward(x)
        return torch.where(net_input >= 0.0, 1, -1)
```

以下是标准的PyTorch训练循环，它包含了前向传播、损失计算、梯度清零、反向传播和优化器更新步骤。



```python
def train_torch_api(model, X, y, epochs=20, lr=0.01, batch_size=32):
    cost_list = []
    # 定义损失函数和优化器
    loss_fn = nn.MSELoss()
    optimizer = torch.optim.SGD(model.parameters(), lr=lr)

    for epoch in range(epochs):
        indices = torch.randperm(X.size(0))
        X_shuffled = X[indices]
        y_shuffled = y[indices]

        for i in range(0, X.size(0), batch_size):
            X_batch = X_shuffled[i:i+batch_size]
            y_batch = y_shuffled[i:i+batch_size]

            # 1. 前向传播，计算预测值
            y_pred = model(X_batch)
            # 2. 计算损失
            loss = loss_fn(y_pred, y_batch)
            cost_list.append(loss.item())

            # 3. 清零上一轮的梯度
            optimizer.zero_grad()
            # 4. 反向传播，计算当前梯度
            loss.backward()
            # 5. 优化器更新参数
            optimizer.step()

        with torch.no_grad():
            print(f'Epoch: {epoch+1:03d} | Loss: {loss.item():.4f}')
    return cost_list
```



运行这个最自动化的版本，结果依然与前两种方法相同。





```python
model_torch = AdalineTorchAPI(num_features=X_train.shape[1])
costs_torch = train_torch_api(model_torch, X_train, y_train)
```



**关于优化器如何工作**：当我们通过 `model.parameters()` 将模型的参数（由 `nn.Linear` 等层自动注册）传递给优化器后，调用 `optimizer.step()` 时，优化器会根据 `loss.backward()` 计算并存储在参数 `.grad` 属性中的梯度值，自动执行参数更新。


## 总结






本节课中我们一起学习了使用PyTorch训练ADALINE模型的三种不同方法。





1.  **手动计算梯度**：我们根据数学公式亲自编码计算梯度。这是一个很好的练习，但对于复杂模型容易出错。
2.  **使用 `torch.autograd.grad`**：我们利用PyTorch的自动微分功能计算梯度，无需手动推导公式，提高了准确性和开发效率。
3.  **使用PyTorch标准API**：我们采用了最常见的PyTorch范式，即 `forward` -> 计算 `loss` -> `zero_grad()` -> `loss.backward()` -> `optimizer.step()`。这是最简洁、最自动化且可扩展性最强的方法，适用于训练任何复杂的神经网络。




![](img/e58953bf514898ea379c25164069a08e_135.png)

通过对比，我们发现三种方法得到了完全相同的训练结果，这既验证了我们手动推导的正确性，也证明了PyTorch自动微分机制的可靠性。掌握第三种标准流程是深入学习PyTorch和深度学习的关键。