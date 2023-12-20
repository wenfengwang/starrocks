---
displayed_sidebar: English
---

# FILES

## Description

Defines data files in remote storage.

从 v3.1.0 版本开始，StarRocks 支持使用表函数 FILES() 在远程存储中定义只读文件。它可以通过文件的路径相关属性访问远程存储，推断文件中数据的表结构，并返回数据行。您可以直接使用 [SELECT](../../sql-statements/data-manipulation/SELECT.md) 查询数据行，使用 [INSERT](../../sql-statements/data-manipulation/INSERT.md) 将数据行加载到现有表中，或者使用 [CREATE TABLE AS SELECT](../../sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md) 创建一个新表并将数据行加载进去。

从 v3.2.0 版本开始，FILES() 支持将数据写入远程存储中的文件。您可以[使用 INSERT INTO FILES() 将数据从 StarRocks 卸载到远程存储](../../../unloading/unload_using_insert_into_files.md)。

目前，FILES() 函数支持以下数据源和文件格式：

- **数据源：**
  - HDFS
  - AWS S3
  - Google Cloud Storage
  - 其他兼容 S3 的存储系统
  - Microsoft Azure Blob Storage
- **文件格式:**
  - Parquet
  - ORC（目前不支持卸载数据）

## Syntax

- **数据加载**：

  ```SQL
  FILES( data_location , data_format [, StorageCredentialParams ] [, columns_from_path ] )
  ```

- **数据卸载**：

  ```SQL
  FILES( data_location , data_format [, StorageCredentialParams ] , unload_data_param )
  ```

## Parameters

所有参数都在 `"key" = "value"` 对中。

### data_location

用于访问文件的 URI。您可以指定路径或文件。

- 要访问 HDFS，需要指定该参数为：

  ```SQL
  "path" = "hdfs://<hdfs_host>:<hdfs_port>/<hdfs_path>"
  -- 示例: "path" = "hdfs://127.0.0.1:9000/path/file.parquet"
  ```

- 访问 AWS S3：

-   如果使用 S3 协议，则需要指定该参数为：

    ```SQL
    "path" = "s3://<s3_path>"
    -- 示例: "path" = "s3://path/file.parquet"
    ```

-   如果使用 S3A 协议，则需要指定该参数为：

    ```SQL
    "path" = "s3a://<s3_path>"
    -- 示例: "path" = "s3a://path/file.parquet"
    ```

- 要访问 Google Cloud Storage，您需要将此参数指定为：

  ```SQL
  "path" = "s3a://<gcs_path>"
  -- 示例: "path" = "s3a://path/file.parquet"
  ```

- 要访问 Azure Blob Storage：

-   如果您的存储账户允许通过 HTTP 访问，则需要将此参数指定为：

    ```SQL
    "path" = "wasb://<container>@<storage_account>.blob.core.windows.net/<blob_path>"
    -- 示例: "path" = "wasb://testcontainer@testaccount.blob.core.windows.net/path/file.parquet"
    ```

-   如果您的存储账户允许通过 HTTPS 访问，则需要将此参数指定为：

    ```SQL
    "path" = "wasbs://<container>@<storage_account>.blob.core.windows.net/<blob_path>"
    -- 示例: "path" = "wasbs://testcontainer@testaccount.blob.core.windows.net/path/file.parquet"
    ```

### data_format

数据文件的格式。有效值：`parquet` 和 `orc`。

### StorageCredentialParams

StarRocks 用于访问您的存储系统的认证信息。

StarRocks 目前支持通过简单认证访问 HDFS，通过基于 IAM 用户的认证访问 AWS S3 和 GCS，以及通过 Shared Key 访问 Azure Blob Storage。

- 使用简单认证访问 HDFS：

  ```SQL
  "hadoop.security.authentication" = "simple",
  "username" = "xxxxxxxxxx",
  "password" = "yyyyyyyyyy"
  ```

  |**键**|**必填**|**描述**|
