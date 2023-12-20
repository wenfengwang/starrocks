---
displayed_sidebar: English
---

# 文件外部表

文件外部表是一种特殊类型的外部表。它允许您直接在外部存储系统中查询 Parquet 和 ORC 数据文件，而无需将数据加载进 StarRocks。此外，文件外部表不依赖元数据存储。在当前版本中，StarRocks 支持以下外部存储系统：HDFS、Amazon S3 以及其他兼容 S3 的存储系统。

该功能从 StarRocks v2.5 版本开始支持。

## 限制

- 文件外部表必须创建在数据库中的[default_catalog](../data_source/catalog/default_catalog.md)。您可以运行[SHOW CATALOGS](../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)命令来查询集群中已创建的目录。
- 只支持 Parquet、ORC、Avro、RCFile 和 SequenceFile 数据文件格式。
- 您只能使用文件外部表来查询目标数据文件中的数据。不支持 INSERT、DELETE 和 DROP 等数据写入操作。

## 先决条件

在创建文件外部表之前，您必须配置您的 StarRocks 集群，以便 StarRocks 能够访问存储目标数据文件的外部存储系统。文件外部表的配置与 Hive 目录的配置相同，唯一的区别是不需要配置元数据存储。请参见[Hive 目录 - Integration preparations](../data_source/catalog/hive_catalog.md#integration-preparations)了解更多有关配置的信息。

## 创建数据库（可选）

连接到 StarRocks 集群后，您可以在现有数据库中创建文件外部表，或者创建一个新的数据库来管理文件外部表。要查询集群中的现有数据库，请运行 [SHOW DATABASES](../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)。然后，您可以运行 `USE \\<db_name\\>` 来切换到目标数据库。

创建数据库的语法如下所示。

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

|参数|必填|说明|
|---|---|---|
|table_name|Yes|文件外部表的名称。命名规则如下： 名称可以包含字母、数字（0-9）和下划线（_）。它必须以字母开头。名称长度不能超过 64 个字符。|
|col_name|Yes|文件外部表中的列名。文件外部表中的列名必须与目标数据文件中的列名相同，但不区分大小写。文件外部表中的列顺序可以与目标数据文件中的列顺序不同。|
|col_type|Yes|文件外部表中的列类型。您需要根据目标数据文件中的列类型指定该参数。有关详细信息，请参阅列类型的映射。|
|空\| NOT NULL|否|文件外部表中的列是否允许为NULL。 NULL：允许 NULL。 NOT NULL：不允许为NULL。必须根据以下规则指定该修饰符：如果目标数据文件中的列没有指定该参数，则可以选择文件外部表中的列不指定或为文件外部表中的列指定NULL。如果目标数据文件中的列指定为NULL，则可以选择不为文件外部表中的列指定该参数，或者为文件外部表中的列指定NULL表。如果为目标数据文件中的列指定 NOT NULL，则还必须为文件外部表中的列指定 NOT NULL。|
|注释|否|文件外部表中列的注释。|
|发动机|是|发动机的类型。将值设置为文件。|
|注释|否|文件外部表的描述。|
|PROPERTIES|Yes|FileLayoutParams：指定目标文件的路径和格式。该属性为必填项。StorageCredentialParams：指定访问对象存储系统所需的身份验证信息。仅 AWS S3 和其他与 S3 兼容的存储系统需要此属性。|

#### 文件布局参数

用于访问目标数据文件的参数集合。

```SQL
"path" = "<file_path>",
"format" = "<file_format>"
"enable_recursive_listing" = "{ true | false }"
```

|参数|必填|说明|
|---|---|---|
|path|Yes|数据文件的路径。如果数据文件存储在HDFS中，则路径格式为hdfs://<HDFS的IP地址>:<端口>/<路径>。默认端口号为 8020。如果使用默认端口，则无需指定。如果数据文件存储在 AWS S3 或其他 S3 兼容的存储系统中，则路径格式为 s3://<存储桶名称>/<文件夹>/。输入路径时请注意以下规则： 如果要访问路径中的所有文件，请以斜杠（/）结尾，例如 hdfs://x.x.x.x/user/hive/warehouse/array2d_parq/data/。当您运行查询时，StarRocks 会遍历该路径下的所有数据文件。它不会使用递归的方式遍历数据文件。如果要访问单个文件，请输入直接指向该文件的路径，例如hdfs://x.x.x.x/user/hive/warehouse/array2d_parq/data。当您运行查询时，StarRocks 仅扫描此数据文件。|
|格式|是|数据文件的格式。有效值：parquet、orc、avro、rctext 或 rcbinary 以及序列。|
|enable_recursive_listing|No|指定是否递归遍历当前路径下的所有文件。默认值： false。|

#### StorageCredentialParams（可选）

关于 StarRocks 如何与目标存储系统集成的参数集合。这个参数集是**可选**的。

只有当目标存储系统为 AWS S3 或其他兼容 S3 的存储时，才需要配置 StorageCredentialParams。

对于其他存储系统，您可以忽略 StorageCredentialParams。

##### AWS S3

如果您需要访问存储在 AWS S3 中的数据文件，请在 StorageCredentialParams 中配置以下认证参数。

- 如果您选择基于实例配置文件的认证方式，请如下配置 StorageCredentialParams：

```JavaScript
"aws.s3.use_instance_profile" = "true",
"aws.s3.region" = "<aws_s3_region>"
```

- 如果您选择基于假定角色的认证方式，请如下配置 StorageCredentialParams：

```JavaScript
"aws.s3.use_instance_profile" = "true",
"aws.s3.iam_role_arn" = "<ARN of your assumed role>",
"aws.s3.region" = "<aws_s3_region>"
```

- 如果您选择基于 IAM 用户的认证方式，请如下配置 StorageCredentialParams：

```JavaScript
"aws.s3.use_instance_profile" = "false",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>",
"aws.s3.region" = "<aws_s3_region>"
```

|参数名称|必填|描述|
|---|---|---|
|aws.s3.use_instance_profile|Yes|指定在访问 AWS S3 时是否启用基于实例配置文件的身份验证方法和假定的基于角色的身份验证方法。有效值：true 和 false。默认值： false。|
|aws.s3.iam_role_arn|是|对您的 AWS S3 存储桶拥有权限的 IAM 角色的 ARN。如果您使用假定的基于角色的身份验证方法来访问 AWS S3，则必须指定此参数。然后，StarRocks 在访问目标数据文件时将承担此角色。|
|aws.s3.region|是|您的 AWS S3 存储桶所在的区域。示例：us-west-1。|
|aws.s3.access_key|否|您的 IAM 用户的访问密钥。如果您使用基于 IAM 用户的身份验证方法访问 AWS S3，则必须指定此参数。|
|aws.s3.secret_key|否|您的 IAM 用户的密钥。如果您使用基于 IAM 用户的身份验证方法访问 AWS S3，则必须指定此参数。|

有关如何选择访问**AWS S3**的认证方式以及如何在**AWS IAM**控制台中配置访问控制策略的信息，请参见[Authentication parameters for accessing AWS S3](../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)部分。

##### 兼容 S3 的存储

如果您需要访问兼容 S3 的存储系统，例如 MinIO，请如下配置 StorageCredentialParams 以确保成功集成：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

下表描述了您需要在 StorageCredentialParams 中配置的参数。

|参数|必填|说明|
|---|---|---|
|aws.s3.enable_ssl|Yes|指定是否启用 SSL 连接。有效值：true 和 false。默认值：true。|
|aws.s3.enable_path_style_access|Yes|指定是否启用路径样式访问。有效值：true 和 false。默认值：假。对于 MinIO，您必须将该值设置为 true。路径样式 URL 使用以下格式：https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>。例如，如果您在美国西部（俄勒冈）区域创建名为 DOC-EXAMPLE-BUCKET1 的存储桶，并且想要访问该存储桶中的 alice.jpg 对象，则可以使用以下路径样式 URL：https:// /s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg.|
|aws.s3.endpoint|是|用于连接到 S3 兼容存储系统而不是 AWS S3 的终端节点。|
|aws.s3.access_key|是|您的 IAM 用户的访问密钥。|
|aws.s3.secret_key|是|您的 IAM 用户的密钥。|

#### 列类型映射

下表提供了目标数据文件与文件外部表之间列类型的映射。

|数据文件|文件外部表|
|---|---|
|整数/整数|整数|
|BIGINT|BIGINT|
|时间戳|日期时间。请注意，TIMESTAMP 会根据当前会话的时区设置转换为不带时区的 DATETIME，并会失去一些精度。|
|字符串|字符串|
|VARCHAR|VARCHAR|
|字符|字符|
|双|双|
|浮动|浮动|
|十进制|十进制|
|布尔值|布尔值|
|阵列|阵列|
|地图|地图|
|结构|结构|

### 示例

#### HDFS

创建名为 t0 的文件外部表，用于查询存储在 HDFS 路径中的 Parquet 数据文件。

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

示例 1：创建文件外部表并使用**实例配置文件**来访问 AWS S3 中的**单个 Parquet 文件**。

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

示例 2：创建文件外部表并使用**假定角色**来访问**所有 ORC 文件** under the target file path in [AWS S3](https://aws.amazon.com/s3)。

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

示例 3：创建文件外部表并使用 **IAM 用户** 来访问 AWS S3 中文件路径下的所有 **ORC 文件**。

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

语法：

```SQL
SELECT <clause> FROM <file_external_table>
```

例如，要查询在[示例 - HDFS](#examples)中创建的文件外部表`t0`中的数据，请执行以下命令：

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

您可以使用 [DESC](../sql-reference/sql-statements/Utility/DESCRIBE.md) 命令查看表的结构，或使用 [DROP TABLE](../sql-reference/sql-statements/data-definition/DROP_TABLE.md) 命令删除表。
