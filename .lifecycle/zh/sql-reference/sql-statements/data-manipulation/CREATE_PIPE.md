---
displayed_sidebar: English
---

# 创建管道

## 描述

创建一个新的管道，用于定义系统使用的 INSERT INTO SELECT FROM FILES 语句，以将数据从指定的源数据文件加载到目标表中。

## 语法

```SQL
CREATE [OR REPLACE] PIPE [db_name.]<pipe_name> 
[PROPERTIES ("<key>" = "<value>"[, "<key> = <value>" ...])]
AS <INSERT_SQL>
```

## 参数

### db_name

管道所属的数据库的唯一名称。

> **注意**
> 每个**管道**都属于特定的**数据库**。如果删除了**某个管道**所属的**数据库**，那么**管道**也会随之被删除，并且即使**数据库**被恢复，**管道**也无法恢复。

### pipe_name

管道的名称。在创建管道的数据库内，管道名称必须是唯一的。

### INSERT_SQL

使用 INSERT INTO SELECT FROM FILES 语句从指定的源数据文件将数据加载到目标表中。

关于 FILES() 表函数的更多信息，请参见[FILES](../../../sql-reference/sql-functions/table-functions/files.md)。

### 属性

一组可选参数，用于指定执行管道的方式。格式：“键”=“值”。

|属性|默认值|描述|
|---|---|---|
|AUTO_INGEST|TRUE|是否启用自动增量数据加载。有效值：TRUE 和 FALSE。如果将此参数设置为 TRUE，则启用自动增量数据加载。如果设置为FALSE，则系统仅加载作业创建时指定的源数据文件内容，后续新的或更新的文件内容将不会加载。对于批量加载，您可以将此参数设置为 FALSE。|
|POLL_INTERVAL|10（秒）|自动增量数据加载的轮询间隔。|
|BATCH_SIZE|1GB|批量加载的数据大小。如果参数值中不包含单位，则使用默认单位字节。|
|BATCH_FILES|256|要批量加载的源数据文件的数量。|

## 示例

在当前数据库中创建一个名为 user_behavior_replica 的管道，用以将 s3://starrocks-datasets/user_behavior_ten_million_rows.parquet 的样本数据集数据加载到 user_behavior_replica 表中：

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
> 请在上述命令中用您的凭证替换 `AAA` 和 `BBB`。任何有效的 `aws.s3.access_key` 和 `aws.s3.secret_key` 均可使用，因为所有经过 AWS 认证的用户都有权限读取该对象。

此示例采用基于 IAM 用户的认证方式，以及一个与 StarRocks 表结构相同的 Parquet 文件。有关其他认证方法和 CREATE PIPE 使用的更多信息，请参阅 [Authenticate to AWS resources](../../../integrations/authenticate_to_aws_resources.md) 和 [FILES](../../../sql-reference/sql-functions/table-functions/files.md)。