|---|---|---|
  |hadoop.security.authentication|否|认证方法。有效值：`simple`（默认）。`simple` 代表简单认证，即不进行认证。|
  |username|是|要用来访问 HDFS 集群 NameNode 的账户的用户名。|
  |password|是|要用来访问 HDFS 集群 NameNode 的账户的密码。|

- 使用 IAM 用户基础认证访问 AWS S3：

  ```SQL
  "aws.s3.access_key" = "xxxxxxxxxx",
  "aws.s3.secret_key" = "yyyyyyyyyy",
  "aws.s3.region" = "<s3_region>"
  ```

  |**键**|**必填**|**描述**|
|---|---|---|
  |aws.s3.access_key|是|用于访问 Amazon S3 存储桶的 Access Key ID。|
  |aws.s3.secret_key|是|用于访问 Amazon S3 存储桶的 Secret Access Key。|
  |aws.s3.region|是|您的 AWS S3 存储桶所在的区域。例如：`us-west-2`。|

- 使用 IAM 用户基础认证访问 GCS：

  ```SQL
  "fs.s3a.access.key" = "xxxxxxxxxx",
  "fs.s3a.secret.key" = "yyyyyyyyyy",
  "fs.s3a.endpoint" = "<gcs_endpoint>"
  ```

  |**键**|**必填**|**描述**|
|---|---|---|
  |fs.s3a.access.key|是|用于访问 GCS 存储桶的 Access Key ID。|
  |fs.s3a.secret.key|是|用于访问 GCS 存储桶的 Secret Access Key。|
  |fs.s3a.endpoint|是|用于访问 GCS 存储桶的端点。例如：`storage.googleapis.com`。|

- 使用 Shared Key 访问 Azure Blob Storage：

  ```SQL
  "azure.blob.storage_account" = "<storage_account>",
  "azure.blob.shared_key" = "<shared_key>"
  ```

  |**键**|**必填**|**描述**|
|---|---|---|
  |azure.blob.storage_account|是|Azure Blob Storage 账户的名称。|
  |azure.blob.shared_key|是|用于访问 Azure Blob Storage 账户的 Shared Key。|

### columns_from_path

从 v3.2 版本开始，StarRocks 可以从文件路径中提取键/值对的值作为列的值。

```SQL
"columns_from_path" = "<column_name> [, ...]"
```

假设数据文件 **file1** 存储在 `/geo/country=US/city=LA/` 格式的路径下。您可以将 `columns_from_path` 参数指定为 `"columns_from_path" = "country, city"`，以提取文件路径中的地理信息作为返回的列的值。更多说明，请参见示例 4。
```
```markdown
<!--

### schema_detect

从 v3.2 版本开始，FILES() 支持自动检测数据模式并合并同一批次的数据文件。StarRocks 首先通过对批次中随机数据文件的某些数据行进行采样来检测数据的模式。然后，StarRocks 将合并批次中所有数据文件的列。

您可以使用以下参数配置采样规则：

- `schema_auto_detect_sample_rows`: 在每个采样数据文件中扫描的数据行数。范围：[-1, 500]。如果此参数设置为 `-1`，则扫描所有数据行。
- `schema_auto_detect_sample_files`: 在每个批次中采样的随机数据文件的数量。有效值：`1`（默认）和 `-1`。如果此参数设置为 `-1`，则扫描所有数据文件。

采样完成后，StarRocks 根据以下规则合并所有数据文件的列：

- 对于具有不同列名或索引的列，每个列被识别为一个独立的列，并最终返回所有独立列的并集。
- 对于具有相同列名但不同数据类型的列，它们被识别为同一列，但具有更通用的数据类型。例如，如果文件 A 中的列 `col1` 是 INT 类型，而文件 B 中是 DECIMAL 类型，则在返回的列中使用 DOUBLE 类型。STRING 类型可用于合并所有数据类型。

如果 StarRocks 无法合并所有列，它将生成一个包含错误信息和所有文件模式的模式错误报告。

