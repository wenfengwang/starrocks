---
displayed_sidebar: "Chinese"
---

# 创建PIPE

## 描述

创建新的PIPE以定义INSERT INTO SELECT FROM FILES语句，该语句由系统用于将数据从指定源数据文件加载到目标表中。

## 语法

```SQL
CREATE [OR REPLACE] PIPE [db_name.]<pipe_name> 
[PROPERTIES ("<key>" = "<value>"[, "<key> = <value>" ...])]
AS <INSERT_SQL>
```

## 参数

### db_name

管道所属数据库的唯一名称。

> **注意**
>
> 每个管道都属于特定数据库。如果删除一个管道所属的数据库，该管道将随着数据库一起被删除，即使数据库恢复也无法恢复该管道。

### pipe_name

管道的名称。管道名称必须在创建管道的数据库内是唯一的。

### INSERT_SQL

用于从指定源数据文件加载数据到目标表的INSERT INTO SELECT FROM FILES语句。

有关FILES()表函数的更多信息，请参阅[FILES](../../../sql-reference/sql-functions/table-functions/files.md)。

### PROPERTIES

一组可选参数，指定如何执行管道。格式：“key” = “value”。

| 属性         | 默认值    | 描述                                                         |
| :------------ | :------------ | :----------------------------------------------------------- |
| AUTO_INGEST   | `TRUE`        | 是否启用自动增量数据加载。有效值为：`TRUE`和`FALSE`。如果将此参数设置为`TRUE`，则启用自动增量数据加载。如果将此参数设置为`FALSE`，系统仅加载在作业创建时指定的源数据文件内容，并且将不会加载后续的新文件或已更新的文件内容。对于大容量加载，可以将此参数设置为`FALSE`。 |
| POLL_INTERVAL | `10` (秒) | 自动增量数据加载的轮询间隔。   |
| BATCH_SIZE    | `1GB`         | 要作为批次加载的数据大小。如果参数值中不包括单位，则使用默认单位字节。 |
| BATCH_FILES   | `256`         | 要作为批次加载的源数据文件数量。     |

## 示例

在当前数据库中创建名为`user_behavior_replica`的管道，以将示例数据集`s3://starrocks-datasets/user_behavior_ten_million_rows.parquet`的数据加载到`user_behavior_replica`表中：

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
> 请在上述命令中使用您的凭据替换`AAA`和`BBB`。可以使用任何有效的`aws.s3.access_key`和`aws.s3.secret_key`，因为该对象可被任何AWS经过身份验证的用户读取。

此示例使用基于IAM用户的身份验证方法和具有与StarRocks表相同模式的Parquet文件。有关其他身份验证方法和CREATE PIPE用法的更多信息，请参阅[认证到AWS资源](../../../integrations/authenticate_to_aws_resources.md)和[FILES](../../../sql-reference/sql-functions/table-functions/files.md)。