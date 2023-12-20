---
displayed_sidebar: English
---

# 显示导出作业

## 描述

查询符合特定条件的导出作业执行信息。

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

此语句可包含以下可选子句：

- FROM

  指定要查询的数据库名称。若未指定FROM子句，StarRocks将查询当前数据库。

- WHERE

  指定过滤导出作业的条件。只有满足指定条件的导出作业才会出现在查询结果集中。

  |参数|必填|说明|
|---|---|---|
  |QUERYID|否|您要查询的导出作业的 ID。该参数用于查询单个导出作业的执行信息。|
  |STATE|否|您要查询的导出作业的状态。有效值：PENDING：指定查询等待调度的导出作业。EXPORING：指定查询正在执行的导出作业。FINISHED：指定查询已成功完成的导出作业。CANCELLED：指定查询正在执行的导出作业。失败了。|

- ORDER BY

  指定根据哪个字段对查询结果集中的导出作业记录进行排序。可以指定多个字段，用逗号（,）分隔。另外，可以使用ASC或DESC关键字来指定根据特定字段将导出作业记录按升序或降序排序。

- LIMIT

  将查询结果集限制为指定的最大行数。有效值为正整数。如未指定LIMIT子句，StarRocks将返回所有满足特定条件的导出作业。

## 返回结果

例如，查询ID为edee47f0-abe1-11ec-b9d1-00163e1e238f的导出作业的执行信息：

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

- JobId：导出作业的ID。
- QueryId：查询的ID。
- State：导出作业的状态。
  有效值：

  - PENDING：导出作业正在等待被调度。
  - EXPORTING：导出作业正在执行中。
  - FINISHED：导出作业已成功完成。
  - CANCELLED：导出作业已失败。

- `Progress`: 导出作业的进度。进度以查询计划的单位来衡量。假设导出作业分为10个查询计划，其中三个已经完成。在这种情况下，导出作业的进度为30%。更多信息请参见["Export data using EXPORT \\u003e Workflow"](../../../unloading/Export.md#workflow)。
- TaskInfo：导出作业的信息。
  信息为一个JSON对象，包含以下键值：

  - partitions：导出数据所在的分区。如果此键的值返回为通配符(*)，则表示导出作业是为了从所有分区导出数据。
  - column_separator：导出数据文件中使用的列分隔符。
  - columns：导出的列名。
  - tablet_num：导出的平板电脑总数。
  - `broker`: 在v2.4及以前版本中，此字段用于返回导出作业使用的broker名称。从v2.5开始，此字段返回空字符串。更多信息请参见 ["Export data using EXPORT \\u003e Background information"](../../../unloading/Export.md#background-information)。
  - coord_num：导出作业被分成的查询计划数量。
  - db：导出数据所属的数据库名称。
  - tbl：导出数据所属的表名称。
  - row_delimiter：导出数据文件中使用的行分隔符。
  - mem_limit：导出作业允许的最大内存量，单位为字节。

- Path：导出数据存储在远端存储上的路径。
- CreateTime：创建导出作业的时间。
- StartTime：开始调度导出作业的时间。
- FinishTime：导出作业完成的时间。
- Timeout：导出作业所用时间超出预期的量，单位为秒。时间从CreateTime开始计算。
- ErrorMsg：导出作业出错的原因。仅当导出作业遇到错误时才返回此字段。

## 示例

- 查询当前数据库中的所有导出作业：

  ```SQL
  SHOW EXPORT;
  ```

- 查询数据库example_db中ID为921d8f80-7c9d-11eb-9342-acde48001122的导出作业：

  ```SQL
  SHOW EXPORT FROM example_db
  WHERE queryid = "921d8f80-7c9d-11eb-9342-acde48001122";
  ```

- 查询数据库example_db中处于EXPORTING状态的导出作业，并指定按StartTime以升序排序结果集中的导出作业记录：

  ```SQL
  SHOW EXPORT FROM example_db
  WHERE STATE = "exporting"
  ORDER BY StartTime ASC;
  ```

- 查询数据库example_db中的所有导出作业，并指定按StartTime以降序排序结果集中的导出作业记录：

  ```SQL
  SHOW EXPORT FROM example_db
  ORDER BY StartTime DESC;
  ```
