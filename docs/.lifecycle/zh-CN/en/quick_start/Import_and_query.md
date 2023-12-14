---
displayed_sidebar: "中文"
---

# 载入和查询数据

本快速入门教程将逐步引导您将数据加载到您创建的表中（有关更多说明，请参见[创建表](../quick_start/Create_table.md)），并对数据运行查询。

StarRocks支持从丰富的数据源加载数据，包括一些主要的云服务、本地文件或流式数据系统。您可以查看[摄入概述](../loading/Loading_intro.md)以获取更多信息。以下步骤将向您展示如何使用INSERT INTO语句将数据插入StarRocks，并对数据运行查询。

> **注意**
>
> 您可以使用现有的StarRocks实例、数据库、表、用户和您自己的数据完成本教程。但是，为简单起见，我们建议您使用教程提供的架构和数据。

## 步骤一：使用INSERT加载数据

您可以使用INSERT插入额外的数据行。请参阅[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)以获取详细说明。

通过您的MySQL客户端登录到StarRocks，并执行以下语句，将以下数据行插入您已创建的`sr_member`表中。

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

如果载入事务成功，将返回以下消息。

```Plain
查询成功，影响到 6 行 (0.07 秒)
{'label':'insertDemo', 'status':'VISIBLE', 'txnId':'5'}
```

> **注意**
>
> 通过INSERT INTO VALUES方式加载数据仅适用于您需要验证一个小数据集的情况。不建议在大规模测试或生产环境中使用。要将大量数据加载到StarRocks，请参见[摄入概述](../loading/Loading_intro.md)以了解适合您场景的其他选项。

## 步骤二：对数据运行查询

StarRocks兼容SQL-92。

- 运行一个简单的查询以列出表中的所有数据行。

  ```SQL
  SELECT * FROM sr_member;
  ```

  返回的结果如下：

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
  6 行 (0.05 秒)
  ```

- 运行一个带有指定条件的标准查询。

  ```SQL
  SELECT sr_id, name 
  FROM sr_member
  WHERE reg_date <= "2022-03-14";
  ```

  返回的结果如下：

  ```Plain
  +-------+----------+
  | sr_id | name     |
  +-------+----------+
  |     1 | tom      |
  |     3 | maruko   |
  |     2 | johndoe  |
  +-------+----------+
  3 行 (0.01 秒)
  ```

- 对指定分区运行查询。

  ```SQL
  SELECT sr_id, name 
  FROM sr_member 
  PARTITION (p2);
  ```

  返回的结果如下：

  ```Plain
  +-------+---------+
  | sr_id | name    |
  +-------+---------+
  |     3 | maruko  |
  |     2 | johndoe |
  +-------+---------+
  2 行 (0.01 秒)
  ```

## 下一步做什么

要了解更多关于StarRocks数据摄入方法的信息，请参见[摄入概述](../loading/Loading_intro.md)。除了大量内置函数外，StarRocks还支持[Java UDFs](../sql-reference/sql-functions/JAVA_UDF.md)，这使您能够创建适合您业务场景的自定义数据处理函数。

您还可以学习如何：

- 在载入时执行[ETL](../loading/Etl_in_loading.md)。
- 创建一个[外部表](../data_source/External_table.md)以访问外部数据源。
- [分析查询计划](../administration/Query_planning.md)以了解如何优化查询性能。