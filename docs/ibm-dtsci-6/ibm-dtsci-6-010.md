# 010：ALTER、DROP与TRUNCATE操作 🛠️



在本节课中，我们将学习如何使用SQL中的`ALTER TABLE`、`DROP TABLE`和`TRUNCATE TABLE`语句来修改、删除数据库表及其数据。这些操作是管理数据库结构的重要组成部分。

## 概述




`ALTER TABLE`语句用于修改现有表的结构，例如添加、修改或删除列。`DROP TABLE`语句用于删除整个表。`TRUNCATE TABLE`语句用于快速删除表中的所有数据行，但保留表结构。


## ALTER TABLE语句

`ALTER TABLE`语句用于更改现有表的结构。其基本语法如下：

```sql
ALTER TABLE table_name action;
```

其中，`action`可以是添加列、修改列数据类型或删除列等操作。


### 添加列




要向表中添加新列，可以使用`ADD COLUMN`子句。例如，向`author`表添加一个存储电话号码的列：





```sql
ALTER TABLE author ADD COLUMN telephone_number BIGINT;
```

此例中，列的数据类型为`BIGINT`，可容纳长达19位的数字。

### 修改列数据类型

要修改列的数据类型，可以使用`ALTER COLUMN`子句。例如，将`telephone_number`列的数据类型从`BIGINT`改为`VARCHAR(20)`，以允许存储带括号、加号或破折号的电话号码：



```sql
ALTER TABLE author ALTER COLUMN telephone_number SET DATA TYPE VARCHAR(20);
```




需要注意的是，如果列中已有数据与新数据类型不兼容，此操作可能会失败。




### 删除列

如果不再需要某列，可以使用`DROP COLUMN`子句将其删除。例如，删除`author`表中的`telephone_number`列：

```sql
ALTER TABLE author DROP COLUMN telephone_number;
```

## DROP TABLE语句

`DROP TABLE`语句用于从数据库中删除整个表。默认情况下，删除表的同时也会删除表中的所有数据。其语法如下：



```sql
DROP TABLE table_name;
```




例如，删除`author`表：





```sql
DROP TABLE author;
```


## TRUNCATE TABLE语句

有时，您可能只想删除表中的所有数据，而不删除表本身。虽然可以使用不带`WHERE`子句的`DELETE`语句实现，但`TRUNCATE TABLE`通常更快、更高效。其语法如下：

```sql
TRUNCATE TABLE table_name IMMEDIATE;
```

`IMMEDIATE`关键字指定立即处理该语句，且操作不可撤销。例如，清空`author`表中的所有数据：

```sql
TRUNCATE TABLE author IMMEDIATE;
```




## 总结




本节课中，我们一起学习了三个重要的SQL数据定义语句：
*   `ALTER TABLE`语句用于修改现有表的结构，例如添加、修改或删除列。
*   `DROP TABLE`语句用于删除数据库中的现有表。
*   `TRUNCATE TABLE`语句用于删除表中的所有数据行。



![](img/7be8037c8c42418437b420e99cb04f76_46.png)

掌握这些语句将帮助您有效地管理和维护数据库结构。