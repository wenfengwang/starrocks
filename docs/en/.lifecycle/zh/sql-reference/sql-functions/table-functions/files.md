---
displayed_sidebar: English
---

# 文件

## 描述

定义远程存储中的数据文件。

从 v3.1.0 开始，StarRocks 支持使用表函数 FILES() 定义远程存储中的只读文件。它可以使用文件的路径相关属性访问远程存储，推断文件中数据的表模式，并返回数据行。您可以使用 [SELECT](../../sql-statements/data-manipulation/SELECT.md) 直接查询数据行，使用 [INSERT](../../sql-statements/data-manipulation/INSERT.md) 将数据行加载到现有表中，或者使用 [CREATE TABLE AS SELECT](../../sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md) 创建新表并将数据行加载到其中。

从 v3.2.0 开始，FILES() 支持将数据写入远程存储中的文件。您可以使用 [INSERT INTO FILES() 将 StarRocks 中的数据卸载到远程存储](../../../unloading/unload_using_insert_into_files.md)。

目前，FILES() 函数支持以下数据源和文件格式：

- **数据源：**
  - HDFS
  - AWS S3
  - Google Cloud Storage
  - 其他兼容 S3 的存储系统
  - Microsoft Azure Blob 存储
- **文件格式：**
  - Parquet
  - ORC（当前不支持卸载数据）

## 语法

- **数据加载**：

  ```SQL
  FILES( data_location , data_format [, StorageCredentialParams ] [, columns_from_path ] )
  ```

- **数据卸载**：

  ```SQL
  FILES( data_location , data_format [, StorageCredentialParams ] , unload_data_param )
  ```

## 参数

所有参数都是 `"key" = "value"` 成对存在。

### data_location

用于访问文件的 URI。您可以指定路径或文件。

- 要访问 HDFS，您需要将此参数指定为：

  ```SQL
  "path" = "hdfs://<hdfs_host>:<hdfs_port>/<hdfs_path>"
  -- 示例: "path" = "hdfs://127.0.0.1:9000/path/file.parquet"
  ```

- 要访问 AWS S3：

  - 如果您使用 S3 协议，则需要将此参数指定为：

    ```SQL
    "path" = "s3://<s3_path>"
    -- 示例: "path" = "s3://path/file.parquet"
    ```

  - 如果使用 S3A 协议，则需要将此参数指定为：

    ```SQL
    "path" = "s3a://<s3_path>"
    -- 示例: "path" = "s3a://path/file.parquet"
    ```

- 要访问 Google Cloud Storage，您需要将此参数指定为：

  ```SQL
  "path" = "s3a://<gcs_path>"
  -- 示例: "path" = "s3a://path/file.parquet"
  ```

- 要访问 Azure Blob 存储：

  - 如果存储帐户允许通过 HTTP 进行访问，则需要将此参数指定为：

    ```SQL
    "path" = "wasb://<container>@<storage_account>.blob.core.windows.net/<blob_path>"
    -- 示例: "path" = "wasb://testcontainer@testaccount.blob.core.windows.net/path/file.parquet"
    ```
  
  - 如果存储帐户允许通过 HTTPS 进行访问，则需要将此参数指定为：

    ```SQL
    "path" = "wasbs://<container>@<storage_account>.blob.core.windows.net/<blob_path>"
    -- 示例: "path" = "wasbs://testcontainer@testaccount.blob.core.windows.net/path/file.parquet"
    ```

### data_format

数据文件的格式。有效值： `parquet` 和 `orc`。

### StorageCredentialParams

StarRocks 用于访问存储系统的认证信息。

StarRocks 目前支持通过简单认证访问 HDFS，通过 IAM 用户认证访问 AWS S3 和 GCS，以及使用 Shared Key 访问 Azure Blob Storage。

