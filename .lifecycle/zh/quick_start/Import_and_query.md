---
displayed_sidebar: English
---

# 加载和查询数据

本快速入门教程将一步步指导您如何将数据加载进您所创建的表中（更多指导信息，请参见[创建表](../quick_start/Create_table.md)章节），以及如何对数据进行查询。

StarRocks 支持从多种丰富的数据源中加载数据，包括一些主流的云服务、本地文件或流式数据系统。您可以查阅[数据摄取概览](../loading/Loading_intro.md)以获得更多信息。接下来的步骤将向您展示如何使用 INSERT INTO 语句向 StarRocks 插入数据，并对这些数据进行查询。

> **注意**
> 您可以利用现有的 StarRocks 实例、数据库、表、用户及您自己的数据来完成本教程。然而，为了简化操作，我们建议您使用**本教程**提供的**架构**和**数据**。

## 步骤一：使用 INSERT 加载数据

您可以使用 INSERT 语句来插入更多的数据行。详细指导信息，请参见 [INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md) 章节。

通过 MySQL 客户端登录 StarRocks，并执行以下语句，将数据插入您所创建的 sr_member 表中。

```SQL
use sr_hub
INSERT INTO sr_member
WITH LABEL insertDemo
VALUES
    (001,"tom",100000,"2022-03-13",true),
    (002,"johndoe",210000,"2022-03-14",false),
    (003,"maruko",200000,"2022-03-14",true),
    (004,"ronaldo",100000,"2022-03-15",false),
    (005,"pavlov",210000,"2022-03-16",false),
    (006,"mohammed",300000,"2022-03-17",true);
```

如果数据加载成功，您将收到如下信息。

```Plain
Query OK, 6 rows affected (0.07 sec)
{'label':'insertDemo', 'status':'VISIBLE', 'txnId':'5'}
```

> **注意**
> 通过 INSERT INTO VALUES 加载数据仅适合在需要验证小数据集的 DEMO 场景中使用。不推荐在大规模测试或生产环境中使用。要向 StarRocks 加载大量数据，请参阅[数据摄取概览](../loading/Loading_intro.md)，了解更适合您场景的其他选项。

## 步骤二：查询数据

StarRocks 兼容 SQL-92 标准。

- 执行一个简单的查询，列出表中的所有数据行。

  ```SQL
  SELECT * FROM sr_member;
  ```

  查询的返回结果如下：

  ```Plain
  +-------+----------+-----------+------------+----------+
  | sr_id | name     | city_code | reg_date   | verified |
  +-------+----------+-----------+------------+----------+
  |     3 | maruko   |    200000 | 2022-03-14 |        1 |
  |     1 | tom      |    100000 | 2022-03-13 |        1 |
  |     4 | ronaldo  |    100000 | 2022-03-15 |        0 |
  |     6 | mohammed |    300000 | 2022-03-17 |        1 |
  |     5 | pavlov   |    210000 | 2022-03-16 |        0 |
  |     2 | johndoe  |    210000 | 2022-03-14 |        0 |
  +-------+----------+-----------+------------+----------+
  6 rows in set (0.05 sec)
  ```

- 执行一个带有特定条件的标准查询。

  ```SQL
  SELECT sr_id, name 
  FROM sr_member
  WHERE reg_date <= "2022-03-14";
  ```

  查询的返回结果如下：

  ```Plain
  +-------+----------+
  | sr_id | name     |
  +-------+----------+
  |     1 | tom      |
  |     3 | maruko   |
  |     2 | johndoe  |
  +-------+----------+
  3 rows in set (0.01 sec)
  ```

- 对特定分区执行查询。

  ```SQL
  SELECT sr_id, name 
  FROM sr_member 
  PARTITION (p2);
  ```

  查询的返回结果如下：

  ```Plain
  +-------+---------+
  | sr_id | name    |
  +-------+---------+
  |     3 | maruko  |
  |     2 | johndoe |
  +-------+---------+
  2 rows in set (0.01 sec)
  ```

## 接下来可以做什么

要了解更多关于 StarRocks 数据摄取方法，请参阅[数据摄取概览](../loading/Loading_intro.md)。除了大量内置函数，StarRocks 还支持[Java UDFs](../sql-reference/sql-functions/JAVA_UDF.md)，允许您创建适合您业务场景的自定义数据处理函数。

您还可以学习如何：

- 在加载数据时执行[ETL when loading](../loading/Etl_in_loading.md)。
- 创建一个[外部表](../data_source/External_table.md)来访问外部数据源。
- [分析查询计划](../administration/Query_planning.md)，以了解如何优化查询性能。
