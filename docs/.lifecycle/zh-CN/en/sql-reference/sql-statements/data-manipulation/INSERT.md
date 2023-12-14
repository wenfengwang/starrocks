---
displayed_sidebar: "Chinese"
---

# INSERT

## 描述

将数据插入特定表或使用数据覆盖特定表。有关应用场景的详细信息，请参阅[使用INSERT加载数据](../../../loading/InsertInto.md)。从v3.2.0起，INSERT支持将数据写入远程存储中的文件。您可以[使用INSERT INTO FILES()将数据从StarRocks卸载到远程存储](../../../unloading/unload_using_insert_into_files.md)。

您可以使用[SUBMIT TASK](./SUBMIT_TASK.md)提交异步INSERT任务。

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

| **参数**       | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| INTO          | 追加数据到表中。                                            |
| OVERWRITE     | 使用数据覆盖表中原有数据。                                   |
| table_name    | 您要加载数据的表的名称。可以使用`db_name.table_name`指定表所属的数据库。 |
| PARTITION     | 欲加载数据的分区。您可以指定多个用逗号(,)分隔的分区。必须设置为目标表中存在的分区。如果指定了此参数，则数据将仅插入到指定的分区。如果未指定此参数，则数据将插入到所有分区。 |
| TEMPORARY PARTITION|您要将数据加载到的[临时分区](../../../table_design/Temporary_partition.md)的名称。您可以指定多个临时分区，必须用逗号(,)分隔。|
| label         | 数据加载事务在数据库中的唯一标识标签。如果未指定，系统会自动生成一个事务标识。我们建议您为事务指定标签。否则，如果发生连接错误并且未返回结果，则无法检查事务状态。您可以通过`SHOW LOAD WHERE label="label"`语句检查事务状态。有关标签命名的限制，请参阅[系统限制](../../../reference/System_limit.md)。 |
| column_name   | 要加载数据的目标列的名称。必须设置为目标表中存在的列。您指定的目标列会按序与源表的列一一对应。如果未指定目标列，则默认为目标表中的所有列。如果源表中的指定列在目标列中不存在，则将使用默认值写入此列；如果指定列没有默认值，则事务将失败。如果源表的列类型与目标表的类型不一致，则系统会对不匹配的列执行隐式转换。如果转换失败，将返回语法解析错误。 |
| expression    | 分配给列的值的表达式。                                       |
| DEFAULT       | 分配列的默认值。                                            |
| query         | 结果将被加载到目标表的查询语句。可以是StarRocks支持的任何SQL语句。 |
| FILES()       | 表函数 [FILES()](../../sql-functions/table-functions/files.md)。您可以使用此函数将数据卸载到远程存储。有关详细信息，请参阅[使用INSERT INTO FILES()将数据卸载到远程存储](../../../unloading/unload_using_insert_into_files.md)。 |

## 返回

```Plain
Query OK, 5 rows affected, 2 warnings (0.05 sec)
{'label':'insert_load_test', 'status':'VISIBLE', 'txnId':'1008'}
```

| 返回          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| rows affected | 表示加载了多少行数据。`warnings`指示过滤掉的行。                |
| label         | 数据加载事务在数据库中的唯一标识标签。可以由用户分配，也可以由系统自动生成。 |
| status        | 表示加载的数据是否可见。`VISIBLE`：数据已成功加载且可见。`COMMITTED`：数据已成功加载，但暂时不可见。 |
| txnId         | 对应于每个INSERT事务的ID编号。                                 |

## 用法说明

- 就当前版本而言，当StarRocks执行INSERT INTO语句时，如果任何数据行与目标表格式不匹配（例如，字符串过长），默认情况下，INSERT事务将失败。您可以将会话变量`enable_insert_strict`设置为`false`，以便系统过滤掉与目标表格式不匹配的数据并继续执行事务。

- 执行INSERT OVERWRITE语句后，StarRocks会为存储原始数据的分区创建临时分区，将数据插入临时分区，并将原始分区与临时分区进行交换。所有这些操作都在Leader FE节点中执行。因此，如果Leader FE节点在执行INSERT OVERWRITE语句时崩溃，则整个加载事务将失败，并且将删除临时分区。

## 示例

以下示例基于包含两列`c1`和`c2`的表`test`。`c2`列具有DEFAULT默认值。

- 将一行数据导入`test`表。

```SQL
INSERT INTO test VALUES (1, 2);
INSERT INTO test (c1, c2) VALUES (1, 2);
INSERT INTO test (c1, c2) VALUES (1, DEFAULT);
INSERT INTO test (c1) VALUES (1);
```

当未指定目标列时，默认情况下列会按序加载到目标表中。因此，在上述示例中，第一个和第二个SQL语句的结果是相同的。

如果目标列（插入数据或未插入数据）使用了DEFAULT作为值，列将使用默认值作为加载的数据。因此，在上述示例中，第三和第四条语句的输出是相同的。

- 一次将多行数据加载到`test`表中。

```SQL
INSERT INTO test VALUES (1, 2), (3, 2 + 2);
INSERT INTO test (c1, c2) VALUES (1, 2), (3, 2 * 2);
INSERT INTO test (c1, c2) VALUES (1, DEFAULT), (3, DEFAULT);
INSERT INTO test (c1) VALUES (1), (3);
```

由于表达式结果相同，第一条和第二条语句的结果是相同的。第三条和第四条语句的结果是相同的，因为它们都使用了默认值。

- 将查询语句的结果导入`test`表。

```SQL
INSERT INTO test SELECT * FROM test2;
INSERT INTO test (c1, c2) SELECT * from test2;
```

- 将查询结果导入`test`表，并指定分区和标签。

```SQL
INSERT INTO test PARTITION(p1, p2) WITH LABEL `label1` SELECT * FROM test2;
INSERT INTO test WITH LABEL `label1` (c1, c2) SELECT * from test2;
```

- 用查询结果覆盖`test`表，并指定分区和标签。

```SQL
INSERT OVERWRITE test PARTITION(p1, p2) WITH LABEL `label1` SELECT * FROM test3;
INSERT OVERWRITE test WITH LABEL `label1` (c1, c2) SELECT * from test3;
```

以下示例将来自AWS S3存储桶`inserttest`中的Parquet文件**parquet/insert_wiki_edit_append.parquet**的数据行插入表`insert_wiki_edit`中：

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