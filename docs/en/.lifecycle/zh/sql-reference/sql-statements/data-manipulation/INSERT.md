---
displayed_sidebar: English
---

# 插入

## 描述

将数据插入到特定表中，或用数据覆盖特定表。有关应用场景的详细信息，请参阅[使用 INSERT 加载数据](../../../loading/InsertInto.md)。从 v3.2.0 开始，INSERT 支持将数据写入远程存储中的文件。您可以使用 [INSERT INTO FILES() 将 StarRocks 中的数据卸载到远程存储](../../../unloading/unload_using_insert_into_files.md)。

您可以使用 [SUBMIT TASK](./SUBMIT_TASK.md) 提交异步 INSERT 任务。

## 语法

- **数据加载**：

  ```sql
  INSERT { INTO | OVERWRITE } [db_name.]<table_name>
  [ PARTITION (<partition_name> [, ...] ) ]
  [ TEMPORARY PARTITION (<temporary_partition_name> [, ...] ) ]
  [ WITH LABEL <label>]
  [ (<column_name>[, ...]) ]
  { VALUES ( { <expression> | DEFAULT } [, ...] ) | <query> }
  ```

- **数据卸载**：

  ```sql
  INSERT INTO FILES()
  [ WITH LABEL <label> ]
  { VALUES ( { <expression> | DEFAULT } [, ...] ) | <query> }
  ```

## 参数

| **参数** | 描述                                                  |
| ------------- | ------------------------------------------------------------ |
| INTO          | 将数据追加到表中。                                 |
| OVERWRITE     | 用数据覆盖表。                            |
| table_name    | 要将数据加载到其中的表的名称。可以使用表所在的数据库指定它 `db_name.table_name`。 |
| PARTITION    |  要将数据加载到的分区。您可以指定多个分区，这些分区必须用逗号 （，） 分隔。它必须设置为目标表中存在的分区。如果指定此参数，则数据将仅插入到指定的分区中。如果不指定此参数，则数据将插入到所有分区中。 |
| TEMPORARY PARTITION|临时分区的名称，用于加载数据。您可以指定多个临时分区，这些分区必须用逗号 （，） 分隔。|
| label         | 数据库中每个数据加载事务的唯一标识标签。如果未指定，系统会自动为事务生成一个。我们建议您指定交易记录的标签。否则，如果发生连接错误，则无法检查事务状态，并且没有返回结果。您可以通过对账单查看交易状态 `SHOW LOAD WHERE label="label"` 。有关命名标签的限制，请参阅 [系统限制](../../../reference/System_limit.md)。 |
| column_name   | 要在其中加载数据的目标列的名称。它必须设置为目标表中存在的列。无论目标列名称是什么，您指定的目标列将按顺序一对一映射到源表的列。如果未指定目标列，则默认值为目标表中的所有列。如果源表中的指定列在目标列中不存在，则默认值将写入该列，如果指定的列没有默认值，则事务将失败。如果源表的列类型与目标表的列类型不一致，系统将对不匹配的列进行隐式转换。如果转换失败，将返回语法解析错误。 |
| expression    | 为列赋值的表达式。                |
| DEFAULT       | 为列分配默认值。                         |
| query         | 查询语句，其结果将被加载到目标表中。它可以是 StarRocks 支持的任何 SQL 语句。 |
| FILES()       | 表函数 [FILES()](../../sql-functions/table-functions/files.md)。您可以使用此函数将数据卸载到远程存储中。有关详细信息，请参阅 [使用 INSERT INTO FILES() 将数据卸载到远程存储](../../../unloading/unload_using_insert_into_files.md)。 |

## 返回

```Plain
Query OK, 5 rows affected, 2 warnings (0.05 sec)
{'label':'insert_load_test', 'status':'VISIBLE', 'txnId':'1008'}
```