- 使用简单认证访问 HDFS：

  ```SQL
  "hadoop.security.authentication" = "simple",
  "username" = "xxxxxxxxxx",
  "password" = "yyyyyyyyyy"
  ```

  | **键**                        | **必填** | **描述**                                              |
  | ------------------------------ | ------------ | ------------------------------------------------------------ |
  | hadoop.security.authentication | 否           | 认证方法。有效值： `simple`（默认）。 `simple` 表示简单认证，表示无认证。 |
  | username                       | 是           | 要用于访问 HDFS 集群 NameNode 的帐户的用户名。 |
  | password                       | 是           | 要用于访问 HDFS 集群 NameNode 的帐户的密码。 |

- 使用 IAM 用户认证访问 AWS S3：

  ```SQL
  "aws.s3.access_key" = "xxxxxxxxxx",
  "aws.s3.secret_key" = "yyyyyyyyyy",
  "aws.s3.region" = "<s3_region>"
  ```

  | **键**           | **必填** | **描述**                                              |
  | ----------------- | ------------ | ------------------------------------------------------------ |
  | aws.s3.access_key | 是           | 用于访问 Amazon S3 存储桶的访问密钥 ID。 |
  | aws.s3.secret_key | 是           | 用于访问 Amazon S3 存储桶的秘密访问密钥。 |
  | aws.s3.region     | 是           | 您的 AWS S3 存储桶所在的区域。示例： `us-west-2`。 |

- 使用 IAM 用户认证访问 GCS：

  ```SQL
  "fs.s3a.access.key" = "xxxxxxxxxx",
  "fs.s3a.secret.key" = "yyyyyyyyyy",
  "fs.s3a.endpoint" = "<gcs_endpoint>"
  ```

  | **键**           | **必填** | **描述**                                              |
  | ----------------- | ------------ | ------------------------------------------------------------ |
  | fs.s3a.access.key | 是           | 用于访问 GCS 存储桶的访问密钥 ID。 |
  | fs.s3a.secret.key | 是           | 用于访问 GCS 存储桶的秘密访问密钥。|
  | fs.s3a.endpoint   | 是           | 用于访问 GCS 存储桶的终端节点。示例： `storage.googleapis.com`。 |

- 使用 Shared Key 访问 Azure Blob 存储：

  ```SQL
  "azure.blob.storage_account" = "<storage_account>",
  "azure.blob.shared_key" = "<shared_key>"
  ```

  | **键**                    | **必填** | **描述**                                              |
  | -------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.blob.storage_account | 是           | Azure Blob 存储帐户的名称。                  |
  | azure.blob.shared_key      | 是           | 用于访问 Azure Blob 存储帐户的共享密钥。 |

### columns_from_path

从 v3.2 开始，StarRocks 可以从文件路径中提取键值对的值作为列的值。

```SQL
"columns_from_path" = "<column_name> [, ...]"
```

假设数据文件 **file1** 存储在路径格式为 `/geo/country=US/city=LA/` 的路径下。您可以指定 `columns_from_path` 参数为 `"columns_from_path" = "country, city"`，以提取文件路径中的地理信息作为返回的列的值。有关详细说明，请参见示例 4。

<!--

### schema_detect


从 v3.2 开始，FILES() 支持对同一批数据文件进行自动模式检测和联合。StarRocks 首先通过对批处理中随机数据文件的某些数据行进行采样来检测数据的模式。然后，StarRocks 将批处理中所有数据文件中的列进行联合。

您可以使用以下参数配置采样规则：

- `schema_auto_detect_sample_rows`：每个采样数据文件中要扫描的数据行数。范围：[-1, 500]。如果此参数设置为`-1`，则扫描所有数据行。
- `schema_auto_detect_sample_files`：每批中要采样的随机数据文件数。有效值：`1`（默认值）和`-1`。如果此参数设置为`-1`，则扫描所有数据文件。

采样后，StarRocks 会根据以下规则将所有数据文件中的列进行联合：

