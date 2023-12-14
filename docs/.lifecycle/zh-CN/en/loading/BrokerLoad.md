---
displayed_sidebar: "English"
---

# 从HDFS或云存储加载数据

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocks提供了基于MySQL的Broker Load加载方法，以帮助您从HDFS或云存储加载大量数据到StarRocks中。

Broker Load以异步加载模式运行。在提交加载作业后，StarRocks会异步运行该作业。您需要使用[SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)语句或`curl`命令来检查作业的结果。

Broker Load支持单表加载和多表加载。您可以通过运行一个Broker Load作业将一个或多个数据文件加载到一个或多个目标表中。Broker Load确保每个加载作业的事务原子性，即在加载多个数据文件的一次加载作业中，所有数据文件的加载必须全部成功或全部失败。永远不会出现这样的情况，即某些数据文件的加载成功而其他数据文件的加载失败。

Broker Load在数据加载时支持数据转换，并支持在数据加载期间由UPSERT和DELETE操作进行的数据更改。有关更多信息，请参见[数据加载时的数据转换](../loading/Etl_in_loading.md)和[通过加载更改数据](../loading/Load_to_Primary_Key_tables.md)。

<InsertPrivNote />

## 背景信息

在v2.4及更早版本中，StarRocks在运行Broker Load作业时依赖于broker来建立您的StarRocks集群与外部存储系统之间的连接。因此，在运行加载语句时，您需要输入`WITH BROKER "<broker_name>"`来指定要在加载语句中使用的broker。这称为“基于broker的加载”。broker是一个独立的、无状态的服务，集成了一个文件系统接口。有了broker，StarRocks就可以访问和读取存储在您的外部存储系统中的数据文件，并可以利用其自己的计算资源对这些数据文件的数据进行预处理和加载。

从v2.5开始，StarRocks在运行Broker Load作业时不再依赖于broker来建立您的StarRocks集群与外部存储系统之间的连接。因此，在加载语句中不再需要指定broker，但您仍需要保留`WITH BROKER`关键字。这称为“免broker加载”。

