---
displayed_sidebar: English
---

# 文件外部表

文件外部表是一种特殊类型的外部表。它允许您直接查询外部存储系统中的 Parquet 和 ORC 数据文件，而无需将数据加载到 StarRocks 中。此外，文件外部表不依赖于元数据存储。在当前版本中，StarRocks 支持以下外部存储系统：HDFS、Amazon S3 以及其他兼容 S3 的存储系统。

该功能自 StarRocks v2.5 版本起支持。

## 限制

- 文件外部表必须在 [default_catalog](../data_source/catalog/default_catalog.md) 中的数据库内创建。您可以运行 [SHOW CATALOGS](../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) 来查询集群中创建的目录。
- 仅支持 Parquet、ORC、Avro、RCFile 和 SequenceFile 数据文件。
- 您只能使用文件外部表来查询目标数据文件中的数据。不支持 INSERT、DELETE 和 DROP 等数据写入操作。

## 先决条件

在创建文件外部表之前，您必须配置 StarRocks 集群，以便 StarRocks 能够访问存储目标数据文件的外部存储系统。文件外部表所需的配置与 Hive 目录所需的配置相同，除了不需要配置元数据存储。有关配置的更多信息，请参阅 [Hive 目录 - 集成准备](../data_source/catalog/hive_catalog.md#integration-preparations)。

## 创建数据库（可选）

连接到 StarRocks 集群后，您可以在现有数据库中创建文件外部表，或创建新数据库来管理文件外部表。要查询集群中现有的数据库，请运行 [SHOW DATABASES](../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)。然后您可以运行 `USE <db_name>` 来切换到目标数据库。

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

|参数|必填|描述|
|---|---|---|
|table_name|是|文件外部表的名称。命名规则如下：<ul><li>名称可以包含字母、数字（0-9）和下划线（_）。它必须以字母开头。</li><li>名称长度不能超过 64 个字符。</li></ul>|
|col_name|是|文件外部表中的列名。文件外部表中的列名必须与目标数据文件中的列名相同，但不区分大小写。文件外部表中的列顺序可以与目标数据文件中的列顺序不同。|
|col_type|是|文件外部表中的列类型。您需要根据目标数据文件中的列类型指定此参数。有关详细信息，请参阅[列类型映射](#mapping-of-column-types)。|
|NULL \| NOT NULL|否|文件外部表中的列是否允许为 NULL。<ul><li>NULL：允许 NULL。</li><li>NOT NULL：不允许 NULL。</li></ul>您必须根据以下规则指定此修饰符：<ul><li>如果目标数据文件中的列未指定此参数，您可以选择不为文件外部表中的列指定，或为文件外部表中的列指定 NULL。</li><li>如果目标数据文件中的列指定为 NULL，您可以选择不为文件外部表中的列指定此参数，或为文件外部表中的列指定 NULL。</li><li>如果目标数据文件中的列指定为 NOT NULL，您也必须为文件外部表中的列指定 NOT NULL。</li></ul>|
|comment|否|文件外部表中列的注释。|
|ENGINE|是|引擎类型。将值设置为 file。|
|comment|否|文件外部表的描述。|
|PROPERTIES|是|<ul><li>`FileLayoutParams`：指定目标文件的路径和格式。此属性是必需的。</li><li>`StorageCredentialParams`：指定访问对象存储系统所需的认证信息。仅对 AWS S3 和其他兼容 S3 的存储系统是必需的。</li></ul>|

#### FileLayoutParams

用于访问目标数据文件的一组参数。

```SQL
"path" = "<file_path>",
"format" = "<file_format>",
"enable_recursive_listing" = "{ true | false }"
```

|参数|必填|描述|
|---|---|---|
|path|是|数据文件的路径。<ul><li>如果数据文件存储在 HDFS 中，路径格式为 `hdfs://<HDFS 的 IP 地址>:<端口>/<路径>`。默认端口号为 8020。如果使用默认端口，则无需指定。</li><li>如果数据文件存储在 AWS S3 或其他兼容 S3 的存储系统中，路径格式为 `s3://<bucket 名称>/<文件夹>/`。</li></ul>输入路径时请注意以下规则：<ul><li>如果您想访问路径中的所有文件，请以斜线（/）结尾，例如 `hdfs://x.x.x.x/user/hive/warehouse/array2d_parq/data/`。当您运行查询时，StarRocks 会遍历该路径下的所有数据文件。它不会递归地遍历数据文件。</li><li>如果您想访问单个文件，请输入直接指向该文件的路径，例如 `hdfs://x.x.x.x/user/hive/warehouse/array2d_parq/data`。当您运行查询时，StarRocks 只会扫描这个数据文件。</li></ul>|
|format|是|数据文件的格式。有效值：`parquet`、`orc`、`avro`、`rctext` 或 `rcbinary` 和 `sequence`。|
|enable_recursive_listing|否|指定是否递归遍历当前路径下的所有文件。默认值：`false`。|

#### StorageCredentialParams（可选）

一组关于 StarRocks 如何与目标存储系统集成的参数。这组参数是**可选**的。

仅当目标存储系统是 AWS S3 或其他兼容 S3 的存储时，才需要配置 `StorageCredentialParams`。

对于其他存储系统，您可以忽略 `StorageCredentialParams`。

##### AWS S3

如果您需要访问存储在 AWS S3 中的数据文件，请在 `StorageCredentialParams` 中配置以下认证参数。

- 如果您选择基于实例配置文件的认证方法，请按以下方式配置 `StorageCredentialParams`：

```JavaScript
"aws.s3.use_instance_profile" = "true",
"aws.s3.region" = "<aws_s3_region>"
```

- 如果您选择基于假设角色的认证方法，请按以下方式配置 `StorageCredentialParams`：

```JavaScript
"aws.s3.use_instance_profile" = "true",
"aws.s3.iam_role_arn" = "<ARN of your assumed role>",
"aws.s3.region" = "<aws_s3_region>"
```
```
```JavaScript
"aws.s3.use_instance_profile" = "false",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>",
"aws.s3.region" = "<aws_s3_region>"
```

|参数名称|必填|描述|
|---|---|---|
|aws.s3.use_instance_profile|是|指定是否启用基于实例配置文件的身份验证方法或基于假定角色的身份验证方法来访问AWS S3。有效值：`true` 和 `false`。默认值：`false`。|
|aws.s3.iam_role_arn|否|您的AWS S3存储桶有权限的IAM角色的ARN。<br />如果您使用基于假定角色的身份验证方法来访问AWS S3，则必须指定此参数。此时，StarRocks会扮演此角色来访问目标数据文件。|
|aws.s3.region|是|您的AWS S3存储桶所在的区域。例如：us-west-1。|
|aws.s3.access_key|否|您的IAM用户的访问密钥。如果您使用基于IAM用户的身份验证方法来访问AWS S3，则必须指定此参数。|
|aws.s3.secret_key|否|您的IAM用户的密钥。如果您使用基于IAM用户的身份验证方法来访问AWS S3，则必须指定此参数。|

有关如何选择访问AWS S3的身份验证方法以及如何在AWS IAM控制台中配置访问控制策略的信息，请参阅[访问AWS S3的身份验证参数](../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)。

##### S3兼容存储

如果您需要访问S3兼容的存储系统（例如MinIO），请按如下方式配置`StorageCredentialParams`以确保成功集成：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

下表描述了您需要在`StorageCredentialParams`中配置的参数。

|参数|必填|说明|
|---|---|---|
|aws.s3.enable_ssl|是|指定是否启用SSL连接。<br />有效值：`true` 和 `false`。默认值：`true`。|
|aws.s3.enable_path_style_access|是|指定是否启用路径样式访问。<br />有效值：`true` 和 `false`。默认值：`false`。对于MinIO，您必须将该值设置为`true`。<br />路径样式URL使用以下格式：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例如，如果您在美国西部（俄勒冈）区域创建名为DOC-EXAMPLE-BUCKET1的存储桶，并且想要访问该存储桶中的`alice.jpg`对象，则可以使用以下路径样式URL：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。|
|aws.s3.endpoint|是|用于连接到S3兼容存储系统而不是AWS S3的终端节点。|
|aws.s3.access_key|是|您的IAM用户的访问密钥。|
|aws.s3.secret_key|是|您的IAM用户的密钥。|

#### 列类型映射

下表提供了目标数据文件和文件外部表之间列类型的映射。

|数据文件|文件外部表|
|---|---|
|INT/INTEGER|INT|
|BIGINT|BIGINT|
|TIMESTAMP|DATETIME。<br />请注意，TIMESTAMP会根据当前会话的时区设置转换为不带时区的DATETIME，并且会失去一些精度。|
|STRING|STRING|
|VARCHAR|VARCHAR|
|CHAR|CHAR|
|DOUBLE|DOUBLE|
|FLOAT|FLOAT|
|DECIMAL|DECIMAL|
|BOOLEAN|BOOLEAN|
|ARRAY|ARRAY|
|MAP|MAP|
|STRUCT|STRUCT|

### 示例

#### HDFS

创建名为`t0`的文件外部表，用于查询存储在HDFS路径中的Parquet数据文件。

```SQL
USE db_example;

CREATE EXTERNAL TABLE t0
(
    name STRING, 
    id INT
) 
ENGINE=file
PROPERTIES 
(
    "path"="hdfs://x.x.x.x:8020/user/hive/warehouse/person_parq/", 
    "format"="parquet"
);
```

#### AWS S3

示例1：创建文件外部表并使用**实例配置文件**来访问**AWS S3**中的**单个Parquet文件**。

```SQL
USE db_example;

CREATE EXTERNAL TABLE table_1
(
    name STRING, 
    id INT
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

示例2：创建文件外部表并使用**假定角色**来访问**AWS S3**中目标文件路径下的**所有ORC文件**。

```SQL
USE db_example;

CREATE EXTERNAL TABLE table_1
(
    name STRING, 
    id INT
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

示例3：创建文件外部表并使用**IAM用户**来访问**AWS S3**中文件路径下的**所有ORC文件**。

```SQL
USE db_example;

CREATE EXTERNAL TABLE table_1
(
    name STRING, 
    id INT
) 
ENGINE=file
PROPERTIES 
(
    "path" = "s3://bucket-test/folder1/", 
    "format" = "orc",
    "aws.s3.use_instance_profile" = "false",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>",
    "aws.s3.region" = "us-west-2" 
);
```

## 查询文件外部表

语法：

```SQL
SELECT <clause> FROM <file_external_table>
```

例如，要从在[示例 - HDFS](#examples)中创建的文件外部表`t0`查询数据，运行以下命令：

```plain
SELECT * FROM t0;

+--------+------+
| name   | id   |
+--------+------+
| jack   |    2 |
| lily   |    1 |
+--------+------+
2行在集(0.08秒)
```
```markdown
# 文件外部表

文件外部表是一种特殊类型的外部表。它允许您直接查询外部存储系统中的 Parquet 和 ORC 数据文件，而无需将数据加载到 StarRocks 中。此外，文件外部表不依赖于元数据存储。在当前版本中，StarRocks 支持以下外部存储系统：HDFS、Amazon S3 和其他兼容 S3 的存储系统。

该功能从 StarRocks v2.5 版本开始支持。

## 限制

- 文件外部表必须在 [default_catalog](../data_source/catalog/default_catalog.md) 中的数据库内创建。您可以运行 [SHOW CATALOGS](../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) 查询集群中创建的目录。
- 仅支持 Parquet、ORC、Avro、RCFile 和 SequenceFile 数据文件。
- 您只能使用文件外部表查询目标数据文件中的数据。不支持 INSERT、DELETE 和 DROP 等数据写入操作。

## 先决条件

在创建文件外部表之前，您必须配置 StarRocks 集群，以便 StarRocks 能够访问存储目标数据文件的外部存储系统。文件外部表所需的配置与 Hive 目录所需的配置相同，只是您不需要配置元数据存储。有关配置的更多信息，请参见 [Hive 目录 - 集成准备](../data_source/catalog/hive_catalog.md#integration-preparations)。

## 创建数据库（可选）

连接到 StarRocks 集群后，您可以在现有数据库中创建文件外部表，或创建新数据库来管理文件外部表。要查询集群中现有的数据库，请运行 [SHOW DATABASES](../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)。然后您可以运行 `USE <db_name>` 切换到目标数据库。

创建数据库的语法如下。

```SQL
CREATE DATABASE [IF NOT EXISTS] <db_name>
```

## 创建文件外部表

访问目标数据库后，您可以在此数据库中创建文件外部表。

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

|参数|是否必须|描述|
|---|---|---|
|table_name|是|文件外部表的名称。命名规则如下：<ul><li> 名称可以包含字母、数字 (0-9) 和下划线 (_)。必须以字母开头。</li><li> 名称长度不能超过 64 个字符。</li></ul>|
|col_name|是|文件外部表中的列名。文件外部表中的列名必须与目标数据文件中的列名相同，但不区分大小写。文件外部表中的列顺序可以与目标数据文件中的不同。|
|col_type|是|文件外部表中的列类型。您需要根据目标数据文件中的列类型指定此参数。有关更多信息，请参见 [列类型映射](#列类型映射)。|
|NULL \| NOT NULL|否|文件外部表中的列是否允许为 NULL。<ul><li>NULL：允许 NULL。</li><li>NOT NULL：不允许 NULL。</li></ul>您必须根据以下规则指定此修饰符：<ul><li>如果目标数据文件中的列未指定此参数，则您可以选择不为文件外部表中的列指定此参数，或为文件外部表中的列指定 NULL。</li><li>如果目标数据文件中的列指定了 NULL，则您可以选择不为文件外部表中的列指定此参数，或为文件外部表中的列指定 NULL。</li><li>如果目标数据文件中的列指定了 NOT NULL，则您也必须为文件外部表中的列指定 NOT NULL。</li></ul>|
|comment|否|文件外部表中列的注释。|
|ENGINE|是|引擎类型。将值设置为 file。|
|comment|否|文件外部表的描述。|
|PROPERTIES|是|<ul><li>`FileLayoutParams`：指定目标文件的路径和格式。此属性是必需的。</li><li>`StorageCredentialParams`：指定访问对象存储系统所需的认证信息。此属性仅对 AWS S3 和其他兼容 S3 的存储系统是必需的。</li></ul>|

#### FileLayoutParams

一组用于访问目标数据文件的参数。

```SQL
"path" = "<file_path>",
"format" = "<file_format>"
"enable_recursive_listing" = "{ true | false }"
```

|参数|是否必须|描述|
|---|---|---|
|path|是|数据文件的路径。<ul><li>如果数据文件存储在 HDFS 中，路径格式为 `hdfs://<HDFS 的 IP 地址>:<端口>/<路径>`。默认端口号为 8020。如果您使用默认端口，则不需要指定。</li><li>如果数据文件存储在 AWS S3 或其他兼容 S3 的存储系统中，路径格式为 `s3://<bucket 名称>/<文件夹>/`。</li></ul> 输入路径时请注意以下规则：<ul><li>如果您想访问路径下的所有文件，请以斜杠 (`/`) 结尾，例如 `hdfs://x.x.x.x/user/hive/warehouse/array2d_parq/data/`。当您运行查询时，StarRocks 遍历路径下的所有数据文件。它不会使用递归遍历数据文件。</li><li>如果您想访问单个文件，请输入直接指向该文件的路径，例如 `hdfs://x.x.x.x/user/hive/warehouse/array2d_parq/data`。当您运行查询时，StarRocks 只扫描这个数据文件。</li></ul>|
|format|是|数据文件的格式。有效值：`parquet`、`orc`、`avro`、`rctext` 或 `rcbinary` 和 `sequence`。|
|enable_recursive_listing|否|指定是否递归遍历当前路径下的所有文件。默认值：`false`。|

#### StorageCredentialParams（可选）

一组关于 StarRocks 如何与目标存储系统集成的参数。此参数集是**可选的**。

只有当目标存储系统是 AWS S3 或其他兼容 S3 的存储时，您才需要配置 `StorageCredentialParams`。

对于其他存储系统，您可以忽略 `StorageCredentialParams`。

##### AWS S3

如果您需要访问存储在 AWS S3 中的数据文件，请在 `StorageCredentialParams` 中配置以下认证参数。

- 如果您选择基于实例配置文件的认证方法，请按如下方式配置 `StorageCredentialParams`：

```JavaScript
"aws.s3.use_instance_profile" = "true",
"aws.s3.region" = "<aws_s3_region>"
```

- 如果您选择基于假设角色的认证方法，请按如下方式配置 `StorageCredentialParams`：

```JavaScript
"aws.s3.use_instance_profile" = "true",
"aws.s3.iam_role_arn" = "<您的假设角色的 ARN>",
"aws.s3.region" = "<aws_s3_region>"
```

- 如果您选择基于 IAM 用户的认证方法，请按如下方式配置 `StorageCredentialParams`：

```JavaScript
"aws.s3.use_instance_profile" = "false",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>",
"aws.s3.region" = "<aws_s3_region>"
```

|参数名称|是否必须|描述|
|---|---|---|
|aws.s3.use_instance_profile|是|指定是否启用基于实例配置文件的认证方法和基于假设角色的认证方法来访问 AWS S3。有效值：`true` 和 `false`。默认值：`false`。|
|aws.s3.iam_role_arn|是|具有您 AWS S3 存储桶权限的 IAM 角色的 ARN。<br />如果您使用基于假设角色的认证方法访问 AWS S3，则必须指定此参数。然后，StarRocks 将在访问目标数据文件时假设此角色。|
|aws.s3.region|是|您的 AWS S3 存储桶所在的区域。示例：us-west-1。|
|aws.s3.access_key|否|您的 IAM 用户的访问密钥。如果您使用基于 IAM 用户的认证方法访问 AWS S3，则必须指定此参数。|
|aws.s3.secret_key|否|您的 IAM 用户的密钥。如果您使用基于 IAM 用户的认证方法访问 AWS S3，则必须指定此参数。|

有关如何选择访问 AWS S3 的认证方法以及如何在 AWS IAM 控制台中配置访问控制策略的信息，请参见 [访问 AWS S3 的认证参数](../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)。

##### 兼容 S3 的存储

如果您需要访问兼容 S3 的存储系统，例如 MinIO，请按如下方式配置 `StorageCredentialParams` 以确保成功集成：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下表格描述了您需要在 `StorageCredentialParams` 中配置的参数。

|参数|是否必须|描述|
|---|---|---|
|aws.s3.enable_ssl|是|指定是否启用 SSL 连接。<br />有效值：`true` 和 `false`。默认值：`true`。|
|aws.s3.enable_path_style_access|是|指定是否启用路径风格访问。<br />有效值：`true` 和 `false`。默认值：`false`。对于 MinIO，您必须将值设置为 `true`。<br />路径风格的 URL 使用以下格式：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例如，如果您在美国西部（俄勒冈）区域创建了一个名为 `DOC-EXAMPLE-BUCKET1` 的存储桶，并且您想要访问该存储桶中的 `alice.jpg` 对象，您可以使用以下路径风格的 URL：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。|
|aws.s3.endpoint|是|用于连接兼容 S3 的存储系统的端点，而不是 AWS S3。|
|aws.s3.access_key|是|您的 IAM 用户的访问密钥。|
|aws.s3.secret_key|是|您的 IAM 用户的密钥。|

#### 列类型映射

以下表格提供了目标数据文件与文件外部表之间的列类型映射。

|数据文件|文件外部表|
|---|---|
|INT/INTEGER|INT|
|BIGINT|BIGINT|
|TIMESTAMP|DATETIME。<br />请注意，TIMESTAMP 根据当前会话的时区设置转换为 DATETIME，没有时区，并且会丢失一些精度。|
|STRING|STRING|
|VARCHAR|VARCHAR|
|CHAR|CHAR|
|DOUBLE|DOUBLE|
|FLOAT|FLOAT|
|DECIMAL|DECIMAL|
|BOOLEAN|BOOLEAN|
|ARRAY|ARRAY|
|MAP|MAP|
|STRUCT|STRUCT|

### 示例

#### HDFS

创建一个名为 `t0` 的文件外部表，以查询存储在 HDFS 路径中的 Parquet 数据文件。

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

示例 1：创建一个文件外部表，并使用 **实例配置文件** 访问 AWS S3 中的 **单个 Parquet 文件**。

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

示例 2：创建一个文件外部表，并使用 **假设角色** 访问 AWS S3 中目标文件路径下的 **所有 ORC 文件**。

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

示例 3：创建一个文件外部表，并使用 **IAM 用户** 访问 AWS S3 中文件路径下的 **所有 ORC 文件**。

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
    "aws.s3.secret_key" = "<iam_user_secret_key>",
    "aws.s3.region" = "us-west-2" 
);
```

## 查询文件外部表

语法：

```SQL
SELECT <clause> FROM <file_external_table>
```

例如，要从 [示例 - HDFS](#示例) 中创建的文件外部表 `t0` 查询数据，请运行以下命令：

```plain
SELECT * FROM t0;

+--------+------+
| name   | id   |
+--------+------+
| jack   |    2 |
| lily   |    1 |
+--------+------+
2 行在集 (0.08 秒)
```

## 管理文件外部表

您可以使用 [DESC](../sql-reference/sql-statements/Utility/DESCRIBE.md) 查看表的架构或使用 [DROP TABLE](../sql-reference/sql-statements/data-definition/DROP_TABLE.md) 删除表。