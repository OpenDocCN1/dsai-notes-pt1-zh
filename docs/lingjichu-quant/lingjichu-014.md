# 014：15.15.4-均线调参实例 📈




在本节课中，我们将学习如何通过调整移动平均线的参数来优化交易策略。我们将使用Python工具遍历不同的参数组合，评估每种组合的表现，从而找到更优的策略配置。

---

上一节我们介绍了双均线策略的基本原理和回测方法。本节中，我们来看看如何通过调整短期和长期移动平均线的参数来优化策略表现。

## 参数空间定义与遍历 🔄


为了找到最佳的参数组合，我们需要定义一个参数空间并进行遍历。以下是定义参数空间和导入必要工具的方法。

首先，我们需要导入用于生成参数组合的工具 `product`。


```python
from itertools import product
```



`product` 函数可以帮助我们生成两个参数空间的所有可能组合。例如，我们可以为短期均线（SMA1）和长期均线（SMA2）分别指定一个取值范围。



```python
# 定义短期均线（SMA1）的参数空间，从20到60，步长为4
sma1_range = range(20, 61, 4)
# 定义长期均线（SMA2）的参数空间，从180到280，步长为10
sma2_range = range(180, 281, 10)
```



接下来，我们使用 `product` 函数生成所有参数组合，并通过循环进行遍历。



```python
# 遍历所有参数组合
for sma1, sma2 in product(sma1_range, sma2_range):
    # 在这里执行策略计算
    pass
```



## 数据准备与策略计算 📊



在遍历每个参数组合时，我们需要重新加载数据并计算相应的指标。以下是数据准备和策略计算的步骤。


首先，重新加载苹果公司的股价数据。




```python
import pandas as pd
# 加载苹果公司股价数据
data = pd.read_csv('apple_stock_price.csv')
```



加载数据后，需要进行数据清洗，例如处理缺失值。


```python
# 去除缺失值
data = data.dropna()
```

接着，计算每日收益率以及指定参数下的短期和长期移动平均线。

```python
# 计算每日收益率
data['returns'] = data['Close'].pct_change()

# 计算短期移动平均线（SMA1）
data['SMA1'] = data['Close'].rolling(window=sma1).mean()

# 计算长期移动平均线（SMA2）
data['SMA2'] = data['Close'].rolling(window=sma2).mean()

# 再次去除因计算移动平均线产生的缺失值
data = data.dropna()
```

然后，根据双均线策略生成交易信号。



```python
# 生成交易信号：当短期均线上穿长期均线时买入（1），下穿时卖出（-1）
data['position'] = 0
data.loc[data['SMA1'] > data['SMA2'], 'position'] = 1
data.loc[data['SMA1'] < data['SMA2'], 'position'] = -1
```



最后，计算策略的累积收益，并与基准（买入并持有）收益进行比较。





```python
# 计算策略的每日收益
data['strategy'] = data['position'].shift(1) * data['returns']

# 计算累积收益
cumulative_returns = (data['returns'] + 1).cumprod()
cumulative_strategy = (data['strategy'] + 1).cumprod()
# 计算策略相对于基准的超额收益
outperformance = cumulative_strategy.iloc[-1] - cumulative_returns.iloc[-1]
```



## 结果收集与展示 📈


我们需要将每次遍历的结果收集起来，以便后续分析和比较。以下是创建结果表格的方法。



首先，初始化一个空的DataFrame来存储结果。



```python
# 初始化结果表格
results = pd.DataFrame()
```


在每次循环中，将当前参数组合及其对应的策略表现添加到结果表格中。



```python
# 在循环内部，构建当前参数组合的结果行
result_row = pd.DataFrame({
    'SMA1': [sma1],
    'SMA2': [sma2],
    'Cumulative_Returns': [cumulative_returns.iloc[-1]],
    'Cumulative_Strategy': [cumulative_strategy.iloc[-1]],
    'Outperformance': [outperformance]
})
# 将当前结果行追加到总结果表格中，忽略索引以保持整洁
results = pd.concat([results, result_row], ignore_index=True)
```


遍历结束后，我们可以查看结果表格，分析不同参数组合的表现。


```python
# 展示前10行结果
print(results.head(10))
```


通过观察 `Outperformance` 列，我们可以判断哪些参数组合使得策略表现优于简单的买入持有策略。正值表示策略表现更好，负值则表示不如基准。



---


![](img/3829f24c07679475a675cd229b786283_90.png)

本节课中我们一起学习了如何通过遍历不同的移动平均线参数来优化双均线交易策略。我们定义了参数空间，使用 `product` 函数进行组合遍历，在每次循环中重新计算策略指标，并收集结果进行比较。这个方法可以帮助我们系统地寻找更有效的策略参数，是量化策略开发中的重要步骤。