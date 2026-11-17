# PyTorch 极简实战教程！P10：L10- 数据集转换 🔄

在本节课中，我们将要学习如何在PyTorch中为数据集应用变换。数据变换是数据预处理的关键步骤，它允许我们在数据输入模型之前对其进行标准化、增强或格式转换。


上一节我们介绍了如何使用内置的数据集和数据加载器。本节中我们来看看如何为自定义数据集添加变换功能，并创建自己的变换类。



## 概述：PyTorch中的变换


PyTorch通过`torchvision.transforms`模块提供了大量预定义的变换。这些变换主要分为几类：


以下是主要的变换类别：

*   **图像变换**：例如`CenterCrop`、`Grayscale`、`RandomHorizontalFlip`。
*   **Tensor变换**：例如`LinearTransformation`、`Normalize`。
*   **转换变换**：例如`ToPILImage`、`ToTensor`，用于在不同数据类型间转换。
*   **通用变换**：例如`Lambda`，允许使用自定义函数，或通过继承基类编写自定义变换类。


此外，可以使用`transforms.Compose`将多个变换组合成一个序列，它们将按顺序依次应用。

## 扩展自定义数据集以支持变换


在之前的教程中，我们实现了一个自定义的葡萄酒数据集。现在，让我们扩展这个类，使其能够接收并应用变换参数。




首先，我们需要修改数据集的`__init__`方法，增加一个可选的`transform`参数。

```python
def __init__(self, transform=None):
    # ... 其他初始化代码 ...
    self.transform = transform
```


接着，在`__getitem__`方法中，我们需要在返回数据样本之前应用变换（如果提供了的话）。




```python
def __getitem__(self, index):
    # ... 获取数据和标签的代码 ...
    sample = (features, label)

    if self.transform:
        sample = self.transform(sample)

    return sample
```



这就是为自定义数据集添加变换支持所需的所有更改。


## 创建自定义变换类


一个自定义变换类本质上是一个可调用对象。它需要实现`__call__`方法，该方法接收一个样本并返回变换后的样本。



### 示例1：转换为Tensor的变换


在上一个教程中，我们在数据集中直接将NumPy数组转换为Tensor。现在我们可以将这个步骤移到一个独立的变换类中。

```python
class ToTensor:
    def __call__(self, sample):
        features, label = sample
        return (torch.from_numpy(features),
                torch.from_numpy(label))
```

这个类接收一个`(features, label)`元组，并使用`torch.from_numpy`将两者都转换为PyTorch Tensor。




### 示例2：乘法变换


让我们再创建一个自定义变换，将输入特征乘以一个给定的因子。


```python
class MulTransform:
    def __init__(self, factor):
        self.factor = factor

    def __call__(self, sample):
        features, label = sample
        features *= self.factor
        return (features, label)
```


这个类在初始化时接收一个`factor`参数，并在`__call__`方法中将特征数据乘以这个因子。


## 组合与应用变换



现在，我们可以使用`transforms.Compose`来组合多个变换，并将它们应用到我们的数据集上。


以下是组合与应用变换的步骤：


1.  导入`torchvision.transforms`。
2.  使用`transforms.Compose`创建一个变换序列。
3.  将组合后的变换传递给数据集构造函数。












```python
import torchvision.transforms as transforms
# 组合变换：先转换为Tensor，然后乘以因子2
composed = transforms.Compose([ToTensor(),
                               MulTransform(factor=2)])
# 创建应用了组合变换的数据集
dataset = WineDataset(transform=composed)
# 获取一个样本并检查
first_data = dataset[0]
features, label = first_data
print(features.type())  # 应显示为torch.Tensor
print(features)         # 数值应为原始值的两倍
```


运行上述代码，你将看到特征数据已经变成了`torch.Tensor`类型，并且每个值都被乘以了2。如果将`factor`改为4，则所有特征值将变为原始值的4倍。



## 总结



本节课中我们一起学习了PyTorch数据集变换的核心概念。我们了解了PyTorch内置的变换类型，学会了如何扩展自定义数据集以支持变换参数，并实践了如何创建自己的变换类（如`ToTensor`和`MulTransform`）。最后，我们掌握了使用`transforms.Compose`来组合并顺序应用多个变换的方法。

![](img/57a6f303b8d13bb7d856025a14ce5b5e_75.png)

数据变换是构建机器学习管道的重要环节，它使得数据预处理更加模块化和灵活。在处理图像等复杂数据时，变换（如裁剪、翻转、归一化）尤为重要。建议你查阅官方文档以探索更多可用的变换。