---
displayed_sidebar: English
---

# SHOW EXPORT

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

该语句可以包含以下可选子句：

- FROM

  指定要查询的数据库名称。如果不指定 FROM 子句，StarRocks 会查询当前数据库。

- WHERE

  指定用于过滤导出作业的条件。只有满足指定条件的导出作业才会出现在查询结果集中。

  |**参数**|**必填**|**描述**|
|---|---|---|
  |QUERYID|否|您要查询的导出作业的 ID。此参数用于查询单个导出作业的执行信息。|
  |STATE|否|您要查询的导出作业的状态。有效值：<ul><li>`PENDING`：指定查询等待调度的导出作业。</li><li>`EXPORTING`：指定查询正在执行的导出作业。</li><li>`FINISHED`：指定查询已成功完成的导出作业。</li><li>`CANCELLED`：指定查询已失败的导出作业。</li></ul>|

- ORDER BY

  指定用于对查询结果集中的导出作业记录进行排序的字段名称。您可以指定多个字段，字段之间必须用逗号（`,`）分隔。此外，您可以使用 `ASC` 或 `DESC` 关键字来指定导出作业记录是根据指定字段以升序或降序排序。

- LIMIT

  将查询结果集限制为指定的最大行数。有效值：正整数。如果不指定 LIMIT 子句，StarRocks 将返回满足指定条件的所有导出作业。

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
- `State`：导出作业的状态。有效值包括：
  - `PENDING`：导出作业正在等待被调度。
  - `EXPORTING`：导出作业正在执行中。
  - `FINISHED`：导出作业已成功完成。
  - `CANCELLED`：导出作业已失败。

- `Progress`：导出作业的进度。进度以查询计划的单位来衡量。例如，如果导出作业被分为10个查询计划，其中三个已完成，则导出作业的进度为30%。更多信息请参见 ["使用 EXPORT > 工作流"](../../../unloading/Export.md#workflow)。
- `TaskInfo`：导出作业的信息。
  信息为一个 JSON 对象，包含以下键值：

  - `partitions`：导出数据所在的分区。如果此键的值为通配符（`*`），则表示导出作业从所有分区导出数据。
  - `column separator`：导出数据文件中使用的列分隔符。
  - `columns`：导出的列名。
  - `tablet num`：被导出的 tablet 总数。
  - `broker`：在 v2.4 及更早版本中，此字段用于返回导出作业使用的 broker 的名称。从 v2.5 开始，此字段返回空字符串。更多信息请参见 ["使用 EXPORT 导出数据 > 背景信息"](../../../unloading/Export.md#background-information)。
  - `coord num`：导出作业分为的查询计划数量。
  - `db`：导出数据所属的数据库名称。
  - `tbl`：导出数据所属的表名称。
  - `row delimiter`：导出数据文件中使用的行分隔符。
  - `mem limit`：导出作业允许的最大内存量。单位：字节。

- `Path`：导出数据存储到远程存储的路径。
- `CreateTime`：创建导出作业的时间。
- `StartTime`：导出作业开始调度的时间。
- `FinishTime`：导出作业完成的时间。
- `Timeout`：导出作业所用时间超出预期的时长。单位：秒。时间从 `CreateTime` 开始计算。
- `ErrorMsg`：导出作业出错的原因。仅当导出作业遇到错误时，才返回此字段。

## 示例

- 查询当前数据库中的所有导出作业：

  ```SQL
  SHOW EXPORT;
  ```

- 查询数据库 `example_db` 中 ID 为 `921d8f80-7c9d-11eb-9342-acde48001122` 的导出作业：

  ```SQL
  SHOW EXPORT FROM example_db
  WHERE QUERYID = "921d8f80-7c9d-11eb-9342-acde48001122";
  ```

- 查询数据库 `example_db` 中处于 `EXPORTING` 状态的导出作业，并指定按 `StartTime` 升序对结果集中的导出作业记录进行排序：

  ```SQL
  SHOW EXPORT FROM example_db
  WHERE STATE = "EXPORTING"
  ORDER BY StartTime ASC;
  ```

- 查询数据库 `example_db` 中的所有导出作业，并指定按 `StartTime` 降序对结果集中的导出作业记录进行排序：

  ```SQL
  SHOW EXPORT FROM example_db
  ORDER BY StartTime DESC;
  ```