> **注意**
>
> 单个批次中的所有数据文件必须是相同的文件格式。


### unload_data_param

从 v3.2 版本开始，FILES() 支持在远程存储中定义可写文件以进行数据卸载。有关详细说明，请参阅[使用 INSERT INTO FILES 卸载数据](../../../unloading/unload_using_insert_into_files.md)。

```sql
-- 从 v3.2 版本开始支持。
unload_data_param::=
    "compression" = "<compression_method>",
    "max_file_size" = "<file_size>",
    "partition_by" = "<column_name> [, ...]" 
    "single" = { "true" | "false" } 
```

|**键**|**必填**|**说明**|
|---|---|---|
|compression|是|卸载数据时使用的压缩方法。有效值：<ul><li>`uncompressed`：不使用压缩算法。</li><li>`gzip`：使用 gzip 压缩算法。</li><li>`brotli`：使用 Brotli 压缩算法。</li><li>`zstd`：使用 Zstd 压缩算法。</li><li>`lz4`：使用 LZ4 压缩算法。</li></ul>|
|max_file_size|否|数据卸载到多个文件时每个数据文件的最大大小。默认值：`1GB`。单位：B、KB、MB、GB、TB 和 PB。|
|partition_by|否|用于将数据文件分区到不同存储路径的列的列表。多个列之间用逗号 (,) 分隔。FILES() 提取指定列的键/值信息，并将数据文件存储在提取的键/值对的存储路径下。有关更多说明，请参阅示例 5。|
|single|否|是否将数据卸载到单个文件中。有效值：<ul><li>`true`：数据存储在单个数据文件中。</li><li>`false`（默认）：如果达到 `max_file_size`，数据存储在多个文件中。</li></ul>|

> **警告**
> 您不能同时指定 `max_file_size` 和 `single`。

## 使用说明

从 v3.2 版本开始，FILES() 除了基本数据类型外，还支持 ARRAY、JSON、MAP、STRUCT 等复杂数据类型。

## 示例

示例 1：查询 AWS S3 存储桶 `inserttest` 中 Parquet 文件 **parquet/par-dup.parquet** 的数据：

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
2 行在集合中 (22.335 秒)
```

示例 2：将 AWS S3 存储桶 `inserttest` 中的 Parquet 文件 **parquet/insert_wiki_edit_append.parquet** 的数据行插入到表 `insert_wiki_edit` 中：

```Plain
MySQL > INSERT INTO insert_wiki_edit
    SELECT * FROM FILES(
        "path" = "s3://inserttest/parquet/insert_wiki_edit_append.parquet",
        "format" = "parquet",
        "aws.s3.access_key" = "XXXXXXXXXX",
        "aws.s3.secret_key" = "YYYYYYYYYY",
        "aws.s3.region" = "us-west-2"
);
Query OK, 2 行受影响 (23.03 秒)
{'label':'insert_d8d4b2ee-ac5c-11ed-a2cf-4e1110a8f63b', 'status':'VISIBLE', 'txnId':'2440'}
```

示例 3：创建名为 `ctas_wiki_edit` 的表，并将 AWS S3 存储桶 `inserttest` 中的 Parquet 文件 **parquet/insert_wiki_edit_append.parquet** 的数据行插入到该表中：

```Plain
MySQL > CREATE TABLE ctas_wiki_edit AS
    SELECT * FROM FILES(
        "path" = "s3://inserttest/parquet/insert_wiki_edit_append.parquet",
        "format" = "parquet",
        "aws.s3.access_key" = "XXXXXXXXXX",
        "aws.s3.secret_key" = "YYYYYYYYYY",
        "aws.s3.region" = "us-west-2"
);
Query OK, 2 行受影响 (22.09 秒)
{'label':'insert_1a217d70-2f52-11ee-9e4a-7a563fb695da', 'status':'VISIBLE', 'txnId':'3248'}
```

示例 4: 查询 Parquet 文件 **/geo/country=US/city=LA/file1.parquet** 的数据（该文件仅包含两列 - `id` 和 `user`），并提取其路径中的键/值信息作为返回的列。

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
2 行在集合中 (3.84 秒)
```

