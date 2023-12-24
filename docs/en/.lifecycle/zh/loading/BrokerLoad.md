---
displayed_sidebar: English
---

# 从 HDFS 或云存储加载数据

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocks 提供了基于 MySQL 的 Broker Load 加载方法，帮助您将大量数据从 HDFS 或云存储加载到 StarRocks 中。

Broker Load 以异步加载模式运行。提交加载作业后，StarRocks 会异步运行该作业。您需要使用 [SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) 语句或 `curl` 命令来检查作业的结果。

Broker Load 支持单表加载和多表加载。您可以通过运行一个 Broker Load 作业将一个或多个数据文件加载到一个或多个目标表中。Broker Load 确保每个加载作业的事务原子性，即在一个加载作业中加载多个数据文件时，要么全部成功，要么全部失败。绝不会出现某些数据文件加载成功而其他文件加载失败的情况。

Broker Load 支持在数据加载时进行数据转换，并支持在数据加载期间通过 UPSERT 和 DELETE 操作进行的数据更改。有关详细信息，请参阅[加载时的数据转换](../loading/Etl_in_loading.md)和[通过加载更改数据](../loading/Load_to_Primary_Key_tables.md)。

<InsertPrivNote />

## 背景信息

在 v2.4 及更早版本中，StarRocks 在运行 Broker Load 作业时，需要 Broker 来建立 StarRocks 集群与外部存储系统之间的连接。因此，您需要输入 `WITH BROKER "<broker_name>"` 以指定要在 load 语句中使用的代理。这称为“基于代理的加载”。代理是与文件系统接口集成的独立无状态服务。通过 Broker，StarRocks 可以访问和读取存储在外部存储系统中的数据文件，并可以使用自己的计算资源对这些数据文件的数据进行预处理和加载。

从 v2.5 开始，StarRocks 在运行 Broker Load 作业时不再依赖 Broker 来建立 StarRocks 集群与外部存储系统的连接。因此，您不再需要在 load 语句中指定 broker，但仍需要保留 `WITH BROKER` 关键字。这称为“无代理加载”。

