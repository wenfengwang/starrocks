---
displayed_sidebar: English
---

# 提交任务

## 描述

提交一个 ETL 语句作为异步任务。这个功能自 StarRocks v2.5 起就得到支持。

StarRocks v3.0 支持提交 [CREATE TABLE AS SELECT](../data-definition/CREATE_TABLE_AS_SELECT.md) 和 [INSERT](./INSERT.md) 的异步任务。

您可以使用 [DROP TASK](./DROP_TASK.md) 来删除一个异步任务。

## 语法

```SQL
SUBMIT TASK [task_name] AS <etl_statement>
```

## 参数

|**参数**|**描述**|
|---|---|
|task_name|任务名称。|
|etl_statement|您想要作为异步任务提交的 ETL 语句。StarRocks 目前支持为 [CREATE TABLE AS SELECT](../data-definition/CREATE_TABLE_AS_SELECT.md) 和 [INSERT](./INSERT.md) 提交异步任务。|

## 使用说明

此语句创建一个 Task，Task 是用于存储执行 ETL 语句的任务的模板。您可以通过查询 Information Schema 中的元数据视图 [`tasks`](../../../reference/information_schema/tasks.md) 来检查 Task 的信息。

```SQL
SELECT * FROM INFORMATION_SCHEMA.tasks;
SELECT * FROM information_schema.tasks WHERE task_name = '<task_name>';
```

执行 Task 后，将相应生成一个 TaskRun。TaskRun 表示执行 ETL 语句的任务。TaskRun 有以下状态：

- `PENDING`：任务正在等待运行。
- `RUNNING`：任务正在运行中。
- `FAILED`：任务运行失败。
- `SUCCESS`：任务运行成功。

您可以通过查询 Information Schema 中的元数据视图 [`task_runs`](../../../reference/information_schema/task_runs.md) 来检查 TaskRun 的状态。

```SQL
SELECT * FROM INFORMATION_SCHEMA.task_runs;
SELECT * FROM information_schema.task_runs WHERE task_name = '<task_name>';
```

## 通过 FE 配置项进行配置

您可以使用以下 FE 配置项来配置异步 ETL 任务：

|**参数**|**默认值**|**描述**|
|---|---|---|
|task_ttl_second|259200|Task 的有效期限。单位：秒。超过有效期的 Task 将被删除。|
|task_check_interval_second|14400|删除无效 Task 的时间间隔。单位：秒。|
|task_runs_ttl_second|259200|TaskRun 的有效期限。单位：秒。超过有效期的 TaskRun 将被自动删除。此外，处于 `FAILED` 和 `SUCCESS` 状态的 TaskRun 也会自动删除。|
|task_runs_concurrency|20|可以并行运行的 TaskRun 的最大数量。|
|task_runs_queue_length|500|等待运行的 TaskRun 的最大数量。如果数量超过默认值，新来的任务将被挂起。|

## 示例

示例 1：提交一个 `CREATE TABLE tbl1 AS SELECT * FROM src_tbl` 的异步任务，并指定任务名称为 `etl0`：

```SQL
SUBMIT TASK etl0 AS CREATE TABLE tbl1 AS SELECT * FROM src_tbl;
```

示例 2：提交一个 `INSERT INTO tbl2 SELECT * FROM src_tbl` 的异步任务，并指定任务名称为 `etl1`：

```SQL
SUBMIT TASK etl1 AS INSERT INTO tbl2 SELECT * FROM src_tbl;
```

示例 3：提交一个 `INSERT OVERWRITE tbl3 SELECT * FROM src_tbl` 的异步任务：

```SQL
SUBMIT TASK AS INSERT OVERWRITE tbl3 SELECT * FROM src_tbl;
```

示例 4：在不指定任务名称的情况下，提交一个 `INSERT OVERWRITE insert_wiki_edit SELECT * FROM source_wiki_edit` 的异步任务，并使用提示将查询超时设置为 `100000` 秒：

```SQL
SUBMIT /*+set_var(query_timeout=100000)*/ TASK AS
INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```