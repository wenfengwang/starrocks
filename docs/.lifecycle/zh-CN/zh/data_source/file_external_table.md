   displayed_sidebar: "Chinese"
   ---
   

   # 文件外部表

   

   文件外部表 (File External Table) 是一种特殊的外部表。您可以通过文件外部表直接查询外部存储系统上的 Parquet 和 ORC 格式的数据文件，无需导入数据。同时，文件外部表也不依赖任何 Metastore。StarRocks 当前支持的外部存储系统包括 HDFS、Amazon S3 及其他兼容 S3 协议的对象存储、阿里云对象存储 OSS 和腾讯云对象存储 COS。

   

   该特性从 StarRocks 2.5 版本开始支持。
   
   ## 使用限制

   

   - 当前仅支持在[default_catalog](../data_source/catalog/default_catalog.md)下的数据库内创建文件外部表，不支持 external catalog。您可以通过[SHOW CATALOGS](../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)来查询集群下的 catalog。

   - 仅支持查询 Parquet、ORC、Avro、RCFile、或 SequenceFile 格式的数据文件。

   - 目前**仅支持读取**目标数据文件中的数据，不支持例如 INSERT、DELETE、DROP 等**写入**操作。

   

   ## 前提条件
   
   创建文件外部表前，您需要在 StarRocks 中进行相应配置，以便集群能够访问数据文件所在的外部存储系统。具体配置步骤和 Hive catalog 相同（区别在于无需配置 Metastore）。详细信息参见[Hive catalog - 准备工作](../data_source/catalog/hive_catalog.md#准备工作)。

   

   ## 创建数据库 （可选）

   

   连接到 StarRocks 集群后，您可以在当前已有的数据库下创建文件外部表；也可以创建一个新的数据库来管理文件外部表。您可以使用[SHOW DATABASES](../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)来查询集群中的数据库，然后执行`USE <db_name>`切换到目标数据库。
   
   创建数据库的语法如下。
   
   ```SQL
   CREATE DATABASE [IF NOT EXISTS] <db_name>
   ```
   
   ## 创建文件外部表
   
   ### 语法
   
   切换到目前数据库后，您可以使用如下语法创建一个文件外部表。

   

   ```SQL
   CREATE EXTERNAL TABLE <table_name> 
   (
       <col_name> <col_type> [NULL | NOT NULL] [COMMENT "<comment>"]
   ) 
   ENGINE=FILE
   COMMENT ["comment"]
   PROPERTIES
   (
       FileLayoutParams,

       StorageCredentialParams

   )
   ```
   
   ### 参数说明
   

   | 参数             | 必选 | 说明                                                         |
   | ---------------- | ---- | ------------------------------------------------------------ |
   | table_name       | 是   | 文件外部表名称。命名要求如下：<ul><li> 必须由字母 (a-z 或 A-Z)、数字 (0-9) 或下划线 (_) 组成，且只能以字母开头。</li><li> 总长度不能超过 64 个字符。</li></ul> |
   | col_name         | 是   | 文件外部表的列名。列名大小写不敏感，需和数据文件中的保持一致，列的顺序无需保持一致。 |
   | col_type         | 是   | 文件外部表的列类型，需要根据[列类型映射](#列类型映射)来填写。 |

   | NULL \| NOT NULL | 否   | 文件外部表中的列是否允许为 NULL。<ul><li> NULL: 允许为 NULL。</li><li> NOT NULL: 不允许为 NULL。</li></ul> 您需要按照如下规则指定该参数：<ul><li> 如数据文件中的列没有指定该参数，则文件外部表中的列可以不指定或指定为 NULL。</li><li> 如数据文件中的列指定为 NULL，则文件外部表中的列可以不指定或指定为 NULL。</li><li> 如数据文件中的列指定为 NOT NULL，则文件外部表列必须指定为 NOT NULL。</li></ul>  |

   | comment          | 否   | 文件外部表的列备注。                                         |

   | ENGINE           | 是   | ENGINE 类型，取值为 file。                                   |

   | comment          | 否   | 文件外部表的备注信息。                                       |

   | PROPERTIES       | 是   | 表属性。 <ul><li> `FileLayoutParams`: 用于指定数据文件的路径和格式，必填。</li><li> `StorageCredentialParams`: 用于配置访问外部存储系统时所需的认证参数。仅当外部存储系统为 AWS S3 或其他兼容 S3 协议的对象存储时需要填写。</li></ul> |

   

   #### FileLayoutParams

   
   ```SQL
   "path" = "<file_path>",
   "format" = "<file_format>"

   "enable_recursive_listing" = "{ true | false }"

   ```
   
   | 参数                     | 必选 | 说明                                                         |
   | ------------------------ | -------- | ------------------------------------------------------------ |
   | path                     | 是       | 数据文件所在的路径。<ul><li> 若文件在 HDFS 上，则路径格式为 `hdfs://<HDFS的IP地址>:<端口号>/<路径>。`其中端口号默认为 8020，如使用默认端口号可忽略不在路径中指定。</li><li> 若文件在 Amazon S3 或其他兼容 S3 协议的对象存储上，则路径格式为 `s3://<bucket名称>/<folder>/`。</li></ul> 填写路径时，需注意以下两点： <ul><li> 如果要遍历路径下所有文件，则设置路径以 '/' 结尾，例如 `hdfs://x.x.x.x/user/hive/warehouse/array2d_parq/data/`。查询时，StarRocks 会遍历该路径下所有文件，但不做递归遍历。</li><li> 如果仅需查询路径下单个文件，则设置路径直接指向文件名，例如 `hdfs://x.x.x.x/user/hive/warehouse/array2d_parq/data`。查询时，StarRocks 会直接扫描该文件。</li></ul> |

   | format                   | 是       | 数据文件格式。取值范围：`parquet`、`orc`、`avro`、`rctext` 或 `rcbinary`、`sequence`。 |

   | enable_recursive_listing | 否       | 是否递归查询路径下所有文件。默认值：`false`。                  |
   
   #### `StorageCredentialParams`（可选）
   
   用于配置访问外部对象存储时所需的认证参数。
   

   仅当数据文件存储在 AWS S3 或其他兼容 S3 协议的对象存储时才需要填写。
   
   如果是其他的文件存储，则可以忽略 `StorageCredentialParams`。
| aws.s3.iam_role_arn         | 否       | IAM角色的ARN，该角色具有访问AWS S3 Bucket的权限。<br />在访问AWS S3时采用Assumed Role鉴权方式，必须指定此参数。StarRocks在访问目标数据文件时将采用此IAM角色。 |
| aws.s3.region               | 是       | AWS S3 Bucket所在的地域。示例：us-west-1。                  |
| aws.s3.access_key           | 否       | IAM用户的Access Key。<br />在访问AWS S3时采用IAM用户鉴权方式，必须指定此参数。 |
| aws.s3.secret_key           | 否       | IAM用户的Secret Key。<br />在访问AWS S3时采用IAM用户鉴权方式，必须指定此参数。 |

关于如何选择用于访问AWS S3的鉴权方式，以及如何在AWS IAM控制台配置访问控制策略，请参见[访问AWS S3的认证参数](../integrations/authenticate_to_aws_resources.md#访问-aws-s3-的认证参数)。

##### 阿里云OSS

如果数据文件存储在阿里云OSS上，需在`StorageCredentialParams`中配置以下认证参数：

```SQL
"aliyun.oss.access_key" = "<用户Access Key>",
"aliyun.oss.secret_key" = "<用户Secret Key>",
"aliyun.oss.endpoint" = "<OSS端点>" 
```

| 参数                            | 是否必须 | 说明                                                         |
| ------------------------------- | -------- | ------------------------------------------------------------ |
| aliyun.oss.endpoint             | 是      | 阿里云OSS端点，例如`oss-cn-beijing.aliyuncs.com`。可根据端点与地域的对应关系进行查找，请参见[访问域名和数据中心](https://help.aliyun.com/document_detail/31837.html)。    |
| aliyun.oss.access_key           | 是      | 指定阿里云账号或RAM用户的AccessKey ID，获取方式，请参见[获取AccessKey](https://help.aliyun.com/document_detail/53045.html)。                                     |
| aliyun.oss.secret_key           | 是      | 指定阿里云账号或RAM用户的AccessKey Secret，获取方式，请参见[获取AccessKey](https://help.aliyun.com/document_detail/53045.html)。                                     |

##### 兼容S3协议的对象存储

如果数据文件存储在兼容S3协议的对象存储上（如MinIO），需在`StorageCredentialParams`中配置以下认证参数：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<S3端点>",
"aws.s3.access_key" = "<IAM用户Access Key>",
"aws.s3.secret_key" = "<IAM用户Secret Key>"
```

| 参数                            | 是否必须 | 说明                                                         |
| ------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl               | 是      | 是否开启SSL连接。<br />取值范围：`true`和`false`。默认值：`true`。  |
| aws.s3.enable_path_style_access | 是      | 是否开启路径类型访问(Path-Style Access)。<br />取值范围：`true`和`false`。默认值：`false`。对于MinIO，必须设置为`true`。<br />路径类型URL使用如下格式：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例如，如果您在美国西部（俄勒冈）区域中创建一个名为`DOC-EXAMPLE-BUCKET1`的存储桶，并希望访问该存储桶中的`alice.jpg`对象，则可使用以下路径类型URL：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                 | 是      | 用于访问兼容S3协议的对象存储的端点。                      |
| aws.s3.access_key               | 是      | IAM用户的Access Key。                                       |
| aws.s3.secret_key               | 是      | IAM用户的Secret Key。                                       |

### 列类型映射

创建文件外部表时，需要根据数据文件的列类型指定文件外部表的列类型，具体映射关系如下。

| 数据文件  | 文件外部表                                                   |
| --------- | ------------------------------------------------------------ |
| INT       | INT                                                          |
| BIGINT    | BIGINT                                                       |
| TIMESTAMP | DATETIME<br />注意：TIMESTAMP转成DATETIME会损失精度，并根据当前会话设置的时区转成无时区的DATETIME。 |
| STRING    | STRING                                                       |
| VARCHAR   | VARCHAR                                                      |
| CHAR      | CHAR                                                         |
| DOUBLE    | DOUBLE                                                       |
| FLOAT     | FLOAT                                                        |
| DECIMAL   | DECIMAL                                                      |
| BOOLEAN   | BOOLEAN                                                      |
| ARRAY     | ARRAY                                                        |

### 创建示例

#### HDFS

创建文件外部表`t0`，用于访问存储在HDFS上的Parquet数据文件。

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

示例1：创建文件外部表`table_1`，用于访问存储在AWS S3上的**单个Parquet**数据文件，基于Instance Profile进行鉴权和认证。

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

示例2：创建文件外部表`table_1`，用于访问存储在AWS S3上某个路径下的**所有ORC数据文件**，基于Assumed Role进行鉴权和认证。

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

示例3：创建文件外部表`table_1`，用于访问存储在AWS S3上某个路径下的**所有ORC数据文件**，基于IAM User进行鉴权和认证。

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
    "aws.s3.access_key" = "<IAM用户Access Key>",
    "aws.s3.secret_key" = "<IAM用户Access Key>",
    "aws.s3.region" = "us-west-2" 
);
```

## 查询文件外部表

语法：

```sql
SELECT <clause> FROM <file_external_table>
```

例如，要查询HDFS示例中的文件外部表`t0`，可执行如下命令：

```plain
SELECT * FROM t0;

+--------+------+
| name   | id   |
+--------+------+
| jack | 2 |
| lily   | 1 |
+--------+------+
2 rows in set (0.08 sec)
```

## 管理文件外部表

您可以通过 [DESC](../sql-reference/sql-statements/Utility/DESCRIBE.md) 命令来查询外部文件表的信息和表结构，或者使用 [DROP TABLE](../sql-reference/sql-statements/data-definition/DROP_TABLE.md) 命令来删除外部文件表。