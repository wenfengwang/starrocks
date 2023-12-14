```md
# Importing Data from HDFS or External Cloud Storage Systems


StarRocks provides a data import method called Broker Load, based on the MySQL protocol, to help you import large amounts of data from HDFS or external cloud storage systems.

Broker Load is an asynchronous data import method. After you submit the import job, StarRocks will execute the job asynchronously. You can check the result of the import job using the [SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) statement or the curl command.

Broker Load supports Single-Table Load and Multi-Table Load. You can import one or more data files into one or more target tables in a single import operation. Broker Load ensures the atomicity of a single import transaction, meaning that multiple data files are all successfully imported or failed together, without partial success or failure.

Broker Load supports data transformation and data changes through UPSERT and DELETE operations during the import process. Please refer to [Data Transformation During Import](./Etl_in_loading.md) and [Data Changes Through Import](../loading/Load_to_Primary_Key_tables.md).

> **Note**

>
> Broker Load operation requires INSERT permission for the target table. If your user account does not have INSERT permission, please refer to [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) to grant permission to the user.


## Background Information

Prior to version 2.4, StarRocks required a Broker to access external storage systems when performing Broker Load, known as "Broker-required import". The import statement needed to specify which Broker to use through `WITH BROKER "<broker_name>"`. Broker is an independent stateless service that encapsulates the file system interface. Through Broker, StarRocks can access and read data files on external storage systems and preprocess and import data files using its own computing resources.

Since version 2.5, StarRocks no longer requires a Broker to access external storage systems when performing Broker Load, known as "Brokerless import". The import statement no longer needs to specify `broker_name`, but still retains the `WITH BROKER` keyword.

It is worth noting that Brokerless import is restricted in certain scenarios where the data source is HDFS, such as in multi-HDFS clusters or multi-Kerberos user scenarios. In these scenarios, Broker-required import can still be used, but it is necessary to ensure that at least one set of independent Brokers is deployed. For information on how to specify authentication methods and HA configurations in various scenarios, please refer to [HDFS](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md##hdfs).

> **Note**
>
> You can use the [SHOW BROKER](../sql-reference/sql-statements/Administration/SHOW_BROKER.md) statement to view the Brokers deployed in the StarRocks cluster. If no Broker is deployed in the cluster, please refer to [Deploying Broker Nodes](../deployment/deploy_broker.md) to complete the Broker deployment.

## Supported Data File Formats

Broker Load supports the following data file formats:

- CSV

- Parquet

- ORC

> **Note**
>
> For CSV formatted data, please note the following:
>
> - StarRocks supports setting a UTF-8 encoded string with a maximum length of 50 bytes as the column delimiter, including common delimiters such as comma (,), Tab, and Pipe (|).
> - The null value is represented by `\N`. For example, if a data file has three columns and the data in a certain row for the first and third columns are `a` and `b` respectively, and the second column has no data, then the empty value for the second column should be represented as `\N`, written as `a,\N,b`, not `a,,b`. `a,,b` represents the second column as an empty string.

## Supported External Storage Systems

Broker Load supports data import from the following external storage systems:

- HDFS

- AWS S3

- Google GCS

- Alibaba Cloud OSS

- Tencent Cloud COS

- Huawei Cloud OBS

- Other S3 protocol-compatible object storage systems (e.g., MinIO)

- Microsoft Azure Storage

## Basic Principles

After submitting the import job, the FE will generate a corresponding query plan and allocate the query plan to multiple BEs for execution based on the number of available BEs and the size of the source data file. Each BE is responsible for executing a portion of the import task. During execution, the BE pulls data from HDFS or cloud storage systems and imports the data into StarRocks after preprocessing. Once all BEs have completed the import, the FE determines the success of the import job.

The following diagram illustrates the main process of Broker Load:

![Broker Load Principle Diagram](../assets/broker_load_how-to-work_zh.png)

## Basic Operations

### Creating a Multi-Table Load Job

Using CSV-formatted data as an example, this section describes how to import multiple data files into multiple destination tables. For detailed syntax and parameter information on importing data in other formats and using Broker Load, please refer to [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md).

Note that in StarRocks, some words are reserved keywords of the SQL language and cannot be used directly in SQL statements. To use these keywords in SQL statements, they must be enclosed in backticks (`). Refer to [Keywords](../sql-reference/sql-statements/keywords.md).