示例 5：将 `sales_records` 中的所有数据行作为多个 Parquet 文件卸载到 HDFS 集群中的 **/unload/partitioned/** 路径下。这些文件根据列 `sales_time` 的值存储在不同的子路径中。

```SQL
INSERT INTO 
FILES(
    "path" = "hdfs://xxx.xx.xxx.xx:9000/unload/partitioned/",
```
```markdown
# 文件

## 描述

定义远程存储中的数据文件。

从v3.1.0版本开始，StarRocks支持使用表函数FILES()定义远程存储中的只读文件。它可以通过文件的路径相关属性访问远程存储，推断文件中数据的表结构，并返回数据行。您可以直接使用[SELECT](../../sql-statements/data-manipulation/SELECT.md)查询数据行，使用[INSERT](../../sql-statements/data-manipulation/INSERT.md)将数据行加载到现有表中，或者使用[CREATE TABLE AS SELECT](../../sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)创建新表并将数据行加载进去。

从v3.2.0版本开始，FILES()支持将数据写入远程存储中的文件。您可以[使用INSERT INTO FILES()将数据从StarRocks卸载到远程存储](../../../unloading/unload_using_insert_into_files.md)。

目前，FILES()函数支持以下数据源和文件格式：

- **数据源：**
  - HDFS
  - AWS S3
  - Google Cloud Storage
  - 其他兼容S3的存储系统
  - Microsoft Azure Blob Storage
- **文件格式：**
  - Parquet
  - ORC（目前不支持卸载数据）

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

所有参数都是`"key" = "value"`形式的配对。

### data_location

用于访问文件的URI。您可以指定路径或文件。

- 要访问HDFS，需要将此参数指定为：

  ```SQL
  "path" = "hdfs://<hdfs_host>:<hdfs_port>/<hdfs_path>"
  -- 示例："path" = "hdfs://127.0.0.1:9000/path/file.parquet"
  ```

- 要访问AWS S3：

-   如果使用S3协议，需要将此参数指定为：

    ```SQL
    "path" = "s3://<s3_path>"
    -- 示例："path" = "s3://path/file.parquet"
    ```

-   如果使用S3A协议，需要将此参数指定为：

    ```SQL
    "path" = "s3a://<s3_path>"
    -- 示例："path" = "s3a://path/file.parquet"
    ```

- 要访问Google Cloud Storage，需要将此参数指定为：

  ```SQL
  "path" = "s3a://<gcs_path>"
  -- 示例："path" = "s3a://path/file.parquet"
  ```

- 要访问Azure Blob Storage：

-   如果您的存储账户允许通过HTTP访问，需要将此参数指定为：

    ```SQL
    "path" = "wasb://<container>@<storage_account>.blob.core.windows.net/<blob_path>"
    -- 示例："path" = "wasb://testcontainer@testaccount.blob.core.windows.net/path/file.parquet"
    ```

-   如果您的存储账户允许通过HTTPS访问，需要将此参数指定为：

    ```SQL
    "path" = "wasbs://<container>@<storage_account>.blob.core.windows.net/<blob_path>"
    -- 示例："path" = "wasbs://testcontainer@testaccount.blob.core.windows.net/path/file.parquet"
    ```

### data_format

数据文件的格式。有效值：`parquet`和`orc`。

### StorageCredentialParams

StarRocks用于访问存储系统的认证信息。

StarRocks目前支持使用简单认证访问HDFS，使用基于IAM用户的认证访问AWS S3和GCS，以及使用Shared Key访问Azure Blob Storage。

- 使用简单认证访问HDFS：

  ```SQL
  "hadoop.security.authentication" = "simple",
  "username" = "xxxxxxxxxx",
  "password" = "yyyyyyyyyy"
  ```

  |**键**|**必需**|**描述**|
|---|---|---|
  |hadoop.security.authentication|否|认证方法。有效值：`simple`（默认）。`simple`表示简单认证，即无认证。|
  |username|是|您想要用来访问HDFS集群NameNode的账户用户名。|
  |password|是|您想要用来访问HDFS集群NameNode的账户密码。|

- 使用基于IAM用户的认证访问AWS S3：

  ```SQL
  "aws.s3.access_key" = "xxxxxxxxxx",
  "aws.s3.secret_key" = "yyyyyyyyyy",
  "aws.s3.region" = "<s3_region>"
  ```

  |**键**|**必需**|**描述**|
|---|---|---|
  |aws.s3.access_key|是|您可以用来访问Amazon S3桶的Access Key ID。|
  |aws.s3.secret_key|是|您可以用来访问Amazon S3桶的Secret Access Key。|
  |aws.s3.region|是|您的AWS S3桶所在的区域。示例：`us-west-2`。|

- 使用基于IAM用户的认证访问GCS：

  ```SQL
  "fs.s3a.access.key" = "xxxxxxxxxx",
  "fs.s3a.secret.key" = "yyyyyyyyyy",
  "fs.s3a.endpoint" = "<gcs_endpoint>"
  ```

  |**键**|**必需**|**描述**|
|---|---|---|
  |fs.s3a.access.key|是|您可以用来访问GCS桶的Access Key ID。|
  |fs.s3a.secret.key|是|您可以用来访问GCS桶的Secret Access Key。|
  |fs.s3a.endpoint|是|您可以用来访问GCS桶的端点。示例：`storage.googleapis.com`。|

- 使用Shared Key访问Azure Blob Storage：

  ```SQL
  "azure.blob.storage_account" = "<storage_account>",
  "azure.blob.shared_key" = "<shared_key>"
  ```

  |**键**|**必需**|**描述**|
|---|---|---|
  |azure.blob.storage_account|是|Azure Blob Storage账户的名称。|
  |azure.blob.shared_key|是|您可以用来访问Azure Blob Storage账户的Shared Key。|

### columns_from_path

从v3.2版本开始，StarRocks可以从文件路径中提取键/值对的值作为列的值。

```SQL
"columns_from_path" = "<column_name> [, ...]"
```

假设数据文件**file1**存储在格式为`/geo/country=US/city=LA/`的路径下。您可以将`columns_from_path`参数指定为`"columns_from_path" = "country, city"`，以将文件路径中的地理信息提取为返回的列的值。更多指导，请参见示例4。

<!--

### schema_detect

从v3.2版本开始，FILES()支持自动检测同一批数据文件的模式并进行合并。StarRocks首先通过对批次中随机数据文件的某些数据行进行采样来检测数据的模式。然后，StarRocks将批次中所有数据文件的列进行合并。

您可以使用以下参数配置采样规则：

- `schema_auto_detect_sample_rows`：在每个采样数据文件中扫描的数据行数。范围：[-1, 500]。如果此参数设置为`-1`，则扫描所有数据行。
- `schema_auto_detect_sample_files`：在每个批次中采样的随机数据文件数。有效值：`1`（默认）和`-1`。如果此参数设置为`-1`，则扫描所有数据文件。

采样完成后，StarRocks根据以下规则合并所有数据文件的列：

- 对于具有不同列名或索引的列，每个列被识别为一个单独的列，并最终返回所有单独列的并集。
- 对于具有相同列名但不同数据类型的列，它们被识别为相同的列，但具有更通用的数据类型。例如，如果文件A中的列`col1`是INT，但文件B中是DECIMAL，则在返回的列中使用DOUBLE。STRING类型可以用来合并所有数据类型。

如果StarRocks无法合并所有列，它会生成一个包含错误信息和所有文件模式的模式错误报告。

> **注意**
>
> 单个批次中的所有数据文件必须是相同的文件格式。


### unload_data_param

从v3.2版本开始，FILES()支持在远程存储中定义可写文件以进行数据卸载。详细指导，请参见[使用INSERT INTO FILES卸载数据](../../../unloading/unload_using_insert_into_files.md)。

```sql
-- 从v3.2版本开始支持。
unload_data_param::=
    "compression" = "<compression_method>",
    "max_file_size" = "<file_size>",
    "partition_by" = "<column_name> [, ...]" 
    "single" = { "true" | "false" } 
```

|**键**|**必需**|**描述**|
|---|---|---|
|compression|是|卸载数据时使用的压缩方法。有效值：<ul><li>`uncompressed`：不使用压缩算法。</li><li>`gzip`：使用gzip压缩算法。</li><li>`brotli`：使用Brotli压缩算法。</li><li>`zstd`：使用Zstd压缩算法。</li><li>`lz4`：使用LZ4压缩算法。</li></ul>|
|max_file_size|否|将数据卸载到多个文件时，每个数据文件的最大大小。默认值：`1GB`。单位：B, KB, MB, GB, TB, PB。|
|partition_by|否|用于将数据文件分区到不同存储路径的列列表。多个列用逗号(,)分隔。FILES()提取指定列的键/值信息，并将数据文件存储在具有提取的键/值对的存储路径下。更多指导，请参见示例5。|
|single|否|是否将数据卸载到单个文件中。有效值：<ul><li>`true`：数据存储在单个数据文件中。</li><li>`false`（默认）：如果达到`max_file_size`，则数据存储在多个文件中。</li></ul>|

> **注意**
> 您不能同时指定`max_file_size`和`single`。

## 使用说明

从v3.2版本开始，FILES()进一步支持复杂数据类型，包括ARRAY, JSON, MAP和STRUCT，除了基本数据类型。

## 示例

示例1：查询AWS S3桶`inserttest`中Parquet文件**parquet/par-dup.parquet**的数据：

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
2行在集合中 (22.335秒)
```

示例2：将AWS S3桶`inserttest`中Parquet文件**parquet/insert_wiki_edit_append.parquet**的数据行插入到表`insert_wiki_edit`中：

```Plain
MySQL > INSERT INTO insert_wiki_edit
    SELECT * FROM FILES(
        "path" = "s3://inserttest/parquet/insert_wiki_edit_append.parquet",
        "format" = "parquet",
        "aws.s3.access_key" = "XXXXXXXXXX",
        "aws.s3.secret_key" = "YYYYYYYYYY",
        "aws.s3.region" = "us-west-2"
);
执行成功，影响2行 (23.03秒)
{'label':'insert_d8d4b2ee-ac5c-11ed-a2cf-4e1110a8f63b', 'status':'VISIBLE', 'txnId':'2440'}
```

示例3：创建一个名为`ctas_wiki_edit`的表，并将AWS S3桶`inserttest`中Parquet文件**parquet/insert_wiki_edit_append.parquet**的数据行插入到该表中：

```Plain
MySQL > CREATE TABLE ctas_wiki_edit AS
    SELECT * FROM FILES(
        "path" = "s3://inserttest/parquet/insert_wiki_edit_append.parquet",
        "format" = "parquet",
        "aws.s3.access_key" = "XXXXXXXXXX",
        "aws.s3.secret_key" = "YYYYYYYYYY",
        "aws.s3.region" = "us-west-2"
);
执行成功，影响2行 (22.09秒)
{'label':'insert_1a217d70-2f52-11ee-9e4a-7a563fb695da', 'status':'VISIBLE', 'txnId':'3248'}
```

示例4：查询HDFS中路径为**/geo/country=US/city=LA/file1.parquet**的Parquet文件（该文件仅包含两列 - `id` 和 `user`）的数据，并将其路径中的键/值信息作为返回的列提取。

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
2行在集合中 (3.84秒)
```

示例5：将`sales_records`中的所有数据行作为多个Parquet文件卸载到HDFS集群路径**/unload/partitioned/**下。这些文件根据列`sales_time`的值存储在不同的子路径中。

```SQL
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