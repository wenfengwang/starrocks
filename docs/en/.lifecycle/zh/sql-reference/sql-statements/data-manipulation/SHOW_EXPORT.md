---
displayed_sidebar: English
---

# 显示导出

## 描述

查询满足指定条件的导出作业的执行信息。

## 语法

```SQL
SHOW EXPORT
[ FROM <db_name> ]
[
WHERE
    [ QUERYID = <query_id> ]
    [ STATE = { "PENDING" | "EXPORTING" | "FINISHED" | "CANCELLED" } ]
]
[ ORDER BY <field_name> [ ASC | DESC ] [, ... ] ]
[ LIMIT <count> ]
```

## 参数

此语句可以包含以下可选子句：

- FROM

  指定要查询的数据库的名称。如果不指定 FROM 子句，StarRocks 将查询当前数据库。

- WHERE

  指定要根据哪些条件筛选导出作业。查询的结果集中仅返回满足指定条件的导出作业。

  | **参数** | **必填** | **描述**                                              |
  | ------------- | ------------ | ------------------------------------------------------------ |
  | QUERYID       | 否           | 要查询的导出作业的 ID。该参数用于查询单个导出作业的执行信息。 |
  | STATE         | 否           | 要查询的导出作业的状态。有效值：<ul><li>`PENDING`：指定查询等待调度的导出作业。</li><li>`EXPORTING`：指定查询正在执行的导出作业。</li><li>`FINISHED`：指定查询已成功完成的导出作业。</li><li>`CANCELLED`：指定查询失败的导出作业。</li></ul> |

- ORDER BY

  指定要根据该字段对查询结果集中的导出作业记录进行排序的字段名称。您可以指定多个字段，这些字段必须用逗号（`,`）分隔。此外，还可以使用 `ASC` 或 `DESC` 关键字指定根据指定的字段按升序或降序对导出作业记录进行排序。

- LIMIT

  将查询的结果集限制为指定的最大行数。有效值：正整数。如果不指定 LIMIT 子句，StarRocks 将返回所有满足指定条件的导出作业。

## 返回结果

例如，查询 ID 为 `edee47f0-abe1-11ec-b9d1-00163e1e238f` 的导出作业的执行信息：

```SQL
SHOW EXPORT
WHERE QUERYID = "edee47f0-abe1-11ec-b9d1-00163e1e238f";
```

返回的执行信息如下：

```SQL
     JobId: 14008
   QueryId: edee47f0-abe1-11ec-b9d1-00163e1e238f
     State: FINISHED
  Progress: 100%
  TaskInfo: {"partitions":["*"],"column separator":"\t","columns":["*"],"tablet num":10,"broker":"","coord num":1,"db":"db0","tbl":"tbl_simple","row delimiter":"\n","mem limit":2147483648}
      Path: hdfs://127.0.0.1:9000/users/230320/
CreateTime: 2023-03-20 11:16:14
 StartTime: 2023-03-20 11:16:17
FinishTime: 2023-03-20 11:16:26
   Timeout: 7200
```

返回结果中的参数说明如下：

- `JobId`：导出作业的 ID。
- `QueryId`：查询的 ID。
- `State`：导出作业的状态。

  有效值：

  - `PENDING`：导出作业正在等待调度。
  - `EXPORTING`：正在执行导出作业。
  - `FINISHED`：导出作业已成功完成。
  - `CANCELLED`：导出作业失败。

- `Progress`：导出作业的进度。进度以查询计划的单位进行度量。假设导出作业分为 10 个查询计划，其中 3 个已完成。在这种情况下，导出作业的进度为 30%。有关详细信息，请参阅[“使用 EXPORT > Workflow 导出数据”。](../../../unloading/Export.md#workflow)
- `TaskInfo`：导出作业的信息。

  该信息是一个 JSON 对象，由以下键组成：

  - `partitions`：导出数据所在的分区。如果返回通配符（`*`）作为此键的值，则运行导出作业以从所有分区导出数据。
  - `column separator`：导出的数据文件中使用的列分隔符。
  - `columns`：导出其数据的列的名称。
  - `tablet num`：导出的平板电脑总数。
  - `broker`：在 v2.4 及更早版本中，此字段用于返回导出作业使用的代理的名称。从 v2.5 开始，此字段返回一个空字符串。有关详细信息，请参阅[“使用 EXPORT >背景信息导出数据”。](../../../unloading/Export.md#background-information)
  - `coord num`：导出作业划分为的查询计划数。
  - `db`：导出数据所属的数据库名称。
  - `tbl`：导出数据所属的表名
  - `row delimiter`：导出的数据文件中使用的行分隔符。
  - `mem limit`：导出作业允许的最大内存量。单位：字节。

- `Path`：导出的数据在远程存储上的存储路径。
- `CreateTime`：导出作业的创建时间。
- `StartTime`：导出作业开始调度的时间。
- `FinishTime`：导出作业完成的时间。
- `Timeout`：导出作业所花费的时间比预期多。单位：秒。时间从 `CreateTime` 计算。
- `ErrorMsg`：导出作业引发错误的原因。仅当导出作业遇到错误时，才会返回此字段。

## 例子

- 查询当前数据库中的所有导出作业：

  ```SQL
  SHOW EXPORT;
  ```

- 查询数据库 `example_db` 中 ID 为 `921d8f80-7c9d-11eb-9342-acde48001122` 的导出作业：

  ```SQL
  SHOW EXPORT FROM example_db
  WHERE queryid = "921d8f80-7c9d-11eb-9342-acde48001122";
  ```

- 查询数据库 `example_db` 中处于 `EXPORTING` 状态的导出作业，并指定按 `StartTime` 升序对结果集中的导出作业记录进行排序：

  ```SQL
  SHOW EXPORT FROM example_db
  WHERE STATE = "exporting"
  ORDER BY StartTime ASC;
  ```

- 查询数据库 `example_db` 中的所有导出作业，并指定按 `StartTime` 降序对结果集中的导出作业记录进行排序：

  ```SQL
  SHOW EXPORT FROM example_db
  ORDER BY StartTime DESC;
  ```
