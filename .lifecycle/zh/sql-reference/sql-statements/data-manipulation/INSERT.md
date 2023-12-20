---
displayed_sidebar: English
---

# 插入

## 描述

将数据插入特定表或使用数据覆盖特定表。详细的应用场景，请参考[Load data with INSERT](../../../loading/InsertInto.md)。从v3.2.0版本开始，INSERT支持将数据写入远程存储中的文件。您可以使用[INSERT INTO FILES()](../../../unloading/unload_using_insert_into_files.md)功能，将StarRocks中的数据卸载到远程存储。

您可以使用[SUBMIT TASK](./SUBMIT_TASK.md)命令提交一个异步INSERT任务。

## 语法

- **加载数据**：

  ```sql
  INSERT { INTO | OVERWRITE } [db_name.]<table_name>
  [ PARTITION (<partition_name> [, ...] ) ]
  [ TEMPORARY PARTITION (<temporary_partition_name> [, ...] ) ]
  [ WITH LABEL <label>]
  [ (<column_name>[, ...]) ]
  { VALUES ( { <expression> | DEFAULT } [, ...] ) | <query> }
  ```

- **卸载数据**：

  ```sql
  INSERT INTO FILES()
  [ WITH LABEL <label> ]
  { VALUES ( { <expression> | DEFAULT } [, ...] ) | <query> }
  ```

## 参数

|参数|说明|
|---|---|
|INTO|将数据追加到表中。|
|OVERWRITE|用数据覆盖表。|
|table_name|要向其中加载数据的表的名称。可以用表所在的数据库指定为 db_name.table_name。|
|PARTITION|要将数据加载到的分区。可以指定多个分区，分区之间用逗号（,）分隔。它必须设置为目标表中存在的分区。如果指定此参数，则数据将仅插入到指定的分区中。如果不指定该参数，数据将被插入到所有分区。|
|临时分区|要将数据加载到的临时分区的名称。可以指定多个临时分区，临时分区之间用逗号（,）分隔。|
|标签|数据库中每个数据加载事务的唯一标识标签。如果不指定，系统会自动为该交易生成一个。我们建议您指定交易的标签。否则，如果出现连接错误且没有返回结果，则无法查看事务状态。您可以通过 SHOW LOAD WHERE label="label" 语句检查交易状态。有关命名标签的限制，请参阅系统限制。|
|column_name|要加载数据的目标列的名称。它必须设置为目标表中存在的列。无论目标列名称是什么，您指定的目标列都会按顺序一对一映射到源表的列。如果未指定目标列，则默认值为目标表中的所有列。如果源表中的指定列在目标列中不存在，则默认值将写入该列，如果指定列没有默认值，则事务将失败。如果源表的列类型与目标表的列类型不一致，系统将对不匹配的列进行隐式转换。如果转换失败，将返回语法解析错误。|
|表达式|为列赋值的表达式。|
|DEFAULT|为列指定默认值。|
|query|查询语句，其结果将加载到目标表中。可以是StarRocks支持的任何SQL语句。|
|FILES()|表函数FILES()。您可以使用此功能将数据卸载到远程存储中。有关详细信息，请参阅使用 INSERT INTO FILES() 将数据卸载到远程存储。|

## 返回

```Plain
Query OK, 5 rows affected, 2 warnings (0.05 sec)
{'label':'insert_load_test', 'status':'VISIBLE', 'txnId':'1008'}
```

|返回|说明|
|---|---|
|受影响的行|指示加载的行数。 warnings 表示被过滤掉的行。|
|标签|数据库中每个数据加载事务的唯一标识标签。可以由用户指定，也可以由系统自动指定。|
|status|指示加载的数据是否可见。 VISIBLE：数据已成功加载并可见。 COMMITTED：数据已成功加载，但暂时不可见。|
|txnId|每个 INSERT 事务对应的 ID 号。|

## 使用说明

- 在当前版本中，当StarRocks执行INSERT INTO语句时，如果数据行与目标表格式不匹配（例如，字符串过长），默认情况下INSERT事务将失败。您可以将会话变量enable_insert_strict设置为false，这样系统就会过滤掉不匹配目标表格式的数据，并继续执行事务。

- 执行INSERT OVERWRITE语句后，StarRocks会为存储原始数据的分区创建临时分区，向临时分区插入数据，然后将原分区与临时分区交换。所有这些操作都在Leader FE节点上执行。因此，如果Leader FE节点在执行INSERT OVERWRITE语句期间崩溃，整个加载事务将失败，并且临时分区会被删除。

## 示例

以下示例基于包含两列c1和c2的表test。c2列有一个默认值DEFAULT。

- 向test表中导入一行数据。

```SQL
INSERT INTO test VALUES (1, 2);
INSERT INTO test (c1, c2) VALUES (1, 2);
INSERT INTO test (c1, c2) VALUES (1, DEFAULT);
INSERT INTO test (c1) VALUES (1);
```

当未指定目标列时，默认情况下列将按顺序加载到目标表中。因此，在上述示例中，第一条和第二条SQL语句的结果相同。

如果目标列（无论是否已插入数据）使用DEFAULT作为值，则该列将采用默认值作为加载的数据。因此，在上述示例中，第三条和第四条语句的输出相同。

- 一次性向test表中加载多行数据。

```SQL
INSERT INTO test VALUES (1, 2), (3, 2 + 2);
INSERT INTO test (c1, c2) VALUES (1, 2), (3, 2 * 2);
INSERT INTO test (c1, c2) VALUES (1, DEFAULT), (3, DEFAULT);
INSERT INTO test (c1) VALUES (1), (3);
```

由于表达式结果相同，第一条和第二条语句的结果相同。第三条和第四条语句的结果也相同，因为它们都使用了默认值。

- 将查询语句的结果导入test表。

```SQL
INSERT INTO test SELECT * FROM test2;
INSERT INTO test (c1, c2) SELECT * from test2;
```

- 将查询结果导入test表，并指定分区和标签。

```SQL
INSERT INTO test PARTITION(p1, p2) WITH LABEL `label1` SELECT * FROM test2;
INSERT INTO test WITH LABEL `label1` (c1, c2) SELECT * from test2;
```

- 使用查询结果覆盖test表，并指定分区和标签。

```SQL
INSERT OVERWRITE test PARTITION(p1, p2) WITH LABEL `label1` SELECT * FROM test3;
INSERT OVERWRITE test WITH LABEL `label1` (c1, c2) SELECT * from test3;
```

以下示例插入数据行从Parquet文件**parquet/insert_wiki_edit_append.parquet**到AWS S3存储桶`inserttest`中，插入到表`insert_wiki_edit`中：

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
