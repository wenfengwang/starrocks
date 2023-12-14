```md
---
displayed_sidebar: "Chinese"
---

# 提交任务

## 描述

将ETL语句提交为异步任务。自StarRocks v2.5起，支持此功能。

StarRocks v3.0支持为[CREATE TABLE AS SELECT](../data-definition/CREATE_TABLE_AS_SELECT.md)和[INSERT](./INSERT.md)提交异步任务。

您可以使用[DROP TASK](./DROP_TASK.md)删除异步任务。

## 语法

```SQL
SUBMIT TASK [task_name] AS <etl_statement>
```

## 参数

| **参数**      | **描述**                                                  |
| ------------- | --------------------------------------------------------- |
| task_name     | 任务名称。                                                 |
| etl_statement | 您要提交为异步任务的ETL语句。StarRocks目前支持为[CREATE TABLE AS SELECT](../data-definition/CREATE_TABLE_AS_SELECT.md)和[INSERT](./INSERT.md)提交异步任务。 |

## 使用说明

此语句创建一个任务（Task），用于执行ETL语句的任务模板。您可以通过查询元数据视图[`information_schema`中的`tasks`](../../../reference/information_schema/tasks.md)来查看任务的信息。

```SQL
SELECT * FROM INFORMATION_SCHEMA.tasks;
SELECT * FROM information_schema.tasks WHERE task_name = '<task_name>';
```

运行任务后，将相应生成一个TaskRun。TaskRun表示执行ETL语句的任务。TaskRun具有以下状态：

- `PENDING`：任务等待运行。
- `RUNNING`：任务正在运行。
- `FAILED`：任务失败。
- `SUCCESS`：任务成功运行。

您可以通过查询元数据视图[`information_schema`中的`task_runs`](../../../reference/information_schema/task_runs.md)来检查TaskRun的状态。

```SQL
SELECT * FROM INFORMATION_SCHEMA.task_runs;
SELECT * FROM information_schema.task_runs WHERE task_name = '<task_name>';
```

## 通过前端配置项进行配置

您可以使用以下前端配置项来配置异步ETL任务：

| **参数**                  | **默认值**  | **描述**                                                  |
| -------------------------- | ----------- | --------------------------------------------------------- |
| task_ttl_second            | 259200      | 任务有效期。单位：秒。超过有效期的任务将被删除。                   |
| task_check_interval_second | 14400       | 删除无效任务的时间间隔。单位：秒。                                   |
| task_runs_ttl_second       | 259200      | TaskRun有效期。单位：秒。超过有效期的TaskRuns将被自动删除。此外，`FAILED`和`SUCCESS`状态的TaskRun也将被自动删除。 |
| task_runs_concurrency      | 20          | 可并行运行的TaskRuns的最大数量。                                |
| task_runs_queue_length     | 500         | 等待运行的TaskRuns的最大数量。如果数量超过默认值，传入任务将被挂起。          |

## 示例

示例1：提交`CREATE TABLE tbl1 AS SELECT * FROM src_tbl`的异步任务，并将任务名称指定为`etl0`：

```SQL
SUBMIT TASK etl0 AS CREATE TABLE tbl1 AS SELECT * FROM src_tbl;
```

示例2：提交`INSERT INTO tbl2 SELECT * FROM src_tbl`的异步任务，并将任务名称指定为`etl1`：

```SQL
SUBMIT TASK etl1 AS INSERT INTO tbl2 SELECT * FROM src_tbl;
```

示例3：提交`INSERT OVERWRITE tbl3 SELECT * FROM src_tbl`的异步任务：

```SQL
SUBMIT TASK AS INSERT OVERWRITE tbl3 SELECT * FROM src_tbl;
```

示例4：提交`INSERT OVERWRITE insert_wiki_edit SELECT * FROM source_wiki_edit`的异步任务，没有指定任务名称，并使用提示将查询超时延长到`100000`秒：

```SQL
SUBMIT /*+set_var(query_timeout=100000)*/ TASK AS
INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```