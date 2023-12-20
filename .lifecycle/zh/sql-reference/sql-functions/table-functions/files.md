---
displayed_sidebar: English
---

# 文件

## 描述

定义远程存储中的数据文件。

从v3.1.0版本开始，StarRocks支持使用`FILES()`表函数在远程存储中定义只读文件。它可以访问远程存储中文件的路径相关属性，推断文件中数据的表结构，并返回数据行。您可以直接使用[SELECT](../../sql-statements/data-manipulation/SELECT.md)查询数据行，使用[INSERT](../../sql-statements/data-manipulation/INSERT.md)将数据行加载到现有表中，或使用[CREATE TABLE AS SELECT](../../sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)创建新表并将数据行加载进去。

从v3.2.0版本开始，FILES()支持将数据写入远程存储中的文件。您可以[使用INSERT INTO FILES()将数据从StarRocks卸载到远程存储](../../../unloading/unload_using_insert_into_files.md)。

目前，FILES()函数支持以下数据源和文件格式：

- **数据源：**
  - HDFS
  - AWS S3
  - Google Cloud Storage
  - 其他兼容S3的存储系统
  - Microsoft Azure Blob Storage
- **文件格式**：
  - Parquet
  - ORC（目前不支持数据卸载）

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

所有参数均以“key”="value"对的形式出现。

### data_location

用于访问文件的URI。您可以指定一个路径或文件。

- 要访问HDFS，您需要将此参数设置为：

  ```SQL
  "path" = "hdfs://<hdfs_host>:<hdfs_port>/<hdfs_path>"
  -- Example: "path" = "hdfs://127.0.0.1:9000/path/file.parquet"
  ```

- 要访问AWS S3：

-   如果您使用的是S3协议，需要将此参数设置为：

    ```SQL
    "path" = "s3://<s3_path>"
    -- Example: "path" = "s3://path/file.parquet"
    ```

-   如果您使用的是S3A协议，需要将此参数设置为：

    ```SQL
    "path" = "s3a://<s3_path>"
    -- Example: "path" = "s3a://path/file.parquet"
    ```

- 要访问Google Cloud Storage，您需要将此参数设置为：

  ```SQL
  "path" = "s3a://<gcs_path>"
  -- Example: "path" = "s3a://path/file.parquet"
  ```

- 要访问Azure Blob Storage：

-   如果您的存储账户允许通过HTTP访问，请将此参数设置为：

    ```SQL
    "path" = "wasb://<container>@<storage_account>.blob.core.windows.net/<blob_path>"
    -- Example: "path" = "wasb://testcontainer@testaccount.blob.core.windows.net/path/file.parquet"
    ```

-   如果您的存储账户允许通过HTTPS访问，请将此参数设置为：

    ```SQL
    "path" = "wasbs://<container>@<storage_account>.blob.core.windows.net/<blob_path>"
    -- Example: "path" = "wasbs://testcontainer@testaccount.blob.core.windows.net/path/file.parquet"
    ```

### data_format

数据文件的格式。有效值为：parquet和orc。

### StorageCredentialParams

StarRocks用于访问您的存储系统的认证信息。

StarRocks目前支持使用简单认证访问HDFS，使用基于IAM用户的认证访问AWS S3和GCS，以及使用共享密钥访问Azure Blob Storage。

- 使用简单认证访问HDFS：

  ```SQL
  "hadoop.security.authentication" = "simple",
  "username" = "xxxxxxxxxx",
  "password" = "yyyyyyyyyy"
  ```

  |键|必填|说明|
|---|---|---|
  |hadoop.security.authentication|否|身份验证方法。有效值：简单（默认）。 simple代表简单认证，即不认证。|
  |用户名|是|要用来访问HDFS集群NameNode的帐户的用户名。|
  |密码|是|要用于访问HDFS集群NameNode的帐户的密码。|

- 使用基于IAM用户的认证访问AWS S3：

  ```SQL
  "aws.s3.access_key" = "xxxxxxxxxx",
  "aws.s3.secret_key" = "yyyyyyyyyy",
  "aws.s3.region" = "<s3_region>"
  ```

  |键|必填|描述|
|---|---|---|
  |aws.s3.access_key|是|可用于访问 Amazon S3 存储桶的访问密钥 ID。|
  |aws.s3.secret_key|是|可用于访问 Amazon S3 存储桶的秘密访问密钥。|
  |aws.s3.region|是|您的 AWS S3 存储桶所在的区域。示例：us-west-2。|

- 使用基于IAM用户的认证访问GCS：

  ```SQL
  "fs.s3a.access.key" = "xxxxxxxxxx",
  "fs.s3a.secret.key" = "yyyyyyyyyy",
  "fs.s3a.endpoint" = "<gcs_endpoint>"
  ```

  |键|必填|说明|
