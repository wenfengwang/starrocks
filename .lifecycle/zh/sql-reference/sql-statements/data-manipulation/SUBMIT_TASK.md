---
displayed_sidebar: English
---

# 提交任务

## 描述

提交一个 ETL 语句作为异步任务。该功能自 StarRocks v2.5 起得到支持。

StarRocks v3.0支持提交异步任务用于[CREATE TABLE AS SELECT](../data-definition/CREATE_TABLE_AS_SELECT.md)和[INSERT](./INSERT.md)操作。

您可以使用 [DROP TASK](./DROP_TASK.md) 命令来删除一个异步任务。

## 语法

```SQL
SUBMIT TASK [task_name] AS <etl_statement>
```

## 参数

|参数|说明|
|---|---|
|task_name|任务名称。|
|etl_statement|要作为异步任务提交的 ETL 语句。 StarRocks 目前支持提交 CREATE TABLE AS SELECT 和 INSERT 异步任务。|

## 使用说明

此语句用于创建一个 Task，它是用来存储执行 ETL 语句任务的模板。您可以通过查询信息模式（Information Schema）中的元数据视图 [`tasks` in Information Schema](../../../reference/information_schema/tasks.md) 来查看 Task 的信息。

```SQL
SELECT * FROM INFORMATION_SCHEMA.tasks;
SELECT * FROM information_schema.tasks WHERE task_name = '<task_name>';
```

执行 Task 之后，将会相应地生成一个 TaskRun。TaskRun 指的是执行 ETL 语句的任务。TaskRun 有以下几种状态：

- PENDING：任务正在等待执行。
- RUNNING：任务正在执行中。
- FAILED：任务执行失败。
- SUCCESS：任务执行成功。

您可以通过查询信息模式中的元数据视图 [`task_runs` in Information Schema](../../../reference/information_schema/task_runs.md) 来检查 TaskRun 的状态。

```SQL
SELECT * FROM INFORMATION_SCHEMA.task_runs;
SELECT * FROM information_schema.task_runs WHERE task_name = '<task_name>';
```

## 通过 FE 配置项进行配置

您可以使用以下 FE 配置项来配置异步 ETL 任务：

|参数|默认值|说明|
|---|---|---|
|task_ttl_second|259200|任务的有效期限。单位：秒。超过有效期的任务将被删除。|
|task_check_interval_second|14400|删除无效任务的时间间隔。单位：秒。|
|task_runs_ttl_second|259200|TaskRun 的有效时间段。单位：秒。超过有效期的TaskRun将被自动删除。此外，处于 FAILED 和 SUCCESS 状态的 TaskRun 也会自动删除。|
|task_runs_concurrency|20|可以并行运行的最大TaskRun数量。|
|task_runs_queue_length|500|等待运行的 TaskRun 的最大数量。如果数量超过默认值，传入的任务将被暂停。|

## 示例

示例 1：提交一个异步任务以创建表格 tbl1，使用 CREATE TABLE tbl1 AS SELECT * FROM src_tbl，并指定任务名称为 etl0：

```SQL
SUBMIT TASK etl0 AS CREATE TABLE tbl1 AS SELECT * FROM src_tbl;
```

示例 2：提交一个异步任务以向表格 tbl2 插入数据，使用 INSERT INTO tbl2 SELECT * FROM src_tbl，并指定任务名称为 etl1：

```SQL
SUBMIT TASK etl1 AS INSERT INTO tbl2 SELECT * FROM src_tbl;
```

示例 3：提交一个异步任务以覆盖表格 tbl3 中的数据，使用 INSERT OVERWRITE tbl3 SELECT * FROM src_tbl：

```SQL
SUBMIT TASK AS INSERT OVERWRITE tbl3 SELECT * FROM src_tbl;
```

示例 4：提交一个异步任务以覆盖 insert_wiki_edit 表中的数据，使用 INSERT OVERWRITE insert_wiki_edit SELECT * FROM source_wiki_edit，并且不指定任务名称，同时通过提示将查询超时时间延长到 100000 秒：

```SQL
SUBMIT /*+set_var(query_timeout=100000)*/ TASK AS
INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```
