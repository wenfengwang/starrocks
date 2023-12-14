---
displayed_sidebar: "Chinese"
---

# 文件外部表

文件外部表是一种特殊类型的外部表。它允许您直接查询外部存储系统中的Parquet和ORC数据文件，而无需将数据加载到StarRocks中。此外，文件外部表不依赖于元存储。在当前版本中，StarRocks支持以下外部存储系统：HDFS、Amazon S3和其他兼容S3的存储系统。

此功能在StarRocks v2.5中得到支持。

## 限制

- 文件外部表必须在[default_catalog](../data_source/catalog/default_catalog.md)中的数据库中创建。您可以运行[SHOW CATALOGS](../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)来查询在集群中创建的目录。
- 仅支持Parquet、ORC、Avro、RCFile和SequenceFile数据文件。
- 您只能使用文件外部表来查询目标数据文件中的数据。不支持像INSERT、DELETE和DROP这样的数据写操作。

## 先决条件

在创建文件外部表之前，您必须配置StarRocks集群，以便StarRocks可以访问存储目标数据文件的外部存储系统。文件外部表所需的配置与Hive目录所需的配置相同，只是您无需配置元存储。有关配置的更多信息，请参阅[Hive目录 - 集成准备](../data_source/catalog/hive_catalog.md#integration-preparations)。

## 创建数据库（可选）

连接到StarRocks集群后，您可以在现有数据库中创建文件外部表，也可以创建新数据库以管理文件外部表。要查询集群中现有的数据库，请运行[SHOW DATABASES](../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)。然后，您可以运行`USE <db_name>`来切换到目标数据库。

创建数据库的语法如下。

```SQL
CREATE DATABASE [IF NOT EXISTS] <db_name>
```

## 创建文件外部表

访问目标数据库后，您可以在该数据库中创建文件外部表。

### 语法

```SQL
CREATE EXTERNAL TABLE <table_name>
(
    <col_name> <col_type> [NULL | NOT NULL] [COMMENT "<comment>"]
) 
ENGINE=file
COMMENT ["comment"]
PROPERTIES
(
    FileLayoutParams,
    StorageCredentialParams
)
```

### 参数

| 参数              | 是否必需 | 描述                                                         |
| ----------------- | -------- | ------------------------------------------------------------ |
| table_name        | 是       | 文件外部表的名称。以下是命名约定：<ul><li>名称可以包含字母、数字（0-9）和下划线（_）。必须以字母开头。</li><li>名称长度不能超过64个字符。</li></ul> |
| col_name          | 是       | 文件外部表中的列名。文件外部表中的列名必须与目标数据文件中的列名相同，但不区分大小写。文件外部表中列的顺序可以与目标数据文件中的列的顺序不同。 |
| col_type          | 是       | 文件外部表中的列类型。您需要根据目标数据文件中的列类型指定此参数。有关更多信息，请参阅[列类型映射](#mapping-of-column-types)。 |
| NULL \| NOT NULL  | 否       | 文件外部表中的列是否允许为空。<ul><li>NULL: 允许为空。</li><li>NOT NULL: 不允许为空。</li></ul>您必须根据以下规则指定此修改器：<ul><li>如果未为目标数据文件中的列指定此参数，则可以选择不为文件外部表中的列指定它，或者为文件外部表中的列指定NULL。</li><li>如果目标数据文件中的列指定为NULL，则可以选择不指定此参数给文件外部表中的列，或者为文件外部表中的列指定NULL。</li><li>如果目标数据文件中的列指定为NOT NULL，则必须还为文件外部表中的列指定NOT NULL。</li></ul> |
| comment           | 否       | 文件外部表中列的注释。                                      |
| ENGINE            | 是       | 引擎类型。将值设置为file。                                    |
| comment           | 否       | 文件外部表的描述。                                          |
| PROPERTIES        | 是       | <ul><li>`FileLayoutParams`:指定目标文件的路径和格式。此属性是必需的。</li><li>`StorageCredentialParams`:指定访问对象存储系统所需的身份验证信息。仅对AWS S3和其他兼容S3的存储系统需要此属性。</li></ul> |

#### FileLayoutParams

关于访问目标数据文件的一组参数。

```SQL
"path" = "<file_path>",
"format" = "<file_format>"
"enable_recursive_listing" = "{ true | false }"
```

| 参数                    | 是否必需 | 描述                                                         |
| ----------------------- | -------- | ------------------------------------------------------------ |
| path                    | 是       | 数据文件的路径。<ul><li>如果数据文件存储在HDFS中，路径格式为`hdfs://<HDFS的IP地址>:<端口>/<路径>`。默认端口号为8020。如果使用默认端口，则无需指定。</li><li>如果数据文件存储在AWS S3或其他兼容S3的存储系统中，路径格式为`s3://<桶名称>/<文件夹>/`。</li></ul>输入路径时请注意以下规则：<ul><li>如果您要访问路径下的所有文件，请以斜杠(`/`)结尾，例如`hdfs://x.x.x.x/user/hive/warehouse/array2d_parq/data/`。当运行查询时，StarRocks将遍历路径下的所有数据文件，不会使用递归遍历数据文件。</li><li>如果您要访问单个文件，请输入直接指向此文件的路径，例如`hdfs://x.x.x.x/user/hive/warehouse/array2d_parq/data`。当运行查询时，StarRocks仅扫描此数据文件。</li></ul> |
| format                  | 是       | 数据文件的格式。有效值：`parquet`、`orc`、`avro`、`rctext`或`rcbinary`以及`sequence`。 |
| enable_recursive_listing| 否       | 指定是否递归遍历当前路径下的所有文件。默认值：`false`。            |

#### StorageCredentialParams (可选)

关于StarRocks如何与目标存储系统集成的一组参数。此参数集是**可选的**。

仅当目标存储系统为AWS S3或其他兼容S3存储系统时，您需要配置`StorageCredentialParams`。

对于其他存储系统，您可以忽略`StorageCredentialParams`。

##### AWS S3

如果需要访问存储在AWS S3中的数据文件，请配置`StorageCredentialParams`中的以下身份验证参数。

- 如果您选择基于实例配置文件的身份验证方法，请如下配置`StorageCredentialParams`：

```JavaScript
"aws.s3.use_instance_profile" = "true",
"aws.s3.region" = "<aws_s3_region>"
```

- 如果您选择基于假设角色的身份验证方法，请如下配置`StorageCredentialParams`：

```JavaScript
"aws.s3.use_instance_profile" = "true",
"aws.s3.iam_role_arn" = "<您假设角色的ARN>",
"aws.s3.region" = "<aws_s3_region>"
```

- 如果您选择基于IAM用户的身份验证方法，请如下配置`StorageCredentialParams`：

```JavaScript
"aws.s3.use_instance_profile" = "false",
"aws.s3.access_key" = "<IAM用户的访问密钥>",
"aws.s3.secret_key" = "<IAM用户的密钥>",
"aws.s3.region" = "<aws_s3_region>"
```

| 参数名称                  | 是否必需 | 描述                                                         |
| ------------------------ | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | 是      | 指定是否启用基于实例配置文件的身份验证方法以及访问AWS S3时的基于假设角色的身份验证方法。有效值：`true`和`false`。默认值：`false`。 |
| aws.s3.iam_role_arn      | 是       | 具有在您的AWS S3存储桶上具有权限的IAM角色的ARN。如果您使用基于假设角色的身份验证方法访问AWS S3，则必须指定此参数。然后，当StarRocks访问目标数据文件时，将假定此角色。 |
| aws.s3.region            | 是       | 您的AWS S3存储桶所在的区域。示例：us-west-1。               |
| aws.s3.access_key        | 否       | 您的IAM用户访问密钥。如果您使用基于IAM用户的身份验证方法访问AWS S3，则必须指定此参数。 |
| aws.s3.secret_key        | 否       | 您的IAM用户密钥。如果您使用基于IAM用户的身份验证方法访问AWS S3，则必须指定此参数。 |

有关如何选择用于访问AWS S3的身份验证方法以及如何在AWS IAM控制台中配置访问控制策略的信息，请参见[访问AWS S3的身份验证参数](../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)。

##### 兼容S3存储

如果需要访问兼容S3存储系统（例如MinIO），请按以下方式配置`StorageCredentialParams`以确保成功集成：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下表格描述了在`StorageCredentialParams`中需要配置的参数。

| 参数                           | 是否必须 | 描述                                                        |
| ------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl               | 是       | 指定是否启用SSL连接。 <br />有效值：`true`和`false`。默认值：`true`。 |
| aws.s3.enable_path_style_access | 是       | 指定是否启用路径样式访问。<br />有效值：`true`和`false`。默认值：`false`。对于MinIO，您必须将值设置为`true`。<br />路径样式URL使用以下格式：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例如，如果您在美国西部（俄勒冈州）区域创建名为`DOC-EXAMPLE-BUCKET1`的存储桶，并且要访问该存储桶中的`alice.jpg`对象，则可以使用以下路径样式URL：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                 | 是       | 用于连接到与AWS S3不同的S3兼容存储系统的端点。             |
| aws.s3.access_key               | 是       | 您的IAM用户的访问密钥。                          |
| aws.s3.secret_key               | 是       | 您的IAM用户的密钥。                          |

#### 列类型映射

以下表格提供了目标数据文件与文件外部表之间的列类型映射。

| 数据文件   | 文件外部表                                          |
| ----------- | ------------------------------------------------------------ |
| INT/INTEGER | INT                                                          |
| BIGINT      | BIGINT                                                       |
| TIMESTAMP   | DATETIME。 <br />请注意，时间戳根据当前会话的时区设置转换为不带时区的DATETIME，并且失去了一些精度。 |
| STRING      | STRING                                                       |
| VARCHAR     | VARCHAR                                                      |
| CHAR        | CHAR                                                         |
| DOUBLE      | DOUBLE                                                       |
| FLOAT       | FLOAT                                                        |
| DECIMAL     | DECIMAL                                                      |
| BOOLEAN     | BOOLEAN                                                      |
| ARRAY       | ARRAY                                                        |
| MAP         | MAP                                                          |
| STRUCT      | STRUCT                                                       |

### 示例

#### HDFS

创建名为`t0`的文件外部表，以查询存储在HDFS路径中的Parquet数据文件。

```SQL
USE db_example;

CREATE EXTERNAL TABLE t0
(
    name string, 
    id int
) 
ENGINE=file
PROPERTIES 
(
    "path"="hdfs://x.x.x.x:8020/user/hive/warehouse/person_parq/", 
    "format"="parquet"
);
```

#### AWS S3

示例1：创建文件外部表并使用**实例配置文件**访问AWS S3中的**单个Parquet文件**。

```SQL
USE db_example;

CREATE EXTERNAL TABLE table_1
(
    name string, 
    id int
) 
ENGINE=file
PROPERTIES 
(
    "path" = "s3://bucket-test/folder1/raw_0.parquet", 
    "format" = "parquet",
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.region" = "us-west-2" 
);
```

示例2：创建文件外部表并使用**假定角色**访问AWS S3中目标文件路径下的**所有ORC文件**。

```SQL
USE db_example;

CREATE EXTERNAL TABLE table_1
(
    name string, 
    id int
) 
ENGINE=file
PROPERTIES 
(
    "path" = "s3://bucket-test/folder1/", 
    "format" = "orc",
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.iam_role_arn" = "arn:aws:iam::51234343412:role/role_name_in_aws_iam",
    "aws.s3.region" = "us-west-2" 
);
```

示例3：创建文件外部表并使用**IAM用户**访问AWS S3中文件路径下的**所有ORC文件**。

```SQL
USE db_example;

CREATE EXTERNAL TABLE table_1
(
    name string, 
    id int
) 
ENGINE=file
PROPERTIES 
(
    "path" = "s3://bucket-test/folder1/", 
    "format" = "orc",
    "aws.s3.use_instance_profile" = "false",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_access_key>",
    "aws.s3.region" = "us-west-2" 
);
```

## 查询文件外部表

语法:

```SQL
SELECT <clause> FROM <file_external_table>
```

例如，要从[示例 - HDFS](#examples)中创建的文件外部表`t0`中查询数据，请运行以下命令：

```plain
SELECT * FROM t0;

+--------+------+
| name   | id   |
+--------+------+
| jack   |    2 |
| lily   |    1 |
+--------+------+
2 rows in set (0.08 sec)
```

## 管理文件外部表

您可以使用[DESC](../sql-reference/sql-statements/Utility/DESCRIBE.md)查看表的模式，或者使用[DROP TABLE](../sql-reference/sql-statements/data-definition/DROP_TABLE.md)删除表。