| 返回        | 描述                                                  |
| ------------- | ------------------------------------------------------------ |
| 受影响的行 | 指示加载的行数。 `warnings` 指示筛选掉的行。 |
| label         | 数据库中每个数据加载事务的唯一标识标签。它可以由用户分配，也可以由系统自动分配。 |
| status        | 指示加载的数据是否可见。 `VISIBLE`：数据加载成功且可见。 `COMMITTED`：数据已成功加载，但暂时不可见。 |
| txnId         | 对应于每个 INSERT 事务的 ID 号。      |

## 使用说明

- 对于当前版本，当 StarRocks 执行 INSERT INTO 语句时，如果任何一行数据与目标表格式不匹配（例如字符串过长），默认情况下 INSERT 事务将失败。您可以将会话变量 `enable_insert_strict` 设置为 `false`，以便系统过滤掉与目标表格式不匹配的数据，并继续执行事务。

- 执行 INSERT OVERWRITE 语句后，StarRocks 会为存储原始数据的分区创建临时分区，将数据插入到临时分区中，并将原始分区与临时分区进行交换。所有这些操作都在 Leader FE 节点中执行。因此，如果 Leader FE 节点在执行 INSERT OVERWRITE 语句时崩溃，则整个加载事务将失败，临时分区将被删除。

## 例

以下示例基于表 `test`，其中包含两列 `c1` 和 `c2`。该 `c2` 列的默认值为 DEFAULT。

- 将一行数据导入 `test` 表中。

```SQL
INSERT INTO test VALUES (1, 2);
INSERT INTO test (c1, c2) VALUES (1, 2);
INSERT INTO test (c1, c2) VALUES (1, DEFAULT);
INSERT INTO test (c1) VALUES (1);
```

如果未指定目标列，则默认情况下会按顺序将列加载到目标表中。因此，在上面的示例中，第一个和第二个 SQL 语句的结果是相同的。

如果目标列（无论是否插入数据）使用 DEFAULT 作为值，则该列将使用默认值作为加载的数据。因此，在上面的示例中，第三和第四语句的输出是相同的。

- 一次将多行数据加载到 `test` 表中。

```SQL
INSERT INTO test VALUES (1, 2), (3, 2 + 2);
INSERT INTO test (c1, c2) VALUES (1, 2), (3, 2 * 2);
INSERT INTO test (c1, c2) VALUES (1, DEFAULT), (3, DEFAULT);
INSERT INTO test (c1) VALUES (1), (3);
```

由于表达式的结果是等效的，因此第一个和第二个语句的结果相同。第三和第四个语句的结果是相同的，因为它们都使用默认值。

- 将查询语句结果导入 `test` 表中。

```SQL
INSERT INTO test SELECT * FROM test2;
INSERT INTO test (c1, c2) SELECT * from test2;
```

- 将查询结果导入 `test` 表中，并指定分区和标签。

```SQL
INSERT INTO test PARTITION(p1, p2) WITH LABEL `label1` SELECT * FROM test2;
INSERT INTO test WITH LABEL `label1` (c1, c2) SELECT * from test2;
```

- 用查询结果覆盖 `test` 表，并指定分区和标签。

```SQL
INSERT OVERWRITE test PARTITION(p1, p2) WITH LABEL `label1` SELECT * FROM test3;
INSERT OVERWRITE test WITH LABEL `label1` (c1, c2) SELECT * from test3;
```

以下示例将 AWS S3 存储桶中 Parquet 文件 **parquet/insert_wiki_edit_append.parquet** 中的数据行插入到表 `insert_wiki_edit` 中：

```Plain
INSERT INTO insert_wiki_edit
    SELECT * FROM FILES(
        "path" = "s3://inserttest/parquet/insert_wiki_edit_append.parquet",
        "format" = "parquet",
        "aws.s3.access_key" = "XXXXXXXXXX",
        "aws.s3.secret_key" = "YYYYYYYYYY",
        "aws.s3.region" = "ap-southeast-1"
);
```
