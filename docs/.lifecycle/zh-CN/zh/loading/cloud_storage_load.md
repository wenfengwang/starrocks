---
displayed_sidebar: "Chinese"
---

# Import from Cloud Storage

StarRocks supports importing large amounts of data from cloud storage systems in two ways: [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) and [INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md).

Before version 3.0, StarRocks only supported the Broker Load import method. Broker Load is an asynchronous import method, meaning that after you submit the import job, StarRocks will execute the import job asynchronously. You can use `SELECT * FROM information_schema.loads` to view the results of the Broker Load job. This feature is supported starting from version 3.1, please refer to the section "[View Import Jobs](#View-import-jobs)" in this article for details.

Broker Load ensures the atomicity of a single import transaction, which means that multiple data files imported at once will either all succeed or all fail, without the possibility of partial success and partial failure.

In addition, Broker Load also supports data transformation during the import process, as well as data changes through UPSERT and DELETE operations. Please refer to [Implementing Data Transformation During Import](../loading/Etl_in_loading.md) and [Implementing Data Changes Through Import](../loading/Load_to_Primary_Key_tables.md).

> **Note**
>
> Broker Load operation requires INSERT permission on the target table. If your user account does not have INSERT permission, please refer to [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) to grant permissions to users.

Starting from version 3.1, StarRocks adds support for directly importing Parquet or ORC formatted data files from AWS S3 using the INSERT statement and the `FILES` keyword, avoiding the hassle of creating external tables in advance. See [INSERT > Importing External Data Files Directly Using the FILES Keyword](../loading/InsertInto.md#Importing-external-data-files-directly-using-insert-into-select-and-the-files-table-function).

This article mainly introduces how to use [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) to import data from cloud storage systems.

## Supported Data File Formats

Broker Load supports the following data file formats:

- CSV

- Parquet

- ORC

> **Note**
>
> For data in CSV format, the following two points should be noted:
>
> - StarRocks supports setting a maximum length of 50 bytes for UTF-8 encoded strings as column delimiters, including common commas (`,`), tabs, and pipes (`|`).
> - Null values are represented as `\N`. For example, if a data file has three columns and the first and third columns of a certain row of data are `a` and `b` respectively, with the second column having no data, then the second column needs to be represented as `\N` to indicate a null value, written as `a,\N,b`, instead of `a,,b`. `a,,b` indicates that the second column is an empty string.

## Basic Principle

After submitting the import job, the FE generates the corresponding query plan and distributes the query plan to multiple BEs based on the number of available BEs and the size of the source data file, so that each BE is responsible for executing a portion of the import task. During execution, each BE pulls data from the external storage system, preprocesses the data, and then imports the data into StarRocks. After all BEs have completed the import, the FE ultimately determines whether the import job is successful.

The following diagram illustrates the main process of Broker Load:

![Broker Load Process Diagram](../assets/broker_load_how-to-work_zh.png)

## Prepare Data Samples

1. Create two CSV formatted data files, `file1.csv` and `file2.csv`, in the local file system. Both data files contain three columns representing user ID, user name, and user score, as follows:

   - `file1.csv`

     ```Plain
     1,Lily,21
     2,Rose,22
     3,Alice,23
     4,Julia,24
     ```

   - `file2.csv`

     ```Plain
     5,Tony,25
     6,Adam,26
     7,Allen,27
     8,Jacky,28
     ```

2. Upload `file1.csv` and `file2.csv` to the specified path in the cloud storage space. Here, it is assumed that they are uploaded to the `input` folder in the AWS S3 storage space `bucket_s3`, the `input` folder in the Google GCS storage space `bucket_gcs`, the `input` folder in the Aliyun OSS storage space `bucket_oss`, the `input` folder in the Tencent Cloud COS storage space `bucket_cos`, the `input` folder in the Huawei Cloud OBS storage space `bucket_obs`, the `input` folder in other S3 protocol-compatible object storage spaces (such as MinIO) `bucket_minio`, and the specified path in Azure Storage.

3. Log in to the StarRocks database (assuming it is `test_db`) and create two primary key model tables, `table1` and `table2`. Both tables contain three columns `id`, `name`, and `score`, representing user ID, user name, and user score. The primary key is the `id` column, as shown below:

   ```SQL
   CREATE TABLE `table1`
      (
          `id` int(11) NOT NULL COMMENT "User ID",
          `name` varchar(65533) NULL DEFAULT "" COMMENT "User Name",
          `score` int(11) NOT NULL DEFAULT "0" COMMENT "User Score"
      )
          ENGINE=OLAP
          PRIMARY KEY(`id`)
          DISTRIBUTED BY HASH(`id`);

   CREATE TABLE `table2`
      (
          `id` int(11) NOT NULL COMMENT "User ID",
          `name` varchar(65533) NULL DEFAULT "" COMMENT "User Name",
          `score` int(11) NOT NULL DEFAULT "0" COMMENT "User Score"
      )
          ENGINE=OLAP
          PRIMARY KEY(`id`)
          DISTRIBUTED BY HASH(`id`);
   ```

## Import from AWS S3

Note that Broker Load supports accessing AWS S3 through S3 or S3A protocols. Therefore, when importing data from AWS S3, the S3 URI of the target file in the file path (`DATA INFILE`) can use `s3://` or `s3a://` as the prefix.

Additionally, the following commands take CSV format and Instance Profile-based authentication as examples. For parameters to be configured when importing data in other formats and using other authentication methods, refer to [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md).

### Importing a Single Data File Into a Single Table

#### Operation Example

Using the following statement to import the data file `file1.csv` from the `input` folder in the AWS S3 storage space `bucket_s3` into the target table `table1`:

```SQL
LOAD LABEL test_db.label_brokerloadtest_101
(
    DATA INFILE("s3a://bucket_s3/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.region" = "<aws_s3_region>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### Query Data

提交导入作业以后，您可以使用 `SELECT * FROM information_schema.loads` 来查看 Broker Load 作业的结果，该功能自 3.1 版本起支持，具体请参见本文“[查看导入作业](#查看导入作业)”小节。

确认导入作业成功以后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句来查询 `table1` 的数据，如下所示：

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Lily  |    21 |
|    2 | Rose  |    22 |
|    3 | Alice |    23 |
|    4 | Julia |    24 |
+------+-------+-------+
4 rows in set (0.01 sec)
```

### 导入多个数据文件到单表

#### 操作示例

通过如下语句，把AWS S3 存储空间 `bucket_s3` 里 `input` 文件夹内所有数据文件（`file1.csv` 和 `file2.csv`）的数据导入到目标表 `table1`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_102
(
    DATA INFILE("s3a://bucket_s3/input/*")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.region" = "<aws_s3_region>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交导入作业以后，您可以使用 `SELECT * FROM information_schema.loads` 来查看 Broker Load 作业的结果，该功能自 3.1 版本起支持，具体请参见本文“[查看导入作业](#查看导入作业)”小节。

确认导入作业成功以后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句来查询 `table1` 的数据，如下所示：

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Lily  |    21 |
|    2 | Rose  |    22 |
|    3 | Alice |    23 |
|    4 | Julia |    24 |
|    5 | Tony  |    25 |
|    6 | Adam  |    26 |
|    7 | Allen |    27 |
|    8 | Jacky |    28 |
+------+-------+-------+
4 rows in set (0.01 sec)
```

### 导入多个数据文件到多表

#### 操作示例

通过如下语句，把 AWS S3 存储空间 `bucket_s3` 里 `input` 文件夹内数据文件 `file1.csv` 和 `file2.csv` 的数据分别导入到目标表 `table1` 和 `table2`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_103
(
    DATA INFILE("s3a://bucket_s3/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("s3a://bucket_s3/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.region" = "<aws_s3_region>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交导入作业以后，您可以使用 `SELECT * FROM information_schema.loads` 来查看 Broker Load 作业的结果，该功能自 3.1 版本起支持，具体请参见本文“[查看导入作业](#查看导入作业)”小节。

确认导入作业成功以后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句来查询 `table1` 和 `table2` 中的数据：

1. 查询 `table1` 的数据，如下所示：

   ```SQL
   SELECT * FROM table1;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    1 | Lily  |    21 |
   |    2 | Rose  |    22 |
   |    3 | Alice |    23 |
   |    4 | Julia |    24 |
   +------+-------+-------+
   4 rows in set (0.01 sec)
   ```

2. 查询 `table2` 的数据，如下所示：

   ```SQL
   SELECT * FROM table2;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    5 | Tony  |    25 |
   |    6 | Adam  |    26 |
   |    7 | Allen |    27 |
   |    8 | Jacky |    28 |
   +------+-------+-------+
   4 rows in set (0.01 sec)
   ```

## 从 Google GCS 导入

注意，由于 Broker Load 只支持通过 gs 协议访问 Google GCS，因此从 Google GCS 导入数据时，必须确保您在文件路径 (`DATA INFILE`) 中传入的目标文件的 GCS URI 使用 `gs://` 作为前缀。

另外注意，下述命令以 CSV 格式和基于 VM 的认证方式为例。有关如何导入其他格式的数据、以及使用其他认证方式时需要配置的参数，参见 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

### 导入单个数据文件到单表

#### 操作示例

通过如下语句，把 Google GCS 存储空间 `bucket_gcs` 里 `input` 文件夹内数据文件 `file1.csv` 的数据导入到目标表 `table1`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_201
(
    DATA INFILE("gs://bucket_gcs/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "gcp.gcs.use_compute_engine_service_account" = "true"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交导入作业以后，您可以使用 `SELECT * FROM information_schema.loads` 来查看 Broker Load 作业的结果，该功能自 3.1 版本起支持，具体请参见本文“[查看导入作业](#查看导入作业)”小节。

确认导入作业成功以后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句来查询 `table1` 的数据，如下所示：

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Lily  |    21 |
|    2 | Rose  |    22 |
|    3 | Alice |    23 |
|    4 | Julia |    24 |
+------+-------+-------+
4 rows in set (0.01 sec)
```

### 导入多个数据文件到单表

#### 操作示例

通过如下语句，把 Google GCS 存储空间 `bucket_gcs` 里 `input` 文件夹内所有数据文件（`file1.csv` 和 `file2.csv`）的数据导入到目标表 `table1`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_202
(
    DATA INFILE("gs://bucket_gcs/input/*")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "gcp.gcs.use_compute_engine_service_account" = "true"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交导入作业以后，您可以使用 `SELECT * FROM information_schema.loads` 来查看 Broker Load 作业的结果，该功能自 3.1 版本起支持，具体请参见本文“[查看导入作业](#查看导入作业)”小节。

确认导入作业成功以后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句来查询 `table1` 的数据，如下所示：

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Lily  |    21 |
|    2 | Rose  |    22 |
|    3 | Alice |    23 |
|    4 | Julia |    24 |
|    5 | Tony  |    25 |
|    6 | Adam  |    26 |
|    7 | Allen |    27 |
|    8 | Jacky |    28 |
+------+-------+-------+
4 rows in set (0.01 sec)
```

### 导入多个数据文件到多表

#### 操作示例

通过如下语句，把 Google GCS 存储空间 `bucket_gcs` 里 `input` 文件夹内数据文件 `file1.csv` 和 `file2.csv` 的数据分别导入到目标表 `table1` 和 `table2`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_203
(
    DATA INFILE("gs://bucket_gcs/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("gs://bucket_gcs/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "gcp.gcs.use_compute_engine_service_account" = "true"
);
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交导入作业以后，您可以使用 `SELECT * FROM information_schema.loads` 来查看 Broker Load 作业的结果，该功能自 3.1 版本起支持，具体请参见本文“[查看导入作业](#查看导入作业)”小节。

确认导入作业成功以后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句来查询 `table1` 和 `table2` 中的数据：

1. 查询 `table1` 的数据，如下所示：

   ```SQL
   SELECT * FROM table1;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    1 | Lily  |    21 |
   |    2 | Rose  |    22 |
   |    3 | Alice |    23 |
   |    4 | Julia |    24 |
   +------+-------+-------+
   4 rows in set (0.01 sec)
   ```

2. 查询 `table2` 的数据，如下所示：

   ```SQL
   SELECT * FROM table2;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    5 | Tony  |    25 |
   |    6 | Adam  |    26 |
   |    7 | Allen |    27 |
   |    8 | Jacky |    28 |
   +------+-------+-------+
   4 rows in set (0.01 sec)
   ```

## 从 Microsoft Azure Storage 导入

注意，从 Azure Storage 导入数据时，您需要根据所使用的访问协议和存储服务来确定文件路径 (`DATA INFILE`) 的前缀：

- 从 Blob Storage 导入数据时，需要根据使用的访问协议在文件路径 (`DATA INFILE`) 里添加 `wasb://` 或 `wasbs://` 作为前缀：
  - 如果使用 HTTP 协议进行访问，请使用 `wasb://` 作为前缀，例如，`wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`。
  - 如果使用 HTTPS 协议进行访问，请使用 `wasbs://` 作为前缀，例如，`wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`。
- 从 Azure Data Lake Storage Gen1 导入数据时，需要在文件路径 (`DATA INFILE`) 里添加 `adl://` 作为前缀，例如， `adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`。
- 从 Data Lake Storage Gen2 导入数据时，需要根据使用的访问协议在文件路径 (`DATA INFILE`) 里添加 `abfs://` 或 `abfss://` 作为前缀：
  - 如果使用 HTTP 协议进行访问，请使用 `abfs://` 作为前缀，例如，`abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。
  - 如果使用 HTTPS 协议进行访问，请使用 `abfss://` 作为前缀，例如，`abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。

另外注意，下述命令以 CSV 格式、Azure Blob Storage 和基于 Shared Key 的认证方式为例。有关如何导入其他格式的数据、以及使用其他 Azure 对象存储服务和其他认证方式时需要配置的参数，参见 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

### 导入单个数据文件到单表

#### 操作示例

通过如下语句，把 Azure Storage 指定路径下数据文件 `file1.csv` 的数据导入到目标表 `table1`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_301
(
    DATA INFILE("wasb[s]://<container>@<storage_account>.blob.core.windows.net/<path>/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "azure.blob.storage_account" = "<blob_storage_account_name>",
    "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交导入作业以后，您可以使用 `SELECT * FROM information_schema.loads` 来查看 Broker Load 作业的结果，该功能自 3.1 版本起支持，具体请参见本文“[查看导入作业](#查看导入作业)”小节。

确认导入作业成功以后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句来查询 `table1` 的数据，如下所示：

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Lily  |    21 |
|    2 | Rose  |    22 |
|    3 | Alice |    23 |
|    4 | Julia |    24 |
+------+-------+-------+
4 rows in set (0.01 sec)
```

### 导入多个数据文件到单表

#### 操作示例

通过如下语句，把 Azure Storage 指定路径下所有数据文件（`file1.csv` 和 `file2.csv`）的数据导入到目标表 `table1`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_302
(
    DATA INFILE("wasb[s]://<container>@<storage_account>.blob.core.windows.net/<path>/*")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "azure.blob.storage_account" = "<blob_storage_account_name>",
    "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交导入作业以后，您可以使用 `SELECT * FROM information_schema.loads` 来查看 Broker Load 作业的结果，该功能自 3.1 版本起支持，具体请参见本文“[查看导入作业](#查看导入作业)”小节。

确定导入作业成功后，您可以使用[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)语句来查询`table1`中的数据，如下所示：

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Lily  |    21 |
|    2 | Rose  |    22 |
|    3 | Alice |    23 |
|    4 | Julia |    24 |
|    5 | Tony  |    25 |
|    6 | Adam  |    26 |
|    7 | Allen |    27 |
|    8 | Jacky |    28 |
+------+-------+-------+
4 rows in set (0.01 sec)
```

### 导入多个数据文件到多表

#### 操作示例

通过以下语句，将Azure存储指定路径下的数据文件 `file1.csv` 和 `file2.csv`分别导入到目标表`table1`和`table2`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_303
(
    DATA INFILE("wasb[s]://<container>@<storage_account>.blob.core.windows.net/<path>/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("wasb[s]://<container>@<storage_account>.blob.core.windows.net/<path>/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "azure.blob.storage_account" = "<blob_storage_account_name>",
    "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
);
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交导入作业后，您可以使用`SELECT * FROM information_schema.loads`来查看Broker Load作业的结果。该功能自3.1版本起支持。具体请参见本文“[查看导入作业](#查看导入作业)”小节。

确认导入作业成功后，您可以使用[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)语句来查询`table1`和`table2`中的数据：

1. 查询`table1`的数据，如下所示：

   ```SQL
   SELECT * FROM table1;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    1 | Lily  |    21 |
   |    2 | Rose  |    22 |
   |    3 | Alice |    23 |
   |    4 | Julia |    24 |
   +------+-------+-------+
   4 rows in set (0.01 sec)
   ```

2. 查询`table2`的数据，如下所示：

   ```SQL
   SELECT * FROM table2;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    5 | Tony  |    25 |
   |    6 | Adam  |    26 |
   |    7 | Allen |    27 |
   |    8 | Jacky |    28 |
   +------+-------+-------+
   4 rows in set (0.01 sec)
   ```

## 从阿里云 OSS导入

下述命令以CSV格式为例。有关如何导入其他格式的数据，请参见[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

### 导入单个数据文件到单表

#### 操作示例

通过以下语句，将阿里云OSS存储空间`bucket_oss`内`input`文件夹中的数据文件`file1.csv`的数据导入到目标表`table1`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_401
(
    DATA INFILE("oss://bucket_oss/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "fs.oss.accessKeyId" = "<oss_access_key>",
    "fs.oss.accessKeySecret" = "<oss_secret_key>",
    "fs.oss.endpoint" = "<oss_endpoint>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交导入作业后，您可以使用`SELECT * FROM information_schema.loads`来查看Broker Load作业的结果。该功能自3.1版本起支持。具体请参见本文“[查看导入作业](#查看导入作业)”小节。

确认导入作业成功后，您可以使用[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)语句来查询`table1`的数据，如下所示：

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Lily  |    21 |
|    2 | Rose  |    22 |
|    3 | Alice |    23 |
|    4 | Julia |    24 |
+------+-------+-------+
4 rows in set (0.01 sec)
```

### 导入多个数据文件到单表

#### 操作示例

通过以下语句，将阿里云OSS存储空间`bucket_oss`内`input`文件夹中的所有数据文件（`file1.csv`和`file2.csv`）的数据导入到目标表`table1`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_402
(
    DATA INFILE("oss://bucket_oss/input/*")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "fs.oss.accessKeyId" = "<oss_access_key>",
    "fs.oss.accessKeySecret" = "<oss_secret_key>",
    "fs.oss.endpoint" = "<oss_endpoint>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交导入作业后，您可以使用`SELECT * FROM information_schema.loads`来查看Broker Load作业的结果。该功能自3.1版本起支持。具体请参见本文“[查看导入作业](#查看导入作业)”小节。

确认导入作业成功后，您可以使用[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)语句来查询`table1`的数据，如下所示：

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Lily  |    21 |
|    2 | Rose  |    22 |
|    3 | Alice |    23 |
|    4 | Julia |    24 |
|    5 | Tony  |    25 |
|    6 | Adam  |    26 |
|    7 | Allen |    27 |
|    8 | Jacky |    28 |
+------+-------+-------+
4 rows in set (0.01 sec)
```

### 导入多个数据文件到多表

#### 操作示例

通过以下命令，将阿里云OSS存储空间`bucket_oss`内`input`文件夹中的数据文件`file1.csv`和`file2.csv`分别导入到目标表`table1`和`table2`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_403
(
    DATA INFILE("oss://bucket_oss/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("oss://bucket_oss/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "fs.oss.accessKeyId" = "<oss_access_key>",
    "fs.oss.accessKeySecret" = "<oss_secret_key>",
    "fs.oss.endpoint" = "<oss_endpoint>"
);
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交导入作业以后，您可以使用 `SELECT * FROM information_schema.loads` 来查看 Broker Load 作业的结果，该功能自 3.1 版本起支持，具体请参见本文“[查看导入作业](#查看导入作业)”小节。

确认导入作业成功以后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句来查询 `table1` 和 `table2` 中的数据：

1. 查询 `table1` 的数据，如下所示：

   ```SQL
   SELECT * FROM table1;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    1 | Lily  |    21 |
   |    2 | Rose  |    22 |
   |    3 | Alice |    23 |
   |    4 | Julia |    24 |
   +------+-------+-------+
   4 rows in set (0.01 sec)
   ```

2. 查询 `table2` 的数据，如下所示：

   ```SQL
   SELECT * FROM table2;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    5 | Tony  |    25 |
   |    6 | Adam  |    26 |
   |    7 | Allen |    27 |
   |    8 | Jacky |    28 |
   +------+-------+-------+
   4 rows in set (0.01 sec)
   ```

## 从腾讯云 COS 导入

下述命令以 CSV 格式为例。有关如何导入其他格式的数据，参见 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

### 导入单个数据文件到单表

#### 操作示例

通过如下语句，把腾讯云 COS 存储空间 `bucket_cos` 里 `input` 文件夹内数据文件 `file1.csv` 的数据导入到目标表 `table1`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_501
(
    DATA INFILE("cosn://bucket_cos/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "fs.cosn.userinfo.secretId" = "<cos_access_key>",
    "fs.cosn.userinfo.secretKey" = "<cos_secret_key>",
    "fs.cosn.bucket.endpoint_suffix" = "<cos_endpoint>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交导入作业以后，您可以使用 `SELECT * FROM information_schema.loads` 来查看 Broker Load 作业的结果，该功能自 3.1 版本起支持，具体请参见本文“[查看导入作业](#查看导入作业)”小节。

确认导入作业成功以后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句来查询 `table1` 的数据，如下所示：

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Lily  |    21 |
|    2 | Rose  |    22 |
|    3 | Alice |    23 |
|    4 | Julia |    24 |
+------+-------+-------+
4 rows in set (0.01 sec)
```

### 导入多个数据文件到单表

#### 操作示例

通过如下语句，把腾讯云 COS 存储空间 `bucket_cos` 里 `input` 文件夹内所有数据文件（`file1.csv` 和 `file2.csv`）的数据导入到目标表 `table1`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_502
(
    DATA INFILE("cosn://bucket_cos/input/*")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "fs.cosn.userinfo.secretId" = "<cos_access_key>",
    "fs.cosn.userinfo.secretKey" = "<cos_secret_key>",
    "fs.cosn.bucket.endpoint_suffix" = "<cos_endpoint>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交导入作业以后，您可以使用 `SELECT * FROM information_schema.loads` 来查看 Broker Load 作业的结果，该功能自 3.1 版本起支持，具体请参见本文“[查看导入作业](#查看导入作业)”小节。

确认导入作业成功以后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句来查询 `table1` 的数据，如下所示：

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Lily  |    21 |
|    2 | Rose  |    22 |
|    3 | Alice |    23 |
|    4 | Julia |    24 |
|    5 | Tony  |    25 |
|    6 | Adam  |    26 |
|    7 | Allen |    27 |
|    8 | Jacky |    28 |
+------+-------+-------+
4 rows in set (0.01 sec)
```

### 导入多个数据文件到多表

#### 操作示例

通过如下语句，把腾讯云 COS 存储空间 `bucket_cos` 里 `input` 文件夹内数据文件 `file1.csv` 和 `file2.csv` 的数据分别导入到目标表 `table1` 和 `table2`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_503
(
    DATA INFILE("cosn://bucket_cos/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("cosn://bucket_cos/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "fs.cosn.userinfo.secretId" = "<cos_access_key>",
    "fs.cosn.userinfo.secretKey" = "<cos_secret_key>",
    "fs.cosn.bucket.endpoint_suffix" = "<cos_endpoint>"
);
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交导入作业以后，您可以使用 `SELECT * FROM information_schema.loads` 来查看 Broker Load 作业的结果，该功能自 3.1 版本起支持，具体请参见本文“[查看导入作业](#查看导入作业)”小节。

确认导入作业成功以后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句来查询 `table1` 和 `table2` 中的数据：

1. 查询 `table1` 的数据，如下所示：

   ```SQL
   SELECT * FROM table1;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    1 | Lily  |    21 |
   |    2 | Rose  |    22 |
   |    3 | Alice |    23 |
   |    4 | Julia |    24 |
   +------+-------+-------+
   4 rows in set (0.01 sec)
   ```

2. 查询 `table2` 的数据，如下所示：

   ```SQL
   SELECT * FROM table2;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    5 | Tony  |    25 |
   |    6 | Adam  |    26 |
   |    7 | Allen |    27 |
   |    8 | Jacky |    28 |
   +------+-------+-------+
   4 rows in set (0.01 sec)
   ```

## 从华为云 OBS 导入

下述命令以 CSV 格式为例。有关如何导入其他格式的数据，参见 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

### 导入单个数据文件到单表

#### 操作示例

```SQL
LOAD LABEL test_db.label_brokerloadtest_601
(
    DATA INFILE("obs://bucket_obs/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "fs.obs.access.key" = "<obs_access_key>",
    "fs.obs.secret.key" = "<obs_secret_key>",
    "fs.obs.endpoint" = "<obs_endpoint>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交导入作业以后，您可以使用 `SELECT * FROM information_schema.loads` 来查看 Broker Load 作业的结果，该功能自 3.1 版本起支持，具体请参见本文“[查看导入作业](#查看导入作业)”小节。

确认导入作业成功以后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句来查询 `table1` 的数据，如下所示：

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Lily  |    21 |
|    2 | Rose  |    22 |
|    3 | Alice |    23 |
|    4 | Julia |    24 |
+------+-------+-------+
4 rows in set (0.01 sec)
```

### 导入多个数据文件到单表

#### 操作示例

通过如下语句，把华为云 OBS 存储空间 `bucket_obs` 里 `input` 文件夹内所有数据文件（`file1.csv` 和 `file2.csv`）的数据导入到目标表 `table1`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_602
(
    DATA INFILE("obs://bucket_obs/input/*")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "fs.obs.access.key" = "<obs_access_key>",
    "fs.obs.secret.key" = "<obs_secret_key>",
    "fs.obs.endpoint" = "<obs_endpoint>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交导入作业以后，您可以使用 `SELECT * FROM information_schema.loads` 来查看 Broker Load 作业的结果，该功能自 3.1 版本起支持，具体请参见本文“[查看导入作业](#查看导入作业)”小节。

确认导入作业成功以后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句来查询 `table1` 的数据，如下所示：

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Lily  |    21 |
|    2 | Rose  |    22 |
|    3 | Alice |    23 |
|    4 | Julia |    24 |
|    5 | Tony  |    25 |
|    6 | Adam  |    26 |
|    7 | Allen |    27 |
|    8 | Jacky |    28 |
+------+-------+-------+
4 rows in set (0.01 sec)
```

### 导入多个数据文件到多表

#### 操作示例

通过如下语句，把华为云 OBS 存储空间 `bucket_obs` 里 `input` 文件夹内数据文件 `file1.csv` 和 `file2.csv` 的数据分别导入到目标表 `table1` 和 `table2`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_603
(
    DATA INFILE("obs://bucket_obs/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("obs://bucket_obs/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "fs.obs.access.key" = "<obs_access_key>",
    "fs.obs.secret.key" = "<obs_secret_key>",
    "fs.obs.endpoint" = "<obs_endpoint>"
);
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交导入作业以后，您可以使用 `SELECT * FROM information_schema.loads` 来查看 Broker Load 作业的结果，该功能自 3.1 版本起支持，具体请参见本文“[查看导入作业](#查看导入作业)”小节。

确认导入作业成功以后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句来查询 `table1` 和 `table2` 中的数据：

1. 查询 `table1` 的数据，如下所示：

   ```SQL
   SELECT * FROM table1;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    1 | Lily  |    21 |
   |    2 | Rose  |    22 |
   |    3 | Alice |    23 |
   |    4 | Julia |    24 |
   +------+-------+-------+
   4 rows in set (0.01 sec)
   ```

2. 查询 `table2` 的数据，如下所示：

   ```SQL
   SELECT * FROM table2;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    5 | Tony  |    25 |
   |    6 | Adam  |    26 |
   |    7 | Allen |    27 |
   |    8 | Jacky |    28 |
   +------+-------+-------+
   4 rows in set (0.01 sec)
   ```
``` SQL
LOAD LABEL test_db.label_brokerloadtest_602
(
    DATA INFILE("obs://bucket_obs/input/*")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "fs.obs.access.key" = "<obs_access_key>",
    "fs.obs.secret.key" = "<obs_secret_key>",
    "fs.obs.endpoint" = "<obs_endpoint>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交导入作业以后，您可以使用 `SELECT * FROM information_schema.loads` 来查看 Broker Load 作业的结果，该功能自 3.1 版本起支持，具体请参见本文“[查看导入作业](#查看导入作业)”小节。

确认导入作业成功以后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句来查询 `table1` 的数据，如下所示：

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Lily  |    21 |
|    2 | Rose  |    22 |
|    3 | Alice |    23 |
|    4 | Julia |    24 |
|    5 | Tony  |    25 |
|    6 | Adam  |    26 |
|    7 | Allen |    27 |
|    8 | Jacky |    28 |
+------+-------+-------+
4 rows in set (0.01 sec)
```

### 导入多个数据文件到多表

#### 操作示例

通过如下语句，把华为云 OBS 存储空间 `bucket_obs` 里 `input` 文件夹内数据文件 `file1.csv` 和 `file2.csv` 的数据分别导入到目标表 `table1` 和 `table2`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_603
(
    DATA INFILE("obs://bucket_obs/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("obs://bucket_obs/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "fs.obs.access.key" = "<obs_access_key>",
    "fs.obs.secret.key" = "<obs_secret_key>",
    "fs.obs.endpoint" = "<obs_endpoint>"
);
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交导入作业以后，您可以使用 `SELECT * FROM information_schema.loads` 来查看 Broker Load 作业的结果，该功能自 3.1 版本起支持，具体请参见本文“[查看导入作业](#查看导入作业)”小节。

确认导入作业成功以后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句来查询 `table1` 和 `table2` 中的数据：

1. 查询 `table1` 的数据，如下所示：

   ```SQL
   SELECT * FROM table1;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    1 | Lily  |    21 |
   |    2 | Rose  |    22 |
   |    3 | Alice |    23 |
   |    4 | Julia |    24 |
   +------+-------+-------+
   4 rows in set (0.01 sec)
   ```

2. 查询 `table2` 的数据，如下所示：

   ```SQL
   SELECT * FROM table2;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    5 | Tony  |    25 |
   |    6 | Adam  |    26 |
   |    7 | Allen |    27 |
   |    8 | Jacky |    28 |
   +------+-------+-------+
   4 rows in set (0.01 sec)
   ```
``` SQL
LOAD LABEL test_db.label_brokerloadtest_603
(
    DATA INFILE("obs://bucket_obs/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("obs://bucket_obs/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "fs.obs.access.key" = "<obs_access_key>",
    "fs.obs.secret.key" = "<obs_secret_key>",
    "fs.obs.endpoint" = "<obs_endpoint>"
);
PROPERTIES
(
    "timeout" = "3600"
);
```
通过如下语句，把 MinIO 存储空间 `bucket_minio` 里 `input` 文件夹内所有数据文件（`file1.csv` 和 `file2.csv`）的数据导入到目标表 `table1`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_702
(
    DATA INFILE("obs://bucket_minio/input/*")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "aws.s3.enable_ssl" = "false",
    "aws.s3.enable_path_style_access" = "true",
    "aws.s3.endpoint" = "<s3_endpoint>",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交导入作业以后，您可以使用 `SELECT * FROM information_schema.loads` 来查看 Broker Load 作业的结果，该功能自 3.1 版本起支持，具体请参见本文“[查看导入作业](#查看导入作业)”小节。

确认导入作业成功以后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句来查询 `table1` 的数据，如下所示：

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Lily  |    21 |
|    2 | Rose  |    22 |
|    3 | Alice |    23 |
|    4 | Julia |    24 |
|    5 | Tony  |    25 |
|    6 | Adam  |    26 |
|    7 | Allen |    27 |
|    8 | Jacky |    28 |
+------+-------+-------+
4 rows in set (0.01 sec)
```

### 导入多个数据文件到多表

#### 操作示例

通过如下语句，把 MinIO 存储空间 `bucket_minio` 里 `input` 文件夹内数据文件 `file1.csv` 和 `file2.csv` 的数据分别导入到目标表 `table1` 和 `table2`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_703
(
    DATA INFILE("obs://bucket_minio/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("obs://bucket_minio/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "aws.s3.enable_ssl" = "false",
    "aws.s3.enable_path_style_access" = "true",
    "aws.s3.endpoint" = "<s3_endpoint>",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>"
);
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交导入作业以后，您可以使用 `SELECT * FROM information_schema.loads` 来查看 Broker Load 作业的结果，该功能自 3.1 版本起支持，具体请参见本文“[查看导入作业](#查看导入作业)”小节。

确认导入作业成功以后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句来查询 `table1` 和 `table2` 中的数据：

1. 查询 `table1` 的数据，如下所示：

   ```SQL
   SELECT * FROM table1;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    1 | Lily  |    21 |
   |    2 | Rose  |    22 |
   |    3 | Alice |    23 |
   |    4 | Julia |    24 |
   +------+-------+-------+
   4 rows in set (0.01 sec)
   ```

2. 查询 `table2` 的数据，如下所示：

   ```SQL
   SELECT * FROM table2;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    5 | Tony  |    25 |
   |    6 | Adam  |    26 |
   |    7 | Allen |    27 |
   |    8 | Jacky |    28 |
   +------+-------+-------+
   4 rows in set (0.01 sec)
   ```

## 查看导入作业

通过 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句从 `information_schema` 数据库中的 `loads` 表来查看 Broker Load 作业的结果。该功能自 3.1 版本起支持。

示例一：通过如下命令查看 `test_db` 数据库中导入作业的执行情况，同时指定查询结果根据作业创建时间 (`CREATE_TIME`) 按降序排列，并且最多显示两条结果数据：

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'test_db'
ORDER BY create_time DESC
LIMIT 2\G
```

返回结果如下所示：

```SQL
*************************** 1. row ***************************
              JOB_ID: 20686
               LABEL: label_brokerload_unqualifiedtest_83
       DATABASE_NAME: test_db
               STATE: FINISHED
            PROGRESS: ETL:100%; LOAD:100%
                TYPE: BROKER
            PRIORITY: NORMAL
           SCAN_ROWS: 8
       FILTERED_ROWS: 0
     UNSELECTED_ROWS: 0
           SINK_ROWS: 8
            ETL_INFO:
           TASK_INFO: resource:N/A; timeout(s):14400; max_filter_ratio:1.0
         CREATE_TIME: 2023-08-02 15:25:22
      ETL_START_TIME: 2023-08-02 15:25:24
     ETL_FINISH_TIME: 2023-08-02 15:25:24
     LOAD_START_TIME: 2023-08-02 15:25:24
    LOAD_FINISH_TIME: 2023-08-02 15:25:27
         JOB_DETAILS: {"All backends":{"77fe760e-ec53-47f7-917d-be5528288c08":[10006],"0154f64e-e090-47b7-a4b2-92c2ece95f97":[10005]},"FileNumber":2,"FileSize":84,"InternalTableLoadBytes":252,"InternalTableLoadRows":8,"ScanBytes":84,"ScanRows":8,"TaskNumber":2,"Unfinished backends":{"77fe760e-ec53-47f7-917d-be5528288c08":[],"0154f64e-e090-47b7-a4b2-92c2ece95f97":[]}}
           ERROR_MSG: NULL
        TRACKING_URL: NULL
        TRACKING_SQL: NULL
REJECTED_RECORD_PATH: NULL
*************************** 2. row ***************************
              JOB_ID: 20624
               LABEL: label_brokerload_unqualifiedtest_82
       DATABASE_NAME: test_db
               STATE: FINISHED
            PROGRESS: ETL:100%; LOAD:100%
                TYPE: BROKER
            PRIORITY: NORMAL
           SCAN_ROWS: 12
       FILTERED_ROWS: 4
     UNSELECTED_ROWS: 0
           SINK_ROWS: 8
            ETL_INFO:
           TASK_INFO: resource:N/A; timeout(s):14400; max_filter_ratio:1.0
         CREATE_TIME: 2023-08-02 15:23:29
      ETL_START_TIME: 2023-08-02 15:23:34
     ETL_FINISH_TIME: 2023-08-02 15:23:34
     LOAD_START_TIME: 2023-08-02 15:23:34
    LOAD_FINISH_TIME: 2023-08-02 15:23:34
```
      JOB_DETAILS: {"All backends":{"78f78fc3-8509-451f-a0a2-c6b5db27dcb6":[10010],"a24aa357-f7de-4e49-9e09-e98463b5b53c":[10006]},"FileNumber":2,"FileSize":158,"InternalTableLoadBytes":333,"InternalTableLoadRows":8,"ScanBytes":158,"ScanRows":12,"TaskNumber":2,"Unfinished backends":{"78f78fc3-8509-451f-a0a2-c6b5db27dcb6":[],"a24aa357-f7de-4e49-9e09-e98463b5b53c":[]}}
           ERROR_MSG: NULL
        TRACKING_URL: http://172.26.195.69:8540/api/_load_error_log?file=error_log_78f78fc38509451f_a0a2c6b5db27dcb7
        TRACKING_SQL: select tracking_log from information_schema.load_tracking_logs where job_id=20624
REJECTED_RECORD_PATH: 172.26.95.92:/home/disk1/sr/be/storage/rejected_record/test_db/label_brokerload_unqualifiedtest_0728/6/404a20b1e4db4d27_8aa9af1e8d6d8bdc
```

例二：通过以下命令查看 `test_db` 数据库中标签为 `label_brokerload_unqualifiedtest_82` 的导入作业的执行情况：

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'test_db' and label = 'label_brokerload_unqualifiedtest_82'\G
```

返回结果如下所示：

```SQL
*************************** 1. row ***************************
              JOB_ID: 20624
               LABEL: label_brokerload_unqualifiedtest_82
       DATABASE_NAME: test_db
               STATE: FINISHED
            PROGRESS: ETL:100%; LOAD:100%
                TYPE: BROKER
            PRIORITY: NORMAL
           SCAN_ROWS: 12
       FILTERED_ROWS: 4
     UNSELECTED_ROWS: 0
           SINK_ROWS: 8
            ETL_INFO:
           TASK_INFO: resource:N/A; timeout(s):14400; max_filter_ratio:1.0
         CREATE_TIME: 2023-08-02 15:23:29
      ETL_START_TIME: 2023-08-02 15:23:34
     ETL_FINISH_TIME: 2023-08-02 15:23:34
     LOAD_START_TIME: 2023-08-02 15:23:34
    LOAD_FINISH_TIME: 2023-08-02 15:23:34
         JOB_DETAILS: {"All backends":{"78f78fc3-8509-451f-a0a2-c6b5db27dcb6":[10010],"a24aa357-f7de-4e49-9e09-e98463b5b53c":[10006]},"FileNumber":2,"FileSize":158,"InternalTableLoadBytes":333,"InternalTableLoadRows":8,"ScanBytes":158,"ScanRows":12,"TaskNumber":2,"Unfinished backends":{"78f78fc3-8509-451f-a0a2-c6b5db27dcb6":[],"a24aa357-f7de-4e49-9e09-e98463b5b53c":[]}}
           ERROR_MSG: NULL
        TRACKING_URL: http://172.26.195.69:8540/api/_load_error_log?file=error_log_78f78fc38509451f_a0a2c6b5db27dcb7
        TRACKING_SQL: select tracking_log from information_schema.load_tracking_logs where job_id=20624
REJECTED_RECORD_PATH: 172.26.95.92:/home/disk1/sr/be/storage/rejected_record/test_db/label_brokerload_unqualifiedtest_0728/6/404a20b1e4db4d27_8aa9af1e8d6d8bdc
```

有关返回字段的说明，参见 [`information_schema.loads`](../reference/information_schema/loads.md)。

## 取消导入作业

当导入作业状态不为 **CANCELLED** 或 **FINISHED** 时，可以通过 [CANCEL LOAD](../sql-reference/sql-statements/data-manipulation/CANCEL_LOAD.md) 语句来取消该导入作业。

例如，可以通过以下语句，撤销 `test_db` 数据库中标签为 `label1` 的导入作业：

```SQL
CANCEL LOAD
FROM test_db
WHERE LABEL = "label1";
```

## 作业拆分与并行执行

一个 Broker Load 作业会拆分成一个或者多个子任务并行处理，一个作业的所有子任务作为一个事务整体成功或失败。作业的拆分通过 `LOAD LABEL` 语句中的 `data_desc` 参数来指定：

- 如果声明多个 `data_desc` 参数对应导入多张不同的表，则每张表数据的导入会拆分成一个子任务。

- 如果声明多个 `data_desc` 参数对应导入同一张表的不同分区，则每个分区数据的导入会拆分成一个子任务。

每个子任务还会拆分成一个或者多个实例，然后这些实例会均匀地被分配到 BE 上并行执行。实例的拆分由以下 [FE 配置](../administration/Configuration.md#配置-fe-动态参数)决定：

- `min_bytes_per_broker_scanner`：单个实例处理的最小数据量，默认为 64 MB。

- `load_parallel_instance_num`：单个 BE 上每个作业允许的并发实例数，默认为 1 个。自 3.1 版本起弃用。

   可以使用如下公式计算单个子任务的实例总数：

   单个子任务的实例总数 = min（单个子任务待导入数据量的总大小/`min_bytes_per_broker_scanner`，`load_parallel_instance_num` x BE 总数）

一般情况下，一个导入作业只有一个 `data_desc`，只会拆分成一个子任务，子任务会拆分成与 BE 总数相等的实例。

## 常见问题

请参见 [Broker Load 常见问题](../faq/loading/Broker_load_faq.md)。