- 对于具有不同列名称或索引的列，每一列都标识为一个单独的列，并最终返回所有单个列的并集。
- 对于具有相同列名但数据类型不同的列，它们被标识为同一列，但具有更通用的数据类型。例如，如果文件 A 中的列 `col1` 是 INT，但文件 B 中的列是 DECIMAL，则返回的列中使用 DOUBLE。STRING 类型可用于合并所有数据类型。

如果 StarRocks 未能将所有列进行联合，则会生成一个包含错误信息和所有文件模式的模式错误报告。

> **注意**
>
> 单个批次中的所有数据文件必须具有相同的文件格式。

-->

### unload_data_param

从 v3.2 开始，FILES() 支持在远程存储中定义可写文件以进行数据卸载。有关详细说明，请参阅 [使用 INSERT INTO FILES 卸载数据](../../../unloading/unload_using_insert_into_files.md)。

```sql
-- 从 v3.2 开始支持。
unload_data_param::=
    "compression" = "<compression_method>",
    "max_file_size" = "<file_size>",
    "partition_by" = "<column_name> [, ...]",
    "single" = { "true" | "false" }
```


| **键**          | **必填** | **描述**                                              |
| ---------------- | ------------ | ------------------------------------------------------------ |
| compression      | 是          | 卸载数据时使用的压缩方法。有效值：<ul><li>`uncompressed`：不使用压缩算法。</li><li>`gzip`：使用 gzip 压缩算法。</li><li>`brotli`：使用 Brotli 压缩算法。</li><li>`zstd`：使用 Zstd 压缩算法。</li><li>`lz4`：使用 LZ4 压缩算法。</li></ul>                  |
| max_file_size    | 否           | 将数据卸载到多个文件中时每个数据文件的最大大小。默认值：`1GB`。单位：B、KB、MB、GB、TB 和 PB。 |
| partition_by     | 否           | 用于将数据文件分区到不同存储路径的列列表。多列用逗号（,）分隔。FILES() 提取指定列的键/值信息，并将数据文件存储在提取的键/值对的存储路径下。有关进一步说明，请参阅示例 5。 |
| single           | 否           | 是否将数据卸载到单个文件中。有效值：<ul><li>`true`：数据存储在单个数据文件中。</li><li>`false`（默认）：如果达到`max_file_size`，数据将存储在多个文件中。</li></ul>                  |

> **注意**
>
> 不能同时指定`max_file_size`和`single`。

## 使用说明

从 v3.2 开始，FILES() 除了基本数据类型外，还支持复杂的数据类型，包括 ARRAY、JSON、MAP 和 STRUCT。

## 例子

示例 1：从 AWS S3 存储桶中的 Parquet 文件 **parquet/par-dup.parquet** 查询数据：

```Plain
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

示例 2：将 AWS S3 存储桶中 Parquet 文件 **parquet/insert_wiki_edit_append.parquet** 中的数据行插入表 `insert_wiki_edit`：

```Plain
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

示例 3：创建一个名为 `ctas_wiki_edit` 的表，并将 AWS S3 存储桶中的 Parquet 文件 **parquet/insert_wiki_edit_append.parquet** 中的数据行插入该表：

```Plain
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

示例 4：从 Parquet 文件 **/geo/country=US/city=LA/file1.parquet**（仅包含两列 - `id` 和 `user`）中查询数据，并提取其路径中的键/值信息作为返回的列。

```Plain
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

示例 5：将 `sales_records` 中的所有数据行作为多个 Parquet 文件卸载到路径 **/unload/partitioned/** 下。这些文件存储在不同的子路径中，这些子路径由列 `sales_time` 中的值区分。

```SQL
INSERT INTO 
FILES(
    "路径" = "hdfs://xxx.xx.xxx.xx:9000/unload/partitioned/",
    "格式" = "parquet",
    "hadoop.security.authentication" = "simple",
    "用户名" = "xxxxx",
    "密码" = "xxxxx",
    "压缩" = "lz4",
    "按销售时间分区" = "partition_by"
)
SELECT * FROM sales_records;