---
displayed_sidebar: "Chinese"
---

# 文件

## 描述

定义远程存储中的数据文件。

从v3.1.0版本开始，StarRocks支持使用表函数FILES()定义远程存储中的只读文件。它可以使用文件的路径相关属性访问远程存储，推断文件中数据的表模式，以及返回数据行。您可以使用[SELECT](../../sql-statements/data-manipulation/SELECT.md)直接查询数据行，使用[INSERT](../../sql-statements/data-manipulation/INSERT.md)将数据行加载到现有表中，或者使用[CREATE TABLE AS SELECT](../../sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)创建新表并将数据行加载到其中。

从v3.2.0版本开始，FILES()支持将数据写入远程存储中的文件。您可以使用INSERT INTO FILES()将数据从StarRocks卸载到远程存储。

目前，FILES()函数支持以下数据源和文件格式：

- **数据源:**
  - HDFS
  - AWS S3
  - Google Cloud Storage
  - 其他兼容S3的存储系统
  - Microsoft Azure Blob Storage
- **文件格式:**
  - Parquet
  - ORC（当前不支持卸载数据）

## 语法

- **数据加载**:

  ```SQL
  FILES( data_location , data_format [, StorageCredentialParams ] [, columns_from_path ] )
  ```

- **数据卸载**:

  ```SQL
  FILES( data_location , data_format [, StorageCredentialParams ] , unload_data_param )
  ```

## 参数

所有参数均为`"key" = "value"`对。

### data_location

用于访问文件的URI。您可以指定路径或文件。

- 访问HDFS时，您需要将此参数指定为：

  ```SQL
  "path" = "hdfs://<hdfs_host>:<hdfs_port>/<hdfs_path>"
  -- 例如: "path" = "hdfs://127.0.0.1:9000/path/file.parquet"
  ```

- 访问AWS S3:

  - 如果使用S3协议，您需要将此参数指定为：

    ```SQL
    "path" = "s3://<s3_path>"
    -- 例如: "path" = "s3://path/file.parquet"
    ```

  - 如果使用S3A协议，您需要将此参数指定为：

    ```SQL
    "path" = "s3a://<s3_path>"
    -- 例如: "path" = "s3a://path/file.parquet"
    ```

- 访问Google Cloud Storage，您需要将此参数指定为：

  ```SQL
  "path" = "s3a://<gcs_path>"
  -- 例如: "path" = "s3a://path/file.parquet"
  ```

- 访问Azure Blob Storage:

  - 如果您的存储帐户允许通过HTTP访问，您需要将此参数指定为：

    ```SQL
    "path" = "wasb://<container>@<storage_account>.blob.core.windows.net/<blob_path>"
    -- 例如: "path" = "wasb://testcontainer@testaccount.blob.core.windows.net/path/file.parquet"
    ```

  - 如果您的存储帐户允许通过HTTPS访问，您需要将此参数指定为：

    ```SQL
    "path" = "wasbs://<container>@<storage_account>.blob.core.windows.net/<blob_path>"
    -- 例如: "path" = "wasbs://testcontainer@testaccount.blob.core.windows.net/path/file.parquet"
    ```

### data_format

数据文件的格式。有效取值: `parquet` 和 `orc`。

### StorageCredentialParams

StarRocks用于访问您的存储系统的身份验证信息。

StarRocks当前支持使用简单身份验证访问HDFS，使用IAM用户身份验证访问AWS S3和GCS，并使用Shared Key访问Azure Blob Storage。

- 使用简单身份验证访问HDFS:

  ```SQL
  "hadoop.security.authentication" = "simple",
  "username" = "xxxxxxxxxx",
  "password" = "yyyyyyyyyy"
  ```

  | **键**                       | **必需** | **描述**                                                     |
  | --------------------------- | -------- | ------------------------------------------------------------ |
  | hadoop.security.authentication | 否       | 认证方法。有效取值: `simple`（默认）。`simple`表示简单身份验证，即无身份验证。 |
  | username                    | 是       | 您要使用来访问HDFS集群的NameNode的帐户的用户名。               |
  | password                    | 是       | 您要使用来访问HDFS集群的NameNode的帐户的密码。               |

