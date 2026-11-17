# 014：从 SQLite 数据库读取数据解决方案（选修部分 B）📊



在本节课中，我们将学习如何从 SQLite 数据库中读取数据，并使用 Pandas 库进行数据处理。我们将通过一个具体的实验案例，逐步讲解建立数据库连接、执行 SQL 查询以及将结果转换为 Pandas DataFrame 的完整流程。



## 概述



我们将通过解决一个实验问题来学习从 SQLite 数据库读取数据的方法。核心步骤包括导入必要的库、建立数据库连接、编写并执行 SQL 查询，以及使用 Pandas 处理查询结果。



## 导入必要库

首先，我们需要导入两个关键的 Python 库。



以下是需要导入的库：
*   `sqlite3`：用于连接和操作 SQLite 数据库。
*   `pandas`：用于数据处理和分析，我们将使用其 `read_sql` 函数。




```python
import sqlite3
import pandas as pd
```



## 建立数据库连接


上一节我们导入了必要的库，本节中我们来看看如何与数据库建立连接。这是读取数据的第一步。



首先，我们需要指定数据库文件的路径。



```python
path = "database_name.db"
```



接下来，我们使用 `sqlite3` 库的 `connect` 方法来创建一个连接对象。这个连接对象是我们与数据库交互的桥梁。



```python
conn = sqlite3.connect(path)
```



执行上述代码后，`conn` 变量就代表了一个有效的数据库连接。

## 执行简单查询并读取数据

建立了数据库连接后，我们就可以执行 SQL 查询了。本节我们将从一个名为 `allstarfull` 的表中读取所有数据。





我们首先需要编写一个 SQL 查询字符串。



```python
query = "SELECT * FROM allstarfull"
```



然后，我们使用 Pandas 的 `read_sql` 函数来执行这个查询。该函数需要两个参数：查询语句和数据库连接对象。



```python
observations = pd.read_sql(query, conn)
```

为了查看读取的数据，我们可以显示前几行。

```python
observations.head()
```

运行后，我们将看到一个包含 `allstarfull` 表所有数据的 Pandas DataFrame。



## 探索数据库结构





在能够读取特定表的数据之后，了解数据库中有哪些表也很有用。SQLite 数据库有一个名为 `sqlite_master` 的系统表，它存储了数据库的元数据。




以下是从 `sqlite_master` 表读取所有信息的步骤：
1.  编写查询：`SELECT * FROM sqlite_master`。
2.  使用 `pd.read_sql` 函数执行查询。
3.  将结果存储在变量中以便查看。



```python
tables_query = "SELECT * FROM sqlite_master"
tables = pd.read_sql(tables_query, conn)
tables
```


执行后，`tables` DataFrame 将显示数据库中所有表的信息，包括表名、类型和创建语句。


## 执行复杂聚合查询

最后，我们来挑战一个更复杂的查询。本节我们将学习如何编写一个聚合查询，从数据中找出“出场次数最多的前三名球员”，并在出现平局时使用“平均起始位置”作为决胜条件。

我们需要从 `allstarfull` 表中聚合数据。查询逻辑如下：
1.  按 `playerID` 分组。
2.  计算每个球员的总出场次数（`SUM(GP)`）和平均起始位置（`AVG(startingPos)`）。
3.  按总出场次数降序排序（出场多的在前）。
4.  如果出场次数相同，则按平均起始位置升序排序（位置编号小的在前，例如投手为1）。
5.  使用 `LIMIT 3` 只取前三名。


对应的 SQL 查询语句如下：





```sql
best_query = """
SELECT playerID,
       SUM(GP) AS num_games_played,
       AVG(startingPos) AS avg_starting_position
FROM allstarfull
GROUP BY playerID
ORDER BY num_games_played DESC, avg_starting_position ASC
LIMIT 3
"""
```




编写好查询后，我们同样使用 `pd.read_sql` 函数来执行它并获取结果。


```python
best = pd.read_sql(best_query, conn)
best
```




执行后，`best` DataFrame 将清晰地展示出场次数最多的前三名球员及其相关统计数据。


## 总结


![](img/3d637859a16398a499b921fb6aa22bc6_94.png)

本节课中我们一起学习了从 SQLite 数据库读取数据的完整流程。我们首先导入了 `sqlite3` 和 `pandas` 库，然后通过指定路径建立了数据库连接。接着，我们实践了如何执行简单的 `SELECT *` 查询来获取整表数据，也学习了如何查询 `sqlite_master` 系统表来探索数据库结构。最后，我们完成了一个复杂的聚合查询案例，掌握了使用 `GROUP BY`、`ORDER BY` 和 `LIMIT` 等子句来提取和分析特定数据的方法。这些技能是进行数据探索和分析的重要基础。