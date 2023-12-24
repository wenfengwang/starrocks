---
displayed_sidebar: English
---

# 创建管道

## 描述

创建一个新的管道，用于定义系统加载数据的 INSERT INTO SELECT FROM FILES 语句，从指定的源数据文件到目标表。

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
>
> 每个管道都属于特定的数据库。如果删除管道所属的数据库，管道也将被删除，即使数据库恢复，管道也无法恢复。

### pipe_name

管道的名称。管道名称在创建管道的数据库中必须是唯一的。

### INSERT_SQL

INSERT INTO SELECT FROM FILES 语句，用于将数据从指定的源数据文件加载到目标表。

有关 FILES() 表函数的更多信息，请参见 [FILES](../../../sql-reference/sql-functions/table-functions/files.md)。

### PROPERTIES

一组可选参数，用于指定如何执行管道。格式： `"key" = "value"`。

| 属性      | 默认值 | 描述                                                  |
| :------------ | :------------ | :----------------------------------------------------------- |
| AUTO_INGEST   | `TRUE`        | 是否启用自动增量数据加载。有效值： `TRUE` 和 `FALSE`。如果将此参数设置为 `TRUE`，则启用自动增量数据加载。如果将此参数设置为 `FALSE` ，系统仅加载创建作业时指定的源数据文件内容，不会加载后续新增或更新的文件内容。对于大容量加载，可以将此参数设置为 `FALSE`。 |
| POLL_INTERVAL | `10` （秒） | 自动增量数据加载的轮询间隔。   |
| BATCH_SIZE    | `1GB`         | 要作为批处理加载的数据大小。如果参数值中未包含单位，则使用默认单位字节。 |
| BATCH_FILES   | `256`         | 要作为批处理加载的源数据文件数。     |

## 例子

在当前数据库中创建一个名为 `user_behavior_replica` 的管道，将示例数据集 `s3://starrocks-datasets/user_behavior_ten_million_rows.parquet` 的数据加载到表 `user_behavior_replica` 中：

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
>
> 在上述命令中用你的凭证替换 `AAA` 和 `BBB`。任何有效的 `aws.s3.access_key` 和 `aws.s3.secret_key` 都可以使用，因为任何经过 AWS 认证的用户都可以读取该对象。

此示例使用了基于 IAM 用户的身份验证方法和具有与 StarRocks 表相同模式的 Parquet 文件。有关其他身份验证方法和 CREATE PIPE 用法的更多信息，请参见 [对 AWS 资源进行身份验证](../../../integrations/authenticate_to_aws_resources.md) 和 [FILES](../../../sql-reference/sql-functions/table-functions/files.md)。
