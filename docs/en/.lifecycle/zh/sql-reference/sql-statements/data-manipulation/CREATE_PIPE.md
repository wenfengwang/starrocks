---
displayed_sidebar: English
---

# CREATE PIPE

## 描述

创建一个新的管道，用于定义系统使用的 INSERT INTO SELECT FROM FILES 语句，将数据从指定的源数据文件加载到目标表。

## 语法

```SQL
CREATE [OR REPLACE] PIPE [db_name.]<pipe_name> 
[PROPERTIES ("<key>" = "<value>"[, "<key>" = "<value>" ...])]
AS <INSERT_SQL>
```

## 参数

### db_name

管道所属的数据库的唯一名称。

> **注意**
> 每个管道都属于特定的数据库。如果删除了管道所属的数据库，管道也会随之被删除，并且即使数据库被恢复，管道也无法恢复。

### pipe_name

管道的名称。在创建管道的数据库中，管道名称必须唯一。

### INSERT_SQL

用于将数据从指定的源数据文件加载到目标表的 INSERT INTO SELECT FROM FILES 语句。

有关 FILES() 表函数的更多信息，请参见 [FILES](../../../sql-reference/sql-functions/table-functions/files.md)。

### PROPERTIES

一组可选参数，用于指定执行管道的方式。格式：`"<key>" = "<value>"`。

|属性|默认值|描述|
|---|---|---|
|AUTO_INGEST|`TRUE`|是否启用自动增量数据加载。有效值：`TRUE` 和 `FALSE`。如果此参数设置为 `TRUE`，则启用自动增量数据加载。如果设置为 `FALSE`，系统只会加载作业创建时指定的源数据文件内容，之后新增或更新的文件内容不会被加载。对于批量加载，可以将此参数设置为 `FALSE`。|
|POLL_INTERVAL|`10`（秒）|自动增量数据加载的轮询间隔。|
|BATCH_SIZE|`1GB`|作为批次加载的数据大小。如果参数值未包含单位，默认单位为字节。|
|BATCH_FILES|`256`|作为批次加载的源数据文件数量。|

## 示例

在当前数据库中创建一个名为 `user_behavior_replica` 的管道，用于将样本数据集 `s3://starrocks-datasets/user_behavior_ten_million_rows.parquet` 的数据加载到 `user_behavior_replica` 表：

```SQL
CREATE PIPE user_behavior_replica
PROPERTIES
(
    "AUTO_INGEST" = "TRUE"
)
AS
INSERT INTO user_behavior_replica
SELECT * FROM FILES
(
    "path" = "s3://starrocks-datasets/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "aws.s3.region" = "us-east-1",
    "aws.s3.access_key" = "AAAAAAAAAAAAAAAAAAAA",
    "aws.s3.secret_key" = "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB"
); 
```

> **注意**
> 在上述命令中，将 `AAA` 和 `BBB` 替换为您的凭证。任何有效的 `aws.s3.access_key` 和 `aws.s3.secret_key` 都可以使用，因为该对象对所有经过 AWS 认证的用户都是可读的。

此示例使用基于 IAM 用户的认证方法和一个与 StarRocks 表结构相同的 Parquet 文件。有关其他认证方法和 CREATE PIPE 使用的更多信息，请参见 [Authenticate to AWS resources](../../../integrations/authenticate_to_aws_resources.md) 和 [FILES](../../../sql-reference/sql-functions/table-functions/files.md)。