当您的数据存储在HDFS中时，可能会遇到免broker加载无法工作的情况。当您的数据存储在跨多个HDFS集群或已配置多个Kerberos用户时，可能会发生这种情况。在这些情况下，您可以转而使用基于broker的加载。为了成功地实现这一点，请确保至少部署了一个独立的broker组。有关如何在这些情况下指定身份验证配置和HA配置的信息，请参见[HDFS](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#hdfs)。

> **注意**
>
> 您可以使用[SHOW BROKER](../sql-reference/sql-statements/Administration/SHOW_BROKER.md)语句来检查部署在StarRocks集群中的broker。如果没有部署broker，可以按照[部署broker](../deployment/deploy_broker.md)中提供的说明部署broker。

## 支持的数据文件格式

Broker Load支持以下数据文件格式：

- CSV

- Parquet

- ORC

> **注意**
>
> 对于CSV数据，请注意以下几点：
>
> - 您可以使用UTF-8字符串，如逗号（,）、制表符或竖线（|），其长度不超过50个字节作为文本分隔符。
> - 空值使用`\N`表示。例如，一个数据文件由三列组成，而该数据文件的一条记录包含第一列和第三列的数据，但不包含第二列的数据。在这种情况下，您需要在第二列中使用`\N`表示空值。这意味着记录必须编译为`a,\N,b`而不是`a,,b`。`a,,b`表示记录的第二列包含空字符串。

## 支持的存储系统

Broker Load支持以下存储系统：

- HDFS

- AWS S3

- Google GCS

- 其他兼容S3的存储系统，如MinIO

- Microsoft Azure Storage

## 工作原理

在将加载作业提交给FE后，FE会生成查询计划，根据可用BE数量和要加载的数据文件大小将查询计划分割成部分，然后将查询计划的每个部分分配给一个可用的BE。在加载过程中，每个涉及的BE会从您的HDFS或云存储系统中获取数据文件的数据，对数据进行预处理，然后将数据加载到您的StarRocks集群中。当所有BE完成其部分的查询计划后，FE会确定加载作业是否成功。

以下图示显示了Broker Load作业的工作流程。

![Broker Load的工作流程](../assets/broker_load_how-to-work_en.png)

## 基本操作

### 创建多表加载作业

本主题以CSV为例，描述了如何将多个数据文件加载到多个表中。有关如何加载其他文件格式的数据以及Broker Load的语法和参数描述，请参见[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

请注意，在StarRocks中，SQL语言中使用了一些字面量作为保留关键字。在SQL语句中不要直接使用这些关键字。如果想在SQL语句中使用这样的关键字，请使用一对反引号(`)括起来。请参见[关键字](../sql-reference/sql-statements/keywords.md)。

#### 数据示例

1. 在本地文件系统中创建CSV文件。

   a. 创建名为`file1.csv`的CSV文件。该文件包含三列，依次表示用户ID、用户名和用户分数。

      ```Plain
      1,Lily,23
      2,Rose,23
      3,Alice,24
      4,Julia,25
      ```

   b. 创建名为`file2.csv`的CSV文件。该文件包含两列，依次表示城市ID和城市名。

      ```Plain
      200,'Beijing'
      ```

2. 在StarRocks数据库`test_db`中创建StarRocks表。

   > **注意**
   >
   > 自v2.5.7起，StarRocks在创建表或添加分区时可以自动设置桶的数量（BUCKETS）。您不再需要手动设置桶的数量。有关详细信息，请参见[确定桶的数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   a. 创建名为`table1`的主键表。该表包含三列：`id`、`name`和`score`，其中`id`是主键。

      ```SQL
      CREATE TABLE `table1`
      (
          `id` int(11) NOT NULL COMMENT "用户ID",
          `name` varchar(65533) NULL DEFAULT "" COMMENT "用户名称",
          `score` int(11) NOT NULL DEFAULT "0" COMMENT "用户分数"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`)
      DISTRIBUTED BY HASH(`id`);
      ```

   b. 创建名为`table2`的主键表。该表包含两列：`id`和`city`，其中`id`是主键。

      ```SQL
      CREATE TABLE `table2`
      (
          `id` int(11) NOT NULL COMMENT "城市ID",
          `city` varchar(65533) NULL DEFAULT "" COMMENT "城市名称"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`)
      DISTRIBUTED BY HASH(`id`);
      ```

3. 将`file1.csv`和`file2.csv`上传到您的HDFS集群的`/user/starrocks/`路径，上传到您的AWS S3存储桶`bucket_s3`的`input`文件夹，上传到您的Google GCS存储桶`bucket_gcs`的`input`文件夹，上传到您的MinIO存储桶`bucket_minio`的`input`文件夹，以及上传到您的Azure Storage的指定路径中。

#### 从HDFS加载数据

执行以下语句，将`file1.csv`和`file2.csv`从您的HDFS集群的`/user/starrocks`路径加载到`table1`和`table2`中：

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

在上述示例中，“StorageCredentialParams”表示一组根据所选身份验证方法而变化的身份验证参数。有关更多信息，请参见[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#hdfs)。

#### 从AWS S3加载数据

执行以下语句，将`file1.csv`和`file2.csv`从您的AWS S3存储桶`bucket_s3`的`input`文件夹分别加载到`table1`和`table2`中：

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
> 代理加载仅支持通过S3A协议访问AWS S3。因此，当您从AWS S3加载数据时，必须将S3 URI中的`s3://`替换为`s3a://`。

在上述示例中，“StorageCredentialParams”表示一组根据所选身份验证方法而变化的身份验证参数。有关更多信息，请参见[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#aws-s3)。

从v3.1开始，StarRocks支持使用INSERT命令和TABLE关键字直接从AWS S3加载Parquet格式或ORC格式文件的数据，无需首先创建外部表。有关更多信息，请参见[使用INSERT加载数据 > 使用TABLE关键字直接从外部源文件插入数据](../loading/InsertInto.md#insert-data-directly-from-files-in-an-external-source-using-table-keyword)。

#### 从Google GCS加载数据

执行以下语句，将`file1.csv`和`file2.csv`从您的Google GCS存储桶`bucket_gcs`的`input`文件夹分别加载到`table1`和`table2`中：

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
> 代理加载仅支持通过gs协议访问Google GCS。因此，当您从Google GCS加载数据时，必须将`gs://`作为文件路径的前缀。

在上述示例中，“StorageCredentialParams”表示一组根据所选身份验证方法而变化的身份验证参数。有关更多信息，请参见[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#google-gcs)。

#### 从其他S3兼容存储系统加载数据

以MinIO为例。您可以执行以下语句，将`file1.csv`和`file2.csv`从您的MinIO存储桶`bucket_minio`的`input`文件夹分别加载到`table1`和`table2`中：

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

在上述示例中，“StorageCredentialParams”表示一组根据所选身份验证方法而变化的身份验证参数。有关更多信息，请参见[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#other-s3-compatible-storage-system)。

#### 从Microsoft Azure Storage加载数据

执行以下语句，从Azure Storage的指定路径加载`file1.csv`和`file2.csv`：

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
> 从Azure Storage加载数据时，您需要根据访问协议和您使用的特定存储服务确定要使用的前缀。上述示例以Blob Storage为例。
>
> - 当您从Blob Storage加载数据时，必须根据用于访问您的存储帐户的协议，在文件路径中包含`wasb://`或`wasbs://`作为前缀：
>   - 如果Blob Storage只允许通过HTTP访问，请使用`wasb://`作为前缀，例如，`wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`。
>   - 如果Blob Storage只允许通过HTTPS访问，请使用`wasbs://`作为前缀，例如，`wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`。
> - 当您从Data Lake Storage Gen1加载数据时，必须在文件路径中包含`adl://`作为前缀，例如，`adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`。
> - 当您从Data Lake Storage Gen2加载数据时，必须根据用于访问您的存储帐户的协议，在文件路径中包含`abfs://`或`abfss://`作为前缀：
>   - 如果Data Lake Storage Gen2只允许通过HTTP访问，请使用`abfs://`作为前缀，例如，`abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。
>   - 如果Data Lake Storage Gen2只允许通过HTTPS访问，请使用`abfss://`作为前缀，例如，`abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。

在上述示例中，“StorageCredentialParams”表示一组根据所选身份验证方法而变化的身份验证参数。有关更多信息，请参见[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#microsoft-azure-storage)。

#### 查询数据

在从您的HDFS集群、AWS S3存储桶或Google GCS存储桶加载数据完成后，您可以使用SELECT语句查询StarRocks表的数据，以验证加载是否成功。

1. 执行以下语句查询`table1`的数据：

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

2. 执行以下语句查询`table2`的数据：

   ```SQL
   MySQL [test_db]> SELECT * FROM table2;
   +------+--------+
   | id   | city   |
   +------+--------+
   | 200  | Beijing|
   +------+--------+
   4 rows in set (0.01 sec)
   ```

### 创建单表加载任务

您还可以将单个数据文件或指定路径中的所有数据文件加载到单个目标表中。假设您的AWS S3存储桶`bucket_s3`包含名为`input`的文件夹。`input`文件夹包含多个数据文件，其中之一名为`file1.csv`。这些数据文件的列数与`table1`的列数相同，并且每个数据文件的列都可以按顺序一对一地映射到`table1`的列。

要将`file1.csv`加载到`table1`中，请执行以下语句：

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

要将`input`文件夹中的所有数据文件加载到`table1`中，请执行以下语句：

```SQL
LOAD LABEL test_db.label_8
(
```javascript
    数据输入文件（"s3a://bucket_s3/input/*"）
    到表table1
    列终止由","
    格式为“CSV”
）
使用BROKER
(
    StorageCredentialParams
);
```

在上面的示例中，`StorageCredentialParams`代表一组根据所选的认证方法而变化的认证参数。有关更多信息，请参见[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#aws-s3)。

### 查看加载作业

经纪人加载允许您使用SHOW LOAD语句或`curl`命令来查看加载作业。

#### 使用SHOW LOAD

有关更多信息，请参见[SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)。

#### 使用curl

语法如下：

```Bash
curl --location-trusted -u <username>:<password> \
    'http://<fe_host>:<fe_http_port>/api/<database_name>/_load_info?label=<label_name>'
```

> **注意**
>
> 如果您使用的帐号未设置密码，则只需输入`<username>：`。

例如，您可以运行以下命令来查看数据库`test_db`中标签为`label1`的加载作业的信息：

```Bash
curl --location-trusted -u <username>:<password> \
    'http://<fe_host>:<fe_http_port>/api/test_db/_load_info?label=label1'
```

`curl`命令以JSON对象`jobInfo`的形式返回了关于加载作业的信息：

```JSON
{"jobInfo":{"dbName":"default_cluster:test_db","tblNames":["table1_simple"],"label":"label1","state":"FINISHED","failMsg":"","trackingUrl":""},"status":"OK","msg":"Success"}%
```

以下表格描述了`jobInfo`中的参数。

| **参数**     | **描述**                                                     |
| ------------ | ------------------------------------------------------------ |
| dbName       | 加载数据的数据库名称                                         |
| tblNames     | 加载数据的表名称。                                           |
| label        | 加载作业的标签。                                             |
| state        | 加载作业的状态。有效值：<ul><li>`PENDING`：加载作业排队等待被调度。</li><li>`QUEUEING`：加载作业在队列中等待被调度。</li><li>`LOADING`：加载作业正在运行。</li><li>`PREPARED`：事务已提交。</li><li>`FINISHED`：加载作业成功。</li><li>`CANCELLED`：加载作业失败。</li></ul>有关更多信息，请参见[数据加载概述](../loading/Loading_intro.md)中的“异步加载”部分。 |
| failMsg      | 加载作业失败的原因。如果加载作业的`state`值为`PENDING`、`LOADING`或`FINISHED`，则对于`failMsg`参数返回`NULL`。如果加载作业的`state`值为`CANCELLED`，则对于`failMsg`参数返回的值由两部分组成：`type`和`msg`。<ul><li>`type`部分可以是以下任何值：</li><ul><li>`USER_CANCEL`：加载作业被手动取消。</li><li>`ETL_SUBMIT_FAIL`：无法提交加载作业。</li><li>`ETL-QUALITY-UNSATISFIED`：加载作业失败，因为不合格数据的百分比超过`max-filter-ratio`参数的值。</li><li>`LOAD-RUN-FAIL`：加载作业在`LOADING`阶段失败。</li><li>`TIMEOUT`：加载作业未能在指定的超时期限内完成。</li><li>`UNKNOWN`：由于未知错误，加载作业失败。</li></ul><li>`msg`部分提供了加载失败的详细原因。</li></ul> |
| trackingUrl  | 用于访问加载作业中检测到的不合格数据的URL。您可以使用`curl`或`wget`命令访问URL并获取不合格数据。如果未检测到不合格数据，则对于`trackingUrl`参数返回`NULL`。 |
| status       | 加载作业的HTTP请求状态。有效值：`OK`和`Fail`。               |
| msg          | 加载作业的HTTP请求的错误信息。                                |

### 取消加载作业

当加载作业处于**CANCELLED**或**FINISHED**阶段时，可以使用[CANCEL LOAD](../sql-reference/sql-statements/data-manipulation/CANCEL_LOAD.md)语句来取消作业。

例如，可以执行以下语句来取消数据库`test_db`中标签为`label1`的加载作业：

```SQL
CANCEL LOAD
FROM test_db
WHERE LABEL = "label";
```

## 作业拆分和并发运行

Broker Load作业可以拆分为一个或多个并发运行的任务。加载作业中的任务在单个事务内运行。它们必须全部成功或全部失败。StarRocks根据您在`LOAD`语句中声明`data_desc`的方式，对每个加载作业进行拆分：

- 如果声明了多个`data_desc`参数，并且每个参数指定了一个不同的表，则会生成一个任务来加载每个表的数据。

- 如果声明了多个`data_desc`参数，每个参数指定了同一表的不同分区，则会生成一个任务来加载每个分区的数据。

此外，每个任务可以进一步拆分为一个或多个实例，这些实例均匀分布并在StarRocks集群的BE上并发运行。StarRocks根据以下[FE配置](../administration/Configuration.md#fe-configuration-items)对每个任务进行拆分：

- `min_bytes_per_broker_scanner`：每个实例处理的最小数据量。默认值为64 MB。

- `load_parallel_instance_num`：在单个BE上每个加载作业中允许的并发实例数量。默认值为1。

  您可以使用下列公式计算单个任务中实例的数量：

  **单个任务中的实例数量 = min(单个任务要加载的数据量/`min_bytes_per_broker_scanner`，`load_parallel_instance_num` x BE的数量)**

在大多数情况下，每个加载作业仅声明了一个`data_desc`，每个加载作业仅拆分为一个任务，且该任务拆分为与BE数量相同的实例数。

## 相关配置项

[FE配置项](../administration/Configuration.md#fe-configuration-items)`max_broker_load_job_concurrency`指定了StarRocks集群内可以同时运行的Broker Load作业的最大数量。

在StarRocks v2.4及更早版本中，如果特定时期内提交的Broker Load作业总数超过最大数量，多余的作业将排队并基于它们的提交时间进行调度。

自StarRocks v2.5起，如果特定时期内提交的Broker Load作业总数超过最大数量，多余的作业将根据其优先级排队并按优先级进行调度。您可以通过在作业创建时使用`priority`参数为作业指定优先级。有关更多信息，请参见[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#opt_properties)。还可以使用[ALTER LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_LOAD.md)修改处于**QUEUEING**或**LOADING**状态的现有作业的优先级。