- 使用IAM用户身份验证访问AWS S3:

  ```SQL
  "aws.s3.access_key" = "xxxxxxxxxx",
  "aws.s3.secret_key" = "yyyyyyyyyy",
  "aws.s3.region" = "<s3_region>"
  ```

  | **键**          | **必需** | **描述**                                              |
  | -------------- | -------- | ----------------------------------------------------- |
  | aws.s3.access_key | 是       | 您可以用来访问Amazon S3存储桶的访问密钥ID。               |
  | aws.s3.secret_key | 是       | 您可以用来访问Amazon S3存储桶的秘密访问密钥。               |
  | aws.s3.region      | 是       | 您的AWS S3存储桶所在的地区。示例: `us-west-2`。                   |

- 使用IAM用户身份验证访问GCS:

  ```SQL
  "fs.s3a.access.key" = "xxxxxxxxxx",
  "fs.s3a.secret.key" = "yyyyyyyyyy",
  "fs.s3a.endpoint" = "<gcs_endpoint>"
  ```

  | **键**           | **必需** | **描述**                                             |
  | --------------- | -------- | ---------------------------------------------------- |
  | fs.s3a.access.key | 是       | 您可以用来访问GCS存储桶的访问密钥ID。                    |
  | fs.s3a.secret.key | 是       | 您可以用来访问GCS存储桶的秘密访问密钥。                    |
  | fs.s3a.endpoint   | 是       | 您可以用来访问GCS存储桶的终端点。示例: `storage.googleapis.com`。 |

- 使用Shared Key访问Azure Blob Storage:

  ```SQL
  "azure.blob.storage_account" = "<storage_account>",
  "azure.blob.shared_key" = "<shared_key>"
  ```

  | **键**                    | **必需** | **描述**                                                     |
  | ----------------------- | -------- | ------------------------------------------------------------ |
  | azure.blob.storage_account | 是       | Azure Blob Storage帐户的名称。                                    |
  | azure.blob.shared_key      | 是       | 您可以用来访问Azure Blob Storage帐户的共享密钥。                     |

### columns_from_path

从v3.2版本开始，StarRocks可以从文件路径中提取键值对的值作为某列的值。

```SQL
"columns_from_path" = "<column_name> [, ...]"
```

假设数据文件**file1**存储在路径的格式为`/geo/country=US/city=LA/`下。您可以将`columns_from_path`参数指定为`"columns_from_path" = "country, city"`，以提取文件路径中的地理信息作为返回的列的值。有关更多说明，请参见示例4。
从v3.2开始，FILES（）支持再远程存储中定义可写文件以卸载数据。有关详细说明，请参见[使用INSERT INTO FILES卸载数据](../../../unloading/unload_using_insert_into_files.md)。

```sql
-- 从v3.2开始支持。
unload_data_param :: =
    "compression" = "compression_method"
    "max_file_size" = "file_size"
    "partition_by" = "column_name [, ...]"
    "single" = {"true" | "false"}
```


| **关键字**         | **必选**   | **描述**                                       |
| ---------------- | ---------- | ---------------------------------------- |
| compression      | 是         | 卸载数据时要使用的压缩方法。有效值：<ul><li>`uncompressed`：不使用压缩算法。</li><li>`gzip`：使用gzip压缩算法。</li><li>`brotli`：使用Brotli压缩算法。</li><li>`zstd`：使用Zstd压缩算法。</li><li>`lz4`：使用LZ4压缩算法。</li></ul>                |
| max_file_size    | 否         | 当数据卸载到多个文件时，每个数据文件的最大大小。默认值：`1GB`。单位：B、KB、MB、GB、TB和PB。 |
| partition_by     | 否         | 用于将数据文件分区至不同存储路径的列列表。多个列以逗号（,）分隔。 FILES（）提取指定列的键/值信息，并将数据文件存储在具有提取的键/值对的存储路径下。有关详细说明，请参见示例5。 |
| single           | 否         | 是否将数据卸载到单个文件。有效值：<ul><li>`true`：数据存储在单个数据文件中。</li><li>`false`（默认）：如果 `max_file_size` 达到，数据将存储在多个文件中。</li></ul>             |

