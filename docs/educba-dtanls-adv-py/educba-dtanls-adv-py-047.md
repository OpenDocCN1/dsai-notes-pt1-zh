# 047：比较与逻辑运算符 🔍




在本节课中，我们将学习NumPy中的比较运算符和逻辑运算符。这些运算符允许我们对数组进行元素级别的比较和逻辑运算，是数据筛选和条件判断的基础。

---



## 比较运算符


在Python中，我们经常使用比较运算符。NumPy也支持这些运算符，并且可以对数组进行元素级别的比较。


首先，我们需要导入NumPy库。


```python
import numpy as np
```


接下来，我们创建两个数组 `a` 和 `b` 来进行比较。



```python
a = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9])
b = np.array([1, 5, 3, 5, 5, 2, 4, 9, 9])
```


现在，我们可以使用 `==` 运算符来比较这两个数组的对应元素是否相等。


```python
print(a == b)
```


执行上述代码会返回一个布尔值数组，其中 `True` 表示对应位置的元素相等，`False` 表示不相等。这个结果数组的数据类型是布尔型。



除了直接使用运算符，NumPy还提供了一个方法 `np.array_equal()` 来检查两个数组是否完全相等。

```python
print(np.array_equal(a, b))
```



如果两个数组的所有元素都相同，则返回 `True`，否则返回 `False`。




---

## 逻辑运算符




上一节我们介绍了比较运算符，本节中我们来看看逻辑运算符。逻辑运算符只能用于布尔数组。


首先，我们定义两个布尔数组。



```python
a_bool = np.array([1, 1, 0, 0], dtype=bool)  # 对应 True, True, False, False
b_bool = np.array([1, 0, 1, 0], dtype=bool)  # 对应 True, False, True, False
```



以下是NumPy中可用的逻辑运算符函数：


*   **`np.logical_and`**：逻辑与。当两个对应元素都为 `True` 时，结果为 `True`。
    ```python
    print(np.logical_and(a_bool, b_bool))
    ```


*   **`np.logical_or`**：逻辑或。当两个对应元素中至少有一个为 `True` 时，结果为 `True`。
    ```python
    print(np.logical_or(a_bool, b_bool))
    ```



*   **`np.logical_xor`**：逻辑异或。当两个对应元素的值不同时，结果为 `True`。
    ```python
    print(np.logical_xor(a_bool, b_bool))
    ```


*   **`np.logical_not`**：逻辑非。对数组中的每个布尔值取反。
    ```python
    print(np.logical_not(a_bool))
    ```


---




## 总结


![](img/e00075feafddefec491466aea5e9f39d_65.png)

本节课中我们一起学习了NumPy中的比较运算符和逻辑运算符。我们掌握了如何使用 `==` 等运算符进行元素级别的数组比较，以及如何使用 `np.logical_and`、`np.logical_or`、`np.logical_xor` 和 `np.logical_not` 函数对布尔数组进行逻辑运算。这些操作是进行数据筛选、条件判断和构建复杂数据操作逻辑的基础。在后续课程中，我们还将学习如何将这些运算符应用于不同形状的数组。