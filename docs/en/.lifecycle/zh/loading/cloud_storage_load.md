---
displayed_sidebar: English
---

# 从云存储加载数据

导入 InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocks 支持使用以下两种方法从云存储加载大量数据：[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 和 [INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

在 v3.0 及更早版本中，StarRocks 仅支持 Broker Load，它以异步加载模式运行。提交加载作业后，StarRocks会异步运行该作业。您可以使用 `SELECT * FROM information_schema.loads` 查询作业结果。此功能从 v3.1 开始支持。有关更多信息，请参见本主题的“[查看加载作业](#view-a-load-job)”部分。

Broker Load 确保每个加载作业的事务原子性，用于加载多个数据文件，这意味着在一个加载作业中，加载多个数据文件必须全部成功或全部失败。不会出现某些数据文件加载成功而其他文件加载失败的情况。

此外，Broker Load 支持在数据加载时进行数据转换，并支持在数据加载期间通过 UPSERT 和 DELETE 操作进行的数据更改。有关更多信息，请参见[数据加载时的数据转换](../loading/Etl_in_loading.md)和[通过加载更改数据](../loading/Load_to_Primary_Key_tables.md)。

<InsertPrivNote />

从 v3.1 开始，StarRocks支持使用 INSERT 命令和 FILES 关键字直接从 AWS S3 加载 Parquet 格式或 ORC 格式的文件数据，无需先创建外部表。有关更多信息，请参见[INSERT > 使用 FILES 关键字直接从外部源中的文件插入数据](../loading/InsertInto.md#insert-data-directly-from-files-in-an-external-source-using-files)。

本主题重点介绍如何使用[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)从云存储加载数据。

## 支持的数据文件格式

Broker Load 支持以下数据文件格式：

- CSV

- Parquet

- ORC

> **注意**
>
> 对于 CSV 数据，请注意以下几点：
>
> - 您可以使用长度不超过 50 个字节的 UTF-8 字符串，例如逗号 (,)、制表符或竖线 (|)，作为文本分隔符。
> - 空值使用 `\N` 表示。例如，数据文件由三列组成，该数据文件中的记录在第一列和第三列中保存数据，但在第二列中不保存任何数据。在这种情况下，您需要在第二列使用 `\N` 表示空值。这意味着记录必须编译为 `a,\N,b` 而不是 `a,,b`。`a,,b` 表示记录的第二列包含空字符串。

## 工作原理

向 FE 提交加载作业后，FE 会生成查询计划，根据可用 BE 的数量和要加载的数据文件的大小，将查询计划拆分为多个部分，然后将查询计划的每个部分分配给可用的 BE。在加载过程中，每个参与的 BE 都会从外部存储系统拉取数据文件的数据，对数据进行预处理，然后将数据加载到 StarRocks 集群中。当所有 BE 完成查询计划的部分后，FE 会判断加载作业是否成功。

下图显示了 Broker Load 作业的工作流程。

![Broker Load 的工作流](../assets/broker_load_how-to-work_en.png)

## 准备数据示例

1. 登录到本地文件系统并创建两个 CSV 格式的数据文件，`file1.csv` 和 `file2.csv`。这两个文件都由三列组成，分别表示用户 ID、用户名和用户分数。

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

2. 将 `file1.csv` 和 `file2.csv` 上传到 AWS S3 存储桶的 `input` 文件夹、Google GCS 存储桶的 `input` 文件夹、S3 兼容存储对象（例如 MinIO）存储桶的 `input` 文件夹以及 Azure 存储的指定路径。

3. 登录到 StarRocks 数据库（例如 `test_db`），创建两个主键表，`table1` 和 `table2`。这两个表都由三列组成：`id`、`name` 和 `score`，其中 `id` 是主键。

   ```SQL
   CREATE TABLE `table1`
      (
          `id` int(11) NOT NULL COMMENT "user ID",
          `name` varchar(65533) NULL DEFAULT "" COMMENT "user name",
          `score` int(11) NOT NULL DEFAULT "0" COMMENT "user score"
      )
          ENGINE=OLAP
          PRIMARY KEY(`id`)
          DISTRIBUTED BY HASH(`id`);
             
   CREATE TABLE `table2`
      (
          `id` int(11) NOT NULL COMMENT "user ID",
          `name` varchar(65533) NULL DEFAULT "" COMMENT "user name",
          `score` int(11) NOT NULL DEFAULT "0" COMMENT "user score"
      )
          ENGINE=OLAP
          PRIMARY KEY(`id`)
          DISTRIBUTED BY HASH(`id`);
   ```

## 从 AWS S3 加载数据

请注意，Broker Load 支持根据 S3 或 S3A 协议访问 AWS S3。因此，当您从 AWS S3 加载数据时，您可以在 `DATA INFILE` 中作为文件路径传递的 S3 URI 中包含 `s3://` 或 `s3a://` 作为前缀。

另请注意，以下示例使用 CSV 文件格式和基于实例配置文件的身份验证方法。有关如何加载其他格式的数据以及使用其他身份验证方法时需要配置的身份验证参数的信息，请参见[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

### 将单个数据文件加载到单个表中

#### 示例

执行以下语句，将存储在 AWS S3 存储桶的 `input` 文件夹中的 `file1.csv` 的数据加载到 `table1`：

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

#### 查询数据

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 查询作业结果。此功能从 v3.1 开始支持。有关更多信息，请参见本主题的“[查看加载作业](#view-a-load-job)”部分。

确认加载作业成功后，您可以使用[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)查询 `table1` 中的数据：

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

### 将多个数据文件加载到单个表中

#### 示例

执行以下语句，将存储在 AWS S3 存储桶的 `input` 文件夹中的所有数据文件（`file1.csv` 和 `file2.csv`）的数据加载到 `table1`：

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

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 查询作业结果。此功能从 v3.1 开始支持。有关更多信息，请参见本主题的“[查看加载作业](#view-a-load-job)”部分。

确认加载作业成功后，您可以使用[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)查询 `table1` 中的数据：

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

### 将多个数据文件加载到多个表中

#### 示例

执行以下语句，将存储在 AWS S3 存储桶的 `input` 文件夹中的 `file1.csv` 和 `file2.csv` 的数据分别加载到 `table1` 和 `table2`：

```SQL

加载标签 test_db.label_brokerloadtest_103
(
    数据 INFILE("s3a://bucket_s3/input/file1.csv")
    到表 table1
    列 以","终止
    (id, name, score)
    ,
    数据 INFILE("s3a://bucket_s3/input/file2.csv")
    到表 table2
    列 以","终止
    (id, name, score)
)
使用经纪人
(
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.region" = "<aws_s3_region>"
)
属性
(
    "timeout" = "3600"
);
```

#### 查询数据

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 查询作业结果。从 v3.1 开始支持此功能。有关详细信息，请参阅本主题的“[查看加载作业](#view-a-load-job)”部分。

确认加载作业成功后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询 `table1` 和 `table2` 的数据：

1. 查询 `table1`：

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
   4 行 (0.01 秒)
   ```

2. 查询 `table2`：

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
   4 行 (0.01 秒)
   ```

## 从 Google GCS 加载数据

请注意，Broker Load 仅支持根据 gs 协议访问 Google GCS。因此，当您从 Google GCS 加载数据时，您必须将 `gs://` 作为前缀包含在 GCS URI 中，作为文件路径 (`DATA INFILE`) 传递。

另请注意，以下示例使用 CSV 文件格式和基于 VM 的身份验证方法。有关如何装入其他格式的数据以及使用其他认证方法时需要配置的认证参数的信息，请参阅 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

### 将单个数据文件加载到单个表中

#### 示例

执行以下语句，将存储在 Google GCS 存储桶 `bucket_gcs` 的 `input` 文件夹中的 `file1.csv` 数据加载到 `table1`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_201
(
    数据 INFILE("gs://bucket_gcs/input/file1.csv")
    到表 table1
    列 以","终止
    (id, name, score)
)
使用经纪人
(
    "gcp.gcs.use_compute_engine_service_account" = "true"
)
属性
(
    "timeout" = "3600"
);
```

#### 查询数据

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 查询作业结果。从 v3.1 开始支持此功能。有关详细信息，请参阅本主题的“[查看加载作业](#view-a-load-job)”部分。

确认加载作业成功后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询 `table1` 的数据：

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
4 行 (0.01 秒)
```

### 将多个数据文件加载到单个表中

#### 示例

执行以下语句，将存储在 Google GCS 存储桶 `bucket_gcs` 的 `input` 文件夹中的所有数据文件（`file1.csv` 和 `file2.csv`）的数据加载到 `table1`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_202
(
    数据 INFILE("gs://bucket_gcs/input/*")
    到表 table1
    列 以","终止
    (id, name, score)
)
使用经纪人
(
    "gcp.gcs.use_compute_engine_service_account" = "true"
)
属性
(
    "timeout" = "3600"
);
```

#### 查询数据

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 查询作业结果。从 v3.1 开始支持此功能。有关详细信息，请参阅本主题的“[查看加载作业](#view-a-load-job)”部分。

确认加载作业成功后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询 `table1` 的数据：

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
4 行 (0.01 秒)
```

### 将多个数据文件加载到多个表中

#### 示例

执行以下语句，将存储在 Google GCS 存储桶 `bucket_gcs` 的 `input` 文件夹中的 `file1.csv` 和 `file2.csv` 数据分别加载到 `table1` 和 `table2` 中：

```SQL
LOAD LABEL test_db.label_brokerloadtest_203
(
    数据 INFILE("gs://bucket_gcs/input/file1.csv")
    到表 table1
    列 以","终止
    (id, name, score)
    ,
    数据 INFILE("gs://bucket_gcs/input/file2.csv")
    到表 table2
    列 以","终止
    (id, name, score)
)
使用经纪人
(
    "gcp.gcs.use_compute_engine_service_account" = "true"
);
属性
(
    "timeout" = "3600"
);
```

#### 查询数据

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 查询作业结果。从 v3.1 开始支持此功能。有关详细信息，请参阅本主题的“[查看加载作业](#view-a-load-job)”部分。

确认加载作业成功后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询 `table1` 和 `table2` 的数据：

1. 查询 `table1`：

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
   4 行 (0.01 秒)
   ```

2. 查询 `table2`：

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
   4 行 (0.01 秒)
   ```

## 从 Microsoft Azure 存储加载数据

请注意，从 Azure 存储加载数据时，需要根据使用的访问协议和特定存储服务确定要使用的前缀：

- 从 Blob 存储加载数据时，必须根据用于访问存储帐户的协议，在文件路径 (`DATA INFILE`) 中包含 `wasb://` 或 `wasbs://` 作为前缀：
  - 如果 Blob 存储仅允许通过 HTTP 进行访问，请使用 `wasb://` 前缀，例如 `wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`.
  - 如果 Blob 存储仅允许通过 HTTPS 进行访问，请使用 `wasbs://` 前缀，例如 `wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`
- 从 Data Lake Storage Gen1 加载数据时，必须在文件路径 (`DATA INFILE`) 中包含 `adl://` 前缀，例如 `adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`.
- 从 Data Lake Storage Gen2 加载数据时，必须根据用于访问存储帐户的协议，在文件路径 (`DATA INFILE`) 中包含 `abfs://` 或 `abfss://` 作为前缀：
  - 如果 Data Lake Storage Gen2 仅允许通过 HTTP 进行访问，请使用 `abfs://` 前缀，例如 `abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>`.
  - 如果 Data Lake Storage Gen2 仅允许通过 HTTPS 进行访问，请使用 `abfss://` 前缀，例如 `abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>`.


另外，请注意，以下示例使用 CSV 文件格式、Azure Blob 存储和基于共享密钥的身份验证方法。有关如何加载其他格式的数据以及在使用其他 Azure 存储服务和身份验证方法时需要配置的身份验证参数的信息，请参阅 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

### 将单个数据文件加载到单个表中

#### 示例

执行以下语句，将存储在 Azure 存储指定路径中的 `file1.csv` 数据加载到 `table1`：

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

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 查询作业结果。此功能从 v3.1 开始支持。有关更多信息，请参阅本主题的 "[查看加载作业](#view-a-load-job)" 部分。

确认加载作业成功后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询 `table1` 的数据：

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

### 将多个数据文件加载到单个表中

#### 示例

执行以下语句，将存储在 Azure 存储指定路径中的所有数据文件（`file1.csv` 和 `file2.csv`）数据加载到 `table1`：

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

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 查询作业结果。此功能从 v3.1 开始支持。有关更多信息，请参阅本主题的 "[查看加载作业](#view-a-load-job)" 部分。

确认加载作业成功后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询 `table1` 的数据：

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

### 将多个数据文件加载到多个表中

#### 示例

执行以下语句，将存储在 Azure 存储指定路径中的 `file1.csv` 和 `file2.csv` 数据分别加载到 `table1` 和 `table2`：

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
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 查询作业结果。此功能从 v3.1 开始支持。有关更多信息，请参阅本主题的 "[查看加载作业](#view-a-load-job)" 部分。

确认加载作业成功后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询 `table1` 和 `table2` 的数据：

1. 查询 `table1`：

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

2. 查询 `table2`：

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

## 从兼容 S3 的存储系统加载数据

以下示例使用 CSV 文件格式和 MinIO 存储系统。有关如何加载其他格式的数据的信息，请参阅 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

### 将单个数据文件加载到单个表中

#### 示例

执行以下语句，将存储在 MinIO 存储桶 `bucket_minio` 的 `input` 文件夹中的 `file1.csv` 数据加载到 `table1`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_401
(
    DATA INFILE("obs://bucket_minio/input/file1.csv")
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

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 查询作业结果。此功能从 v3.1 开始支持。有关更多信息，请参阅本主题的 "[查看加载作业](#view-a-load-job)" 部分。

确认加载作业成功后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询 `table1` 的数据：

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

### 将多个数据文件加载到单个表中

#### 示例

执行以下语句，将存储在 MinIO 存储桶 `bucket_minio` 的 `input` 文件夹中的所有数据文件（`file1.csv` 和 `file2.csv`）数据加载到 `table1`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_402
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

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 查询作业结果。此功能从 v3.1 开始支持。有关更多信息，请参阅本主题的 "[查看加载作业](#view-a-load-job)" 部分。

确认加载作业成功后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询 `table1` 的数据：

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

### 将多个数据文件加载到多个表中

#### 示例

执行以下语句，将存储在 MinIO 存储桶 `bucket_minio` 的 `input` 文件夹中的 `file1.csv` 和 `file2.csv` 数据分别加载到 `table1` 和 `table2`：

```SQL
LOAD LABEL test_db.label_brokerloadtest_403
(
    DATA INFILE("obs://bucket_minio/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("obs://bucket_minio/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, city)
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

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 查询作业结果。此功能从 v3.1 版本开始支持。有关详细信息，请参阅本主题的“[查看加载作业](#view-a-load-job)”部分。

确认加载作业成功后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询 `table1` 和 `table2` 的数据：

1. 查询 `table1`：

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

2. 查询 `table2`：

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

## 查看加载作业

使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句从 `information_schema` 数据库中的 `loads` 表中查询一个或多个加载作业的结果。此功能从 v3.1 版本开始支持。

示例1：查询在 `test_db` 数据库上执行的加载作业的结果。在查询语句中，指定最多返回两个结果，并且返回结果必须按创建时间（`CREATE_TIME`）降序排序。

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'test_db'
ORDER BY create_time DESC
LIMIT 2\G
```

返回如下结果：

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
         JOB_DETAILS: {"All backends":{"78f78fc3-8509-451f-a0a2-c6b5db27dcb6":[10010],"a24aa357-f7de-4e49-9e09-e98463b5b53c":[10006]},"FileNumber":2,"FileSize":158,"InternalTableLoadBytes":333,"InternalTableLoadRows":8,"ScanBytes":158,"ScanRows":12,"TaskNumber":2,"Unfinished backends":{"78f78fc3-8509-451f-a0a2-c6b5db27dcb6":[],"a24aa357-f7de-4e49-9e09-e98463b5b53c":[]}}
           ERROR_MSG: NULL
        TRACKING_URL: http://172.26.195.69:8540/api/_load_error_log?file=error_log_78f78fc38509451f_a0a2c6b5db27dcb7
        TRACKING_SQL: select tracking_log from information_schema.load_tracking_logs where job_id=20624
REJECTED_RECORD_PATH: 172.26.95.92:/home/disk1/sr/be/storage/rejected_record/test_db/label_brokerload_unqualifiedtest_0728/6/404a20b1e4db4d27_8aa9af1e8d6d8bdc
```

示例 2：查询在 `test_db` 数据库上执行的加载作业（标签为 `label_brokerload_unqualifiedtest_82`）的结果：

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'test_db' and label = 'label_brokerload_unqualifiedtest_82'\G
```

返回如下结果：

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

有关返回结果中的字段的信息，请参阅 [Information Schema > loads](../reference/information_schema/loads.md)。

## 取消加载作业

当加载作业不处于 **CANCELLED** 或 **FINISHED** 阶段时，您可以使用 [CANCEL LOAD](../sql-reference/sql-statements/data-manipulation/CANCEL_LOAD.md) 语句取消该作业。

例如，您可以执行以下语句取消在 `test_db` 数据库中标签为 `label1` 的加载作业：

```SQL
CANCEL LOAD
FROM test_db
WHERE LABEL = "label1";
```

## 作业拆分和并发运行

Broker Load 作业可以拆分为一个或多个并发运行的任务。加载作业中的任务在单个事务中运行。它们必须全部成功或全部失败。StarRocks 根据您在 `LOAD` 语句中声明的 `data_desc` 对每个加载作业进行拆分：

- 如果声明了多个 `data_desc` 参数，每个参数指定一个不同的表，则会生成一个任务来加载每个表的数据。

- 如果声明了多个 `data_desc` 参数，每个参数为同一表指定了不同的分区，则会生成一个任务来加载每个分区的数据。

此外，每个任务还可以进一步分割为一个或多个实例，这些实例会均匀分布到 StarRocks 集群的 BE 上，并同时运行。StarRocks根据以下[FE配置对每个任务进行拆分](../administration/FE_configuration.md#fe-configuration-items)：

- `min_bytes_per_broker_scanner`：每个实例处理的最小数据量。默认值为 64 MB。

- `load_parallel_instance_num`：在单个 BE 上每个加载作业中允许的并发实例数。默认值为 1。
  
  您可以使用以下公式来计算单个任务中的实例数：

  **单个任务的实例数 = min（单个任务要加载的数据量/`min_bytes_per_broker_scanner`，`load_parallel_instance_num` x BE 数）**

在大多数情况下，每个加载作业只声明一个`data_desc`，每个加载作业只拆分为一个任务，并且该任务会被拆分为与 BE 数量相同的实例数。

## 故障排除

请参阅[Broker加载常见问题解答](../faq/loading/Broker_load_faq.md)。