> **注意**
>
> 不能同时指定 `max_file_size` 和 `single`。

## 使用说明

从v3.2开始，FILES（）还支持复杂数据类型，包括ARRAY、JSON、MAP和STRUCT，以及基本数据类型。

## 示例

示例1：从AWS S3存储桶`inserttest`中的Parquet文件**parquet/par-dup.parquet**查询数据：

```plain
MySQL > SELECT * FROM FILES(
     "path" = "s3://inserttest/parquet/par-dup.parquet",
     "format" = "parquet",
     "aws.s3.access_key" = "XXXXXXXXXX",
     "aws.s3.secret_key" = "YYYYYYYYYY",
     "aws.s3.region" = "us-west-2"
);
+------+---------------------------------------------------------+
| c1   | c2                                                      |
+------+---------------------------------------------------------+
|    1 | {"1": "key", "1": "1", "111": "1111", "111": "aaaa"}    |
|    2 | {"2": "key", "2": "NULL", "222": "2222", "222": "bbbb"} |
+------+---------------------------------------------------------+
2 rows in set (22.335 sec)
```

示例2：将AWS S3存储桶`inserttest`中的Parquet文件**parquet/insert_wiki_edit_append.parquet**中的数据行插入到表`insert_wiki_edit`中：

```plain
MySQL > INSERT INTO insert_wiki_edit
    SELECT * FROM FILES(
        "path" = "s3://inserttest/parquet/insert_wiki_edit_append.parquet",
        "format" = "parquet",
        "aws.s3.access_key" = "XXXXXXXXXX",
        "aws.s3.secret_key" = "YYYYYYYYYY",
        "aws.s3.region" = "us-west-2"
);
Query OK, 2 rows affected (23.03 sec)
{'label':'insert_d8d4b2ee-ac5c-11ed-a2cf-4e1110a8f63b', 'status':'VISIBLE', 'txnId':'2440'}
```

示例3：创建名为`ctas_wiki_edit`的表，并将AWS S3存储桶`inserttest`中的Parquet文件**parquet/insert_wiki_edit_append.parquet**中的数据行插入到该表中：

```plain
MySQL > CREATE TABLE ctas_wiki_edit AS
    SELECT * FROM FILES(
        "path" = "s3://inserttest/parquet/insert_wiki_edit_append.parquet",
        "format" = "parquet",
        "aws.s3.access_key" = "XXXXXXXXXX",
        "aws.s3.secret_key" = "YYYYYYYYYY",
        "aws.s3.region" = "us-west-2"
);
Query OK, 2 rows affected (22.09 sec)
{'label':'insert_1a217d70-2f52-11ee-9e4a-7a563fb695da', 'status':'VISIBLE', 'txnId':'3248'}
```

示例4：从Parquet文件**/geo/country=US/city=LA/file1.parquet**（仅包含两列 - `id` 和 `user`）查询数据，并以其路径中的键/值信息作为返回列。

```plain
SELECT * FROM FILES(
    "path" = "hdfs://xxx.xx.xxx.xx:9000/geo/country=US/city=LA/file1.parquet",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "xxxxx",
    "password" = "xxxxx",
    "columns_from_path" = "country, city"
);
+------+---------+---------+------+
| id   | user    | country | city |
+------+---------+---------+------+
|    1 | richard | US      | LA   |
|    2 | amber   | US      | LA   |
+------+---------+---------+------+
2 rows in set (3.84 sec)
```

示例5：将`sales_records`中的所有数据行作为多个Parquet文件卸载到HDFS集群的路径**/unload/partitioned/**。这些文件存储在不同的子路径中，以`sales_time`列的值来区分。

```sql
INSERT INTO 
FILES(
    "path" = "hdfs://xxx.xx.xxx.xx:9000/unload/partitioned/",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "xxxxx",
    "password" = "xxxxx",
    "compression" = "lz4",
    "partition_by" = "sales_time"
)
SELECT * FROM sales_records;
```