#### Data Example

1. Create a CSV-formatted data file in the local file system.

   a. Create a data file named `file1.csv`. The file contains three columns representing user ID, user name, and user score, as follows:

      ```Plain
      1,Lily,23
      2,Rose,23
      3,Alice,24
      4,Julia,25
      ```

   b. Create a data file named `file2.csv`. The file contains two columns representing city ID and city name, as follows:

      ```Plain
      200,'Beijing'
      ```

2. Create a StarRocks table in the `test_db` database.

   > **Note**
   >
   > Since version 2.5, StarRocks supports automatically setting the number of buckets (BUCKETS) when creating tables and adding partitions. You do not need to manually set the number of buckets. For more information, please refer to [Determining the Number of Buckets](../table_design/Data_distribution.md#determining-the-number-of-buckets).

   a. Create a primary key model table named `table1`. The table contains three columns, `id`, `name`, and `score`, representing user ID, user name, and user score, respectively. The primary key is the `id` column, as follows:

      ```SQL
      CREATE TABLE `table1`
      (
          `id` int(11) NOT NULL COMMENT "User ID",
          `name` varchar(65533) NULL DEFAULT "" COMMENT "User name",
          `score` int(11) NOT NULL DEFAULT "0" COMMENT "User score"
      )
          ENGINE=OLAP
          PRIMARY KEY(`id`)
          DISTRIBUTED BY HASH(`id`);
      ```

3. 创建名为 `table2` 的主键模型表。表包含 `id` 和 `city` 两列，分别代表城市 ID 和城市名称，主键为 `id` 列，具体结构如下:

      ```SQL
      CREATE TABLE `table2`
      (
          `id` int(11) NOT NULL COMMENT "城市 ID",
          `city` varchar(65533) NULL DEFAULT "" COMMENT "城市名称"
      )
          ENGINE=OLAP
          PRIMARY KEY(`id`)
          DISTRIBUTED BY HASH(`id`);
   ```

3. 将创建好的数据文件 `file1.csv` 和 `file2.csv` 分别上传至 HDFS 集群的 `/user/starrocks/` 路径下、AWS S3 存储空间 `bucket_s3` 的 `input` 文件夹、Google GCS 存储空间 `bucket_gcs` 的 `input` 文件夹、阿里云 OSS 存储空间 `bucket_oss` 的 `input` 文件夹、腾讯云 COS 存储空间 `bucket_cos` 的 `input` 文件夹、华为云 OBS 存储空间 `bucket_obs` 的 `input` 文件夹、以及其他兼容 S3 协议的对象存储空间（如 MinIO） `bucket_minio` 的 `input` 文件夹，以及 Azure Storage 中的指定路径下。

#### 从 HDFS 导入

可以使用以下语句，将 HDFS 集群 `/user/starrocks/` 路径下的 CSV 文件 `file1.csv` 和 `file2.csv` 分别导入到 StarRocks 表 `table1` 和 `table2` 中：

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

以上示例中，`StorageCredentialParams` 表示一组认证参数。具体包含哪些参数需要根据您使用的认证方式来确定，请参见 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#hdfs)。

从 3.1 版本起，StarRocks 支持使用 INSERT 语句和 TABLE 关键字直接从 HDFS 导入 Parquet 或 ORC 格式的数据文件，避免了需要先创建外部表的麻烦。有关详情，请参见 [通过 INSERT 语句导入数据 > 通过 TABLE 关键字直接导入外部数据文件](../loading/InsertInto.md#通过-insert-into-select-以及表函数-files-导入外部数据文件)。

#### 从 AWS S3 导入

可以通过以下语句，将 AWS S3 存储空间 `bucket_s3` 的 `input` 文件夹中的 CSV 文件 `file1.csv` 和 `file2.csv` 分别导入到 StarRocks 表 `table1` 和 `table2` 中：

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

> **说明**
>
> 由于 Broker Load 只支持通过 S3A 协议访问 AWS S3，因此在从 AWS S3 导入数据时，`DATA INFILE` 中传入的目标文件的 S3 URI，前缀必须将 `s3://` 修改为 `s3a://`。

从 3.1 版本起，StarRocks 支持使用 INSERT 语句和 TABLE 关键字直接从 AWS S3 导入 Parquet 或 ORC 格式的数据文件，避免了需要先创建外部表的麻烦。有关详情，请参见 [通过 INSERT 语句导入数据 > 通过 TABLE 关键字直接导入外部数据文件](../loading/InsertInto.md#通过-insert-into-select-以及表函数-files-导入外部数据文件)。

#### 从 Google GCS 导入

可以使用以下语句，将 Google GCS 存储空间 `bucket_gcs` 的 `input` 文件夹中的 CSV 文件 `file1.csv` 和 `file2.csv` 分别导入到 StarRocks 表 `table1` 和 `table2` 中：

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


> **说明**
>
> 由于 Broker Load 只支持通过 gs 协议访问 Google GCS，因此在从 Google GCS 导入数据时，请确保文件路径传入的目标文件的 GCS URI 使用 `gs://` 作为前缀。

以上示例中，`StorageCredentialParams` 表示一组认证参数。具体包含哪些参数需要根据您使用的认证方式定制，详情请参见 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#google-gcs)。

#### 从阿里云 OSS 导入

可以通过以下语句，将阿里云 OSS 存储空间 `bucket_oss` 的 `input` 文件夹中的 CSV 文件 `file1.csv` 和 `file2.csv` 分别导入到 StarRocks 表 `table1` 和 `table2` 中：

```SQL
LOAD LABEL test_db.label4
(
    DATA INFILE("oss://bucket_oss/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("oss://bucket_oss/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, city)
)
WITH BROKER
(
    StorageCredentialParams
);
```

以上示例中，`StorageCredentialParams` 表示一组认证参数。具体包含哪些参数需要根据您使用的认证方式定制，详情请参见 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#阿里云-oss)。

#### 从腾讯云 COS 导入

可以通过以下语句，将腾讯云 COS 存储空间 `bucket_cos` 的 `input` 文件夹中的 CSV 文件 `file1.csv` 和 `file2.csv` 分别导入到 StarRocks 表 `table1` 和 `table2` 中：

```SQL
LOAD LABEL test_db.label5
(
    DATA INFILE("cosn://bucket_cos/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("cosn://bucket_cos/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, city)
)
WITH BROKER
(
    StorageCredentialParams
);
```

在上面的示例中，`StorageCredentialParams` 表示一组认证参数。具体包含哪些参数需要根据您使用的认证方式来确定。有关详细信息，请参见 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#腾讯云-cos)。

#### 从华为云 OBS 导入

您可以使用以下语句，将华为云 OBS 存储空间 `bucket_obs` 中 `input` 文件夹内的 CSV 文件 `file1.csv` 和 `file2.csv` 分别导入到 StarRocks 表 `table1` 和 `table2` 中：

```SQL
LOAD LABEL test_db.label6
(
    DATA INFILE("obs://bucket_obs/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("obs://bucket_obs/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, city)
)
WITH BROKER
(
    StorageCredentialParams
);
```

> **说明**
>
> 在从华为云 OBS 导入数据时，您需要先下载[依赖库](https://github.com/huaweicloud/obsa-hdfs/releases/download/v45/hadoop-huaweicloud-2.8.3-hw-45.jar)并将其添加到 **$BROKER_HOME/lib/** 路径下，然后重新启动 Broker。

在上述示例中，`StorageCredentialParams` 表示一组认证参数。具体包含哪些参数需要根据您使用的认证方式来确定。有关详细信息，请参见 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#华为云-obs)。

#### 从其他兼容 S3 协议的对象存储导入

您可以使用以下语句，将兼容 S3 协议的对象存储空间（如MinIO） `bucket_minio` 中 `input` 文件夹内的 CSV 文件 `file1.csv` 和 `file2.csv` 分别导入到 StarRocks 表 `table1` 和 `table2` 中：

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

在上述示例中，`StorageCredentialParams` 表示一组认证参数。具体包含哪些参数需要根据您使用的认证方式来确定。有关详细信息，请参见 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#其他兼容-s3-协议的对象存储)。

#### 从 Microsoft Azure Storage 导入

通过以下语句，您可以将 Azure Storage 指定路径下的 CSV 文件 `file1.csv` 和 `file2.csv` 分别导入到 StarRocks 表 `table1` 和 `table2` 中：

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
> 在从 Azure Storage 导入数据时，您需要根据使用的访问协议和存储服务来确定文件路径中的前缀。上述示例以 Blob Storage 为例。
>
> - 从 Blob Storage 导入数据时，需要根据使用的访问协议在文件路径里添加 `wasb://` 或 `wasbs://` 作为前缀：
>   - 如果使用 HTTP 协议进行访问，请使用 `wasb://` 作为前缀，例如，`wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`。
>   - 如果使用 HTTPS 协议进行访问，请使用 `wasbs://` 作为前缀，例如，`wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`。
> - 从 Azure Data Lake Storage Gen1 导入数据时，需要在文件路径里添加 `adl://` 作为前缀，例如， `adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`。
> - 从 Data Lake Storage Gen2 导入数据时，需要根据使用的访问协议在文件路径里添加 `abfs://` 或 `abfss://` 作为前缀：
>   - 如果使用 HTTP 协议进行访问，请使用 `abfs://` 作为前缀，例如，`abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。
>   - 如果使用 HTTPS 协议进行访问，请使用 `abfss://` 作为前缀，例如，`abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。

在上述示例中，`StorageCredentialParams` 表示一组认证参数。具体包含哪些参数需要根据您使用的认证方式来确定。有关详细信息，请参见 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#microsoft-azure-storage)。

#### 查询数据

在从HDFS、AWS S3、Google GCS、阿里云OSS、腾讯云COS或者华为云OBS导入完成后，您可以使用 SELECT 语句来查看 StarRocks 表的数据，验证数据已经成功导入。

1. 查询 `table1` 表的数据，如下所示：

   ```SQL
   SELECT * FROM table1;
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

2. 查询 `table2` 表的数据，如下所示：

   ```SQL
   SELECT * FROM table2;
   +------+--------+
   | id   | city   |
   +------+--------+
   | 200  | 北京    |
   +------+--------+
   4 rows in set (0.01 sec)
   ```

### 创建单表导入 (Single-Table Load) 作业

您还可以指定导入一个数据文件或者一个路径下所有数据文件到一张目标表。假设您的AWS S3存储空间 `bucket_s3` 里 `input` 文件夹下包含多个数据文件，其中一个数据文件名为 `file1.csv`。这些数据文件与目标表 `table1` 包含的列数相同，并且这些列能按顺序一一对应到目标表 `table1` 中的列。

如果要把数据文件 `file1.csv` 导入到目标表 `table1` 中，可以执行如下语句:

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
)；
```

如果要把 `input` 文件夹下的所有数据文件都导入到目标表 `table1` 中，可以执行如下语句:

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
)；
```

### 查看导入作业

Broker Load 支持通过 SHOW LOAD 语句和 curl 命令两种方式来查看导入作业的执行情况。

#### 使用 SHOW LOAD 语句

请参见 [SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)。

#### 使用 curl 命令

命令语法如下：

```Bash

curl --location-trusted -u <username>:<password> \
    'http://<fe_host>:<fe_http_port>/api/<database_name>/_load_info?label=<label_name>'
```


> **说明**
>
> 如果账号没有设置密码，这里只需要传入 `<username>:`。

例如，可以通过如下命令查看 `db1` 数据库中标签为 `label1` 的导入作业的执行情况：

```Bash
curl --location-trusted -u <username>:<password> \
    'http://<fe_host>:<fe_http_port>/api/db1/_load_info?label=label1'
```

命令执行后，以 JSON 格式返回导入作业的结果信息 `jobInfo`，如下所示：

```JSON
{"jobInfo":{"dbName":"default_cluster:db1","tblNames":["table1_simple"],"label":"label1","state":"FINISHED","failMsg":"","trackingUrl":""},"status":"OK","msg":"Success"}%
```

`jobInfo` 中包含如下参数：

| **参数**    | **说明**                                                     |
| ----------- | ------------------------------------------------------------ |
| dbName      | 目标 StarRocks 表所在的数据库的名称。                               |
| tblNames    | 目标 StarRocks 表的名称。                        |
| label       | 导入作业的标签。                                             |
| state       | 导入作业的状态，包括：<ul><li>`PENDING`：导入作业正在等待执行中。</li><li>`QUEUEING`：导入作业正在等待执行中。</li><li>`LOADING`：导入作业正在执行中。</li><li>`PREPARED`：事务已提交。</li><li>`FINISHED`：导入作业成功。</li><li>`CANCELLED`：导入作业失败。</li></ul>请参见[异步导入](./Loading_intro.md#异步导入)。 |
| failMsg     | 导入作业的失败原因。当导入作业的状态为`PENDING`，`LOADING`或`FINISHED`时，该参数值为`NULL`。当导入作业的状态为`CANCELLED`时，该参数值包括 `type` 和 `msg` 两部分：<ul><li>`type` 包括如下取值：</li><ul><li>`USER_CANCEL`：导入作业被手动取消。</li><li>`ETL_SUBMIT_FAIL`：导入任务提交失败。</li><li>`ETL-QUALITY-UNSATISFIED`：数据质量不合格，即导入作业的错误数据率超过了 `max-filter-ratio`。</li><li>`LOAD-RUN-FAIL`：导入作业在 `LOADING` 状态失败。</li><li>`TIMEOUT`：导入作业未在允许的超时时间内完成。</li><li>`UNKNOWN`：未知的导入错误。</li></ul><li>`msg` 显示有关失败原因的详细信息。</li></ul> |
| trackingUrl | 导入作业中质量不合格数据的访问地址。可以使用 `curl` 命令或 `wget` 命令访问该地址。如果导入作业中不存在质量不合格的数据，则返回空值。 |
| status      | 导入请求的状态，包括 `OK` 和 `Fail`。                        |
| msg         | HTTP 请求的错误信息。                                        |

### 取消导入作业

当导入作业状态不为 **CANCELLED** 或 **FINISHED** 时，可以通过 [CANCEL LOAD](../sql-reference/sql-statements/data-manipulation/CANCEL_LOAD.md) 语句来取消该导入作业。

例如，可以通过以下语句，撤销 `db1` 数据库中标签为 `label1` 的导入作业：

```SQL
CANCEL LOAD
FROM db1
WHERE LABEL = "label";
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

## 相关配置项

[FE 配置项](../administration/Configuration.md#fe-配置项) `max_broker_load_job_concurrency` 指定了 StarRocks 集群中可以并行执行的 Broker Load 作业的最大数量。

StarRocks v2.4 及以前版本中，如果某一时间段内提交的 Broker Load 作业总数超过最大数量，则超出作业会按照各自的提交时间放到队列中排队等待调度。

StarRocks v2.5 版本中，如果某一时间段内提交的 Broker Load 作业总数超过最大数量，则超出的作业会按照作业创建时指定的优先级被放到队列中排队等待调度。参见 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#opt_properties) 文档中的可选参数 `priority`。您可以使用 [ALTER LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_LOAD.md) 语句修改处于 **QUEUEING** 状态或者 **LOADING** 状态的 Broker Load 作业的优先级。

## 常见问题

请参见 [Broker Load 常见问题](../faq/loading/Broker_load_faq.md)。