|---|---|---|
  |fs.s3a.access.key|是|可用于访问 GCS 存储桶的访问密钥 ID。|
  |fs.s3a.secret.key|是|可用于访问 GCS 存储桶的秘密访问密钥。|
  |fs.s3a.endpoint|是|可用于访问 GCS 存储桶的端点。示例：storage.googleapis.com。|

- 使用共享密钥访问Azure Blob Storage：

  ```SQL
  "azure.blob.storage_account" = "<storage_account>",
  "azure.blob.shared_key" = "<shared_key>"
  ```

  |键|必填|说明|
|---|---|---|
  |azure.blob.storage_account|是|Azure Blob 存储帐户的名称。|
  |azure.blob.shared_key|是|可用于访问 Azure Blob 存储帐户的共享密钥。|

### columns_from_path

从v3.2版本开始，StarRocks能够从文件路径中提取键/值对作为列的值。

```SQL
"columns_from_path" = "<column_name> [, ...]"
```

假设数据文件**file1**存储在路径`/geo/country=US/city=LA/`下。您可以将`columns_from_path`参数设置为`"columns_from_path" = "country, city"`，以提取文件路径中的地理信息作为返回列的值。更多说明，请参见示例4。

<!--

### schema_detect

From v3.2 onwards, FILES() supports automatic schema detection and unionization of the same batch of data files. StarRocks first detects the schema of the data by sampling certain data rows of a random data file in the batch. Then, StarRocks unionizes the columns from all the data files in the batch.

You can configure the sampling rule using the following parameters:

- `schema_auto_detect_sample_rows`: the number of data rows to scan in each sampled data file. Range: [-1, 500]. If this parameter is set to `-1`, all data rows are scanned. 
- `schema_auto_detect_sample_files`: the number of random data files to sample in each batch. Valid values: `1` (default) and `-1`. If this parameter is set to `-1`, all data files are scanned.

After the sampling, StarRocks unionizes the columns from all the data files according to these rules:

- For columns with different column names or indices, each column is identified as an individual column, and, eventually, the union of all individual columns is returned.
- For columns with the same column name but different data types, they are identified as the same column but with a more general data type. For example, if the column `col1` in file A is INT but DECIMAL in file B, DOUBLE is used in the returned column. The STRING type can be used to unionize all data types.

If StarRocks fails to unionize all the columns, it generates a schema error report that includes the error information and all the file schemas.

> **CAUTION**
>
> All data files in a single batch must be of the same file format.


### unload_data_param

从v3.2版本开始，FILES()支持定义远程存储中的可写文件，以便进行数据卸载。详细说明，请参见[Unload data using INSERT INTO FILES](../../../unloading/unload_using_insert_into_files.md)。

```sql
-- Supported from v3.2 onwards.
unload_data_param::=
    "compression" = "<compression_method>",
    "max_file_size" = "<file_size>",
    "partition_by" = "<column_name> [, ...]" 
    "single" = { "true" | "false" } 
```

|键|必填|说明|
|---|---|---|
|压缩|是|卸载数据时使用的压缩方法。有效值：uncompressed：不使用压缩算法。gzip：使用gzip压缩算法。brotli：使用Brotli压缩算法。zstd：使用Zstd压缩算法。lz4：使用LZ4压缩算法。|
|max_file_size|No|数据卸载到多个文件时每个数据文件的最大大小。默认值：1GB。单位：B、KB、MB、GB、TB 和 PB。|
|partition_by|No|用于将数据文件分区到不同存储路径的列的列表。多个列之间用逗号 (,) 分隔。 FILES()提取指定列的键/值信息，并将数据文件存储在提取的键/值对的存储路径下。如需进一步说明，请参阅示例 5。|
|single|否|是否将数据卸载到单个文件中。有效值：true：数据存储在单个数据文件中。false（默认）：如果达到 max_file_size，数据存储在多个文件中。|

> **注意事项**
> 您不能同时指定`max_file_size`和`single`。

## 使用说明

从v3.2版本开始，FILES()除了支持基本数据类型外，还支持包括ARRAY、JSON、MAP和STRUCT在内的复杂数据类型。

## 示例

示例 1: 查询数据来自AWS S3存储桶 `inserttest` 中的Parquet文件 **parquet/par-dup.parquet**:

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

示例 2: 将数据行从Parquet文件**parquet/insert_wiki_edit_append.parquet**插入到AWS S3存储桶`inserttest`中的表`insert_wiki_edit`中：

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

示例3：创建名为`ctas_wiki_edit`的表，并将Parquet文件**parquet/insert_wiki_edit_append.parquet**中的数据行插入到AWS S3存储桶`inserttest`中的表中：

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

示例 4: 查询数据来自Parquet文件 **/geo/country=US/city=LA/file1.parquet**（其中只包含两列 - `id` 和 `user`），并提取路径中的键/值信息作为返回的列。

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

示例5：将`sales_records`中的所有数据行作为多个Parquet文件卸载到HDFS集群的**/unload/partitioned/**路径下。这些文件根据`sales_time`列中的值存储在不同的子路径中。

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