当您的数据存储在 HDFS 中时，您可能会遇到无代理加载不起作用的情况。当您的数据存储在多个 HDFS 集群中或配置了多个 Kerberos 用户时，可能会发生这种情况。在这些情况下，您可以改用基于代理的加载。要成功执行此操作，请确保至少部署了一个独立的代理组。有关如何在这些情况下指定身份验证配置和 HA 配置的信息，请参阅 [HDFS](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#hdfs)。

> **注意**
>
> 您可以使用 [SHOW BROKER](../sql-reference/sql-statements/Administration/SHOW_BROKER.md) 语句来查看 StarRocks 集群中部署的 Broker。如果未部署代理，您可以按照[部署代理](../deployment/deploy_broker.md)中提供的说明来部署代理。

## 支持的数据文件格式

Broker Load 支持以下数据文件格式：

- CSV

- Parquet

- ORC

> **注意**
>
> 对于 CSV 数据，请注意以下几点：
>
> - 您可以使用长度不超过 50 个字节的 UTF-8 字符串，例如逗号（,）、制表符或竖线（|），作为文本分隔符。
> - 空值使用 `\N` 表示。例如，数据文件由三列组成，该数据文件中的记录在第一列和第三列中保存数据，但在第二列中不保存任何数据。在这种情况下，您需要在第二列使用 `\N` 表示空值，而不是使用 `a,,b`。`a,,b` 表示记录的第二列包含一个空字符串。

## 支持的存储系统

Broker Load 支持以下存储系统：

- HDFS

- AWS S3

- Google GCS

- 其他兼容 S3 的存储系统，例如 MinIO

- Microsoft Azure 存储

## 工作原理

向 FE 提交加载作业后，FE 会生成一个查询计划，根据可用 BE 的数量和要加载的数据文件的大小，将查询计划拆分为多个部分，然后将查询计划的每个部分分配给可用的 BE。在加载过程中，每个参与的 BE 都会从 HDFS 或云存储系统中拉取数据文件的数据，对数据进行预处理，然后将数据加载到 StarRocks 集群中。当所有 BE 完成查询计划的部分后，FE 会判断加载作业是否成功。

下图显示了 Broker Load 作业的工作流程。

![Broker Load 的工作流](../assets/broker_load_how-to-work_en.png)

## 基本操作

### 创建多表加载作业

本主题以 CSV 为例，介绍如何将多个数据文件加载到多个表中。有关如何加载其他文件格式的数据以及 Broker Load 的语法和参数描述的信息，请参阅 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

需要注意的是，在 StarRocks 中，一些文字被 SQL 语言用作保留关键字。不要在 SQL 语句中直接使用这些关键字。如果要在 SQL 语句中使用此类关键字，请将其括在一对反引号（`）中。请参阅 [关键字](../sql-reference/sql-statements/keywords.md)。

#### 数据示例

1. 在本地文件系统中创建 CSV 文件。

   a. 创建名为 `file1.csv` 的 CSV 文件。该文件由三列组成，分别依次表示用户 ID、用户名和用户分数。

      ```Plain
      1,Lily,23
      2,Rose,23
      3,Alice,24
      4,Julia,25
      ```

   b. 创建名为 `file2.csv` 的 CSV 文件。该文件由两列组成，分别依次表示城市 ID 和城市名称。

      ```Plain
      200,'Beijing'
      ```

2. 在 StarRocks 数据库中创建 StarRocks 表 `test_db`。

   > **注意**
   >
   > 从 v2.5.7 开始，StarRocks 可以在您创建表或添加分区时自动设置 BUCKET 数量。您不再需要手动设置存储桶数量。有关详细信息，请参阅 [确定存储桶数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   a. 创建名为 `table1` 的主键表。该表由三列组成：`id`、`name`和`score`，其中 `id` 是主键。

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
      ```

   b. 创建名为 `table2` 的主键表。该表由两列组成：`id` 和 `city`，其中 `id` 是主键。

      ```SQL
      CREATE TABLE `table2`
      (
          `id` int(11) NOT NULL COMMENT "city ID",
          `city` varchar(65533) NULL DEFAULT "" COMMENT "city name"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`)
      DISTRIBUTED BY HASH(`id`);
      ```

3. 将 `file1.csv` 和 `file2.csv` 上传到您的 HDFS 集群的 `/user/starrocks/` 路径，上传到 AWS S3 存储桶 `bucket_s3` 的 `input` 文件夹，上传到 Google GCS 存储桶 `bucket_gcs` 的 `input` 文件夹，上传到 MinIO 存储桶 `bucket_minio` 的 `input` 文件夹，并上传到 Azure 存储的指定路径。

#### 从 HDFS 加载数据

执行以下语句，将 `file1.csv` 和 `file2.csv` 从您的 HDFS 集群的 `/user/starrocks` 路径分别加载到 `table1` 和 `table2` 中：

```SQL
LOAD LABEL test_db.label1
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, city)
)
WITH BROKER
(
    StorageCredentialParams
)
PROPERTIES
(
    "timeout" = "3600"
);
```

在上述示例中，`StorageCredentialParams` 表示一组身份验证参数，这些参数因您选择的身份验证方法而异。有关更多信息，请参阅 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#hdfs)。

#### 从 AWS S3 加载数据

执行以下语句，将 `file1.csv` 和 `file2.csv` 从 AWS S3 存储桶 `bucket_s3` 的 `input` 文件夹分别加载到 `table1` 和 `table2` 中：

```SQL
LOAD LABEL test_db.label2
(
    DATA INFILE("s3a://bucket_s3/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("s3a://bucket_s3/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, city)
)
WITH BROKER
(
    StorageCredentialParams
);
```

> **注意**
>
> Broker Load 仅支持使用 S3A 协议访问 AWS S3。因此，当您从 AWS S3 加载数据时，必须将作为文件路径的 S3 URI 中的`s3://`替换为 `s3a://`。

在上述示例中，`StorageCredentialParams` 表示一组身份验证参数，这些参数因您选择的身份验证方法而异。有关更多信息，请参阅 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#aws-s3)。

从 v3.1 开始，StarRocks 支持使用 INSERT 命令和 TABLE 关键字直接从 AWS S3 加载 Parquet 格式或 ORC 格式文件的数据，省去了先创建外部表的麻烦。有关详细信息，请参阅 [使用 INSERT 加载数据 > 使用 TABLE 关键字直接从外部源中的文件插入数据](../loading/InsertInto.md#insert-data-directly-from-files-in-an-external-source-using-files)。

#### 从 Google GCS 加载数据

执行以下语句，将 `file1.csv` 和 `file2.csv` 从 Google GCS 存储桶 `bucket_gcs` 的 `input` 文件夹分别加载到 `table1` 和 `table2` 中：

```SQL
LOAD LABEL test_db.label3
(
    DATA INFILE("gs://bucket_gcs/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("gs://bucket_gcs/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, city)
)
WITH BROKER
(
    StorageCredentialParams
);
```

> **注意**
>
> Broker Load 仅支持使用 gs 协议访问 Google GCS。因此，当您从 Google GCS 加载数据时，您必须在作为文件路径的 GCS URI 中包含`gs://`作为前缀。

在上述示例中，`StorageCredentialParams` 表示一组身份验证参数，这些参数因您选择的身份验证方法而异。有关更多信息，请参阅 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#google-gcs)。

#### 从其他兼容 S3 的存储系统加载数据

以 MinIO 为例。您可以执行以下语句，将 `file1.csv` 和 `file2.csv` 从 MinIO 存储桶 `bucket_minio` 的 `input` 文件夹分别加载到 `table1` 和 `table2` 中：

```SQL
LOAD LABEL test_db.label7
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
    StorageCredentialParams
);
```

在上述示例中，`StorageCredentialParams` 表示一组身份验证参数，这些参数因您选择的身份验证方法而异。有关更多信息，请参阅 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#other-s3-compatible-storage-system)。

#### 从 Microsoft Azure 存储加载数据

执行以下语句，以从 Azure 存储的指定路径加载 `file1.csv` 和 `file2.csv`：

```SQL
LOAD LABEL test_db.label8
(
    DATA INFILE("wasb[s]://<container>@<storage_account>.blob.core.windows.net/<path>/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("wasb[s]://<container>@<storage_account>.blob.core.windows.net/<path>/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, city)
)
WITH BROKER
(
    StorageCredentialParams
);
```

> **注意**
  >
  > 从 Azure 存储加载数据时，需要根据使用的访问协议和特定存储服务确定要使用的前缀。上述示例以 Blob 存储为例。
  >
  > - 从 Blob 存储加载数据时，必须根据用于访问存储帐户的协议在文件路径中包含`wasb://`或`wasbs://`作为前缀：
  >   - 如果 Blob 存储仅允许通过 HTTP 进行访问，请使用`wasb://`作为前缀，例如 `wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`.
  >   - 如果 Blob 存储仅允许通过 HTTPS 进行访问，请使用`wasbs://`作为前缀，例如： `wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`
  > - 从 Data Lake Storage Gen1 加载数据时，必须在文件路径中包含`adl://`作为前缀，例如 `adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`.
  > - 从 Data Lake Storage Gen2 加载数据时，必须根据用于访问存储帐户的协议在文件路径中包含`abfs://`或`abfss://`作为前缀：
  >   - 如果 Data Lake Storage Gen2 仅允许通过 HTTP 进行访问，请使用`abfs://`作为前缀，例如 `abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>`.
  >   - 如果 Data Lake Storage Gen2 仅允许通过 HTTPS 进行访问，请使用`abfss://`作为前缀，例如 `abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。

在上述示例中，`StorageCredentialParams` 表示一组身份验证参数，这些参数因您选择的身份验证方法而异。有关更多信息，请参阅 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#microsoft-azure-storage)。

#### 查询数据

在从 HDFS 集群、AWS S3 存储桶或 Google GCS 存储桶加载数据完成后，您可以使用 SELECT 语句查询 StarRocks 表的数据，以验证加载是否成功。

1. 执行以下语句查询 `table1`：

   ```SQL
   MySQL [test_db]> SELECT * FROM table1;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    1 | Lily  |    23 |
   |    2 | Rose  |    23 |
   |    3 | Alice |    24 |
   |    4 | Julia |    25 |
   +------+-------+-------+
   4 rows in set (0.00 sec)
   ```

1. 执行以下语句查询 `table2` 的数据：

   ```SQL
   MySQL [test_db]> SELECT * FROM table2;
   +------+--------+
   | id   | city   |
   +------+--------+
   | 200  | Beijing|
   +------+--------+
   4 rows in set (0.01 sec)
   ```

### 创建单表加载作业

您还可以将指定路径中的单个数据文件或所有数据文件加载到单个目标表中。假设您的 AWS S3 存储桶 `bucket_s3` 包含一个名为 `input` 的文件夹。该 `input` 文件夹包含多个数据文件，其中一个名为 `file1.csv`。这些数据文件由与 `table1` 相同数量的列组成，并且每个数据文件中的列可以按顺序一对一映射到 `table1` 中的列。

要将 `file1.csv` 加载到 `table1` 中，请执行以下语句：

```SQL
LOAD LABEL test_db.label_7
(
    DATA INFILE("s3a://bucket_s3/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    FORMAT AS "CSV"
)
WITH BROKER 
(
    StorageCredentialParams
);
```

要将 `input` 文件夹中的所有数据文件加载到 `table1` 中，请执行以下语句：

```SQL
LOAD LABEL test_db.label_8
(
    DATA INFILE("s3a://bucket_s3/input/*")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    FORMAT AS "CSV"
)
WITH BROKER 
(
    StorageCredentialParams
);
```

在上述示例中，`StorageCredentialParams` 表示一组身份验证参数，这些参数根据您选择的身份验证方法而异。有关更多信息，请参阅 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#aws-s3)。

### 查看加载作业

Broker Load 允许您使用 SHOW LOAD 语句或 `curl` 命令查看加载作业。

#### 使用 SHOW LOAD

有关详细信息，请参阅 [SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)。

#### 使用 curl

语法如下：

```Bash
curl --location-trusted -u <username>:<password> \
    'http://<fe_host>:<fe_http_port>/api/<database_name>/_load_info?label=<label_name>'
```

> **注意**
>
> 如果您使用的帐户未设置密码，则只需输入 `<username>:`。

例如，您可以执行以下命令来查看 `test_db` 数据库中标签为 `label1` 的加载作业信息：

```Bash
curl --location-trusted -u <username>:<password> \
    'http://<fe_host>:<fe_http_port>/api/test_db/_load_info?label=label1'
```

`curl` 命令以 JSON 对象的形式返回有关加载作业的信息 `jobInfo`：

```JSON
{"jobInfo":{"dbName":"default_cluster:test_db","tblNames":["table1_simple"],"label":"label1","state":"FINISHED","failMsg":"","trackingUrl":""},"status":"OK","msg":"Success"}%
```

下表描述了 `jobInfo` 中的参数。

| **参数** | **描述**                                              |
| ------------- | ------------------------------------------------------------ |
| dbName（数据库名称）        | 数据加载到的数据库的名称           |
| tblNames（表名称）      | 数据加载到的表的名称             |
| label（标签）         | 加载作业的标签                                   |
| state（状态）         | 加载作业的状态。有效值：<ul><li>`PENDING`：加载作业在队列中等待调度</li><li>`QUEUEING`：加载作业在队列中等待调度</li><li>`LOADING`：加载作业正在运行</li><li>`PREPARED`：事务已提交</li><li>`FINISHED`：加载作业成功</li><li>`CANCELLED`：加载作业失败</li></ul>有关详细信息，请参阅[数据加载概述](../loading/Loading_intro.md)中的“异步加载”部分 |
| failMsg（失败消息）       | 加载作业失败的原因。如果加载作业的状态值为 `PENDING`、 `LOADING` 或 `FINISHED`，则 `failMsg` 参数返回 `NULL`。如果加载作业的状态值为 `CANCELLED`，则 `failMsg` 参数返回的值由两部分组成：`type` 和 `msg`。<ul><li>`type` 部分可以是以下任何值：</li><ul><li>`USER_CANCEL`：加载作业已手动取消</li><li>`ETL_SUBMIT_FAIL`：加载作业提交失败</li><li>`ETL-QUALITY-UNSATISFIED`：加载作业失败，因为不合格数据的百分比超过 `max-filter-ratio` 参数的值</li><li>`LOAD-RUN-FAIL`：加载作业在 `LOADING` 阶段失败</li><li>`TIMEOUT`：加载作业在指定的超时期限内无法完成</li><li>`UNKNOWN`：由于未知错误，加载作业失败</li></ul><li>`msg` 部分提供了加载失败的详细原因</li></ul> |
| trackingUrl（跟踪 URL）   | 用于访问加载作业中检测到的不合格数据的 URL。您可以使用 `curl` 或 `wget` 命令访问该 URL 并获取不合格数据。如果未检测到不合格的数据，则 `trackingUrl` 参数返回 `NULL`。 |
| status（状态）        | 加载作业的 HTTP 请求状态。有效值：`OK` 和 `Fail` |
| msg（消息）           | 加载作业的 HTTP 请求错误信息 |

### 取消加载作业

当加载作业不处于 **CANCELLED** 或 **FINISHED** 阶段时，可以使用 [CANCEL LOAD](../sql-reference/sql-statements/data-manipulation/CANCEL_LOAD.md) 语句取消该作业。

例如，您可以执行以下语句取消 `test_db` 数据库中标签为 `label1` 的加载作业：

```SQL
CANCEL LOAD
FROM test_db
WHERE LABEL = "label";
```

## 作业拆分和并发运行

Broker Load 作业可以拆分为一个或多个并发运行的任务。加载作业中的任务在单个事务中运行。它们必须全部成功或全部失败。StarRocks 根据您在 `LOAD` 语句中声明的 `data_desc` 对每个加载作业进行拆分：

- 如果声明了多个 `data_desc` 参数，每个参数指定一个不同的表，则会生成一个任务来加载每个表的数据。

- 如果声明了多个 `data_desc` 参数，每个参数都为同一表指定了不同的分区，则会生成一个任务来加载每个分区的数据。

此外，每个任务还可以进一步拆分为一个或多个实例，这些实例均匀分布在 StarRocks 集群的 BE 上并发运行。StarRocks 根据以下 [FE 配置项](../administration/FE_configuration.md#fe-configuration-items) 对每个任务进行拆分：

- `min_bytes_per_broker_scanner`：每个实例处理的最小数据量。默认值为 64 MB。

- `load_parallel_instance_num`：单个 BE 上每个加载作业中允许的并发实例数。默认值为 1。
  
  您可以使用以下公式计算单个任务中的实例数：

  **单个任务中的实例数 = min（单个任务要加载的数据量/`min_bytes_per_broker_scanner`，`load_parallel_instance_num` x BE 数）**

在大多数情况下，每个加载作业只声明一个 `data_desc`，每个加载作业只拆分为一个任务，并且该任务拆分为与 BE 数量相同的实例数。

## 相关配置项

[FE 配置项](../administration/FE_configuration.md#fe-configuration-items) `max_broker_load_job_concurrency` 指定了 StarRocks 集群中可以并发运行的最大 Broker Load 作业数量。
在 StarRocks v2.4 及之前的版本中，如果特定时间段内提交的 Broker Load 作业总数超过最大数量，则过多的作业会根据其提交时间进行排队和调度。

从 StarRocks v2.5 开始，如果在特定时间段内提交的 Broker Load 作业总数超过最大数量，则过多的作业会根据其优先级进行排队和调度。您可以通过在创建作业时使用 `priority` 参数来指定作业的优先级。请参阅 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#opt_properties)。您还可以使用 [ALTER LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_LOAD.md) 来修改处于 **QUEUEING** 或 **LOADING** 状态的现有作业的优先级。