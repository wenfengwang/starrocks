---
displayed_sidebar: English
---


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

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 来查询作业结果。此功能从 v3.1 版本开始支持。有关更多信息，请参见本主题的“[查看加载作业](#view-a-load-job)”部分。

确认加载作业成功后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 来查询 `table1` 和 `table2` 的数据：

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

## 从 Google GCS 加载数据

请注意，Broker Load 仅支持根据 gs 协议访问 Google GCS。因此，当您从 Google GCS 加载数据时，您必须在作为文件路径 (`DATA INFILE`) 传递的 GCS URI 中包含 `gs://` 作为前缀。

另请注意，以下示例使用 CSV 文件格式和基于 VM 的身份验证方法。有关如何加载其他格式的数据以及使用其他身份验证方法时需要配置的身份验证参数的信息，请参阅 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

### 将单个数据文件加载到单个表中

#### 示例

执行以下语句将存储在您的 Google GCS 存储桶 `bucket_gcs` 的 `input` 文件夹中的 `file1.csv` 数据加载到 `table1` 中：

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

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 来查询作业结果。此功能从 v3.1 版本开始支持。有关更多信息，请参见本主题的“[查看加载作业](#view-a-load-job)”部分。

确认加载作业成功后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 来查询 `table1` 的数据：

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

执行以下语句将存储在您的 Google GCS 存储桶 `bucket_gcs` 的 `input` 文件夹中的所有数据文件（`file1.csv` 和 `file2.csv`）的数据加载到 `table1` 中：

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

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 来查询作业结果。此功能从 v3.1 版本开始支持。有关更多信息，请参见本主题的“[查看加载作业](#view-a-load-job)”部分。

确认加载作业成功后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 来查询 `table1` 的数据：

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

执行以下语句将您的 Google GCS 存储桶 `bucket_gcs` 的 `input` 文件夹中存储的 `file1.csv` 和 `file2.csv` 的数据分别加载到 `table1` 和 `table2` 中：

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
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 来查询作业结果。此功能从 v3.1 版本开始支持。有关更多信息，请参见本主题的“[查看加载作业](#view-a-load-job)”部分。

确认加载作业成功后，您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 来查询 `table1` 和 `table2` 的数据：

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
执行以下语句将存储在 Azure 存储指定路径中的 `file1.csv` 数据加载到 `table1` 中：

```SQL
LOAD LABEL test_db.label_brokerloadtest_301
(
    DATA INFILE("wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/file1.csv")
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

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 查询作业结果。此功能从 v3.1 起支持。有关详细信息，请参阅本主题的“[查看加载作业](#view-a-load-job)”部分。

确认加载作业成功后，可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询 `table1` 的数据：

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

执行以下语句将存储在 Azure 存储指定路径中的所有数据文件（`file1.csv` 和 `file2.csv`）的数据加载到 `table1` 中：

```SQL
LOAD LABEL test_db.label_brokerloadtest_302
(
    DATA INFILE("wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/*")
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

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 查询作业结果。此功能从 v3.1 起支持。有关详细信息，请参阅本主题的“[查看加载作业](#view-a-load-job)”部分。

确认加载作业成功后，可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询 `table1` 的数据：

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
8 rows in set (0.01 sec)
```

### 将多个数据文件加载到多个表中

#### 示例

执行以下语句将 Azure 存储指定路径中存储的 `file1.csv` 和 `file2.csv` 的数据分别加载到 `table1` 和 `table2` 中：

```SQL
LOAD LABEL test_db.label_brokerloadtest_303
(
    DATA INFILE("wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/file2.csv")
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

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 查询作业结果。此功能从 v3.1 起支持。有关详细信息，请参阅本主题的“[查看加载作业](#view-a-load-job)”部分。

确认加载作业成功后，可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询 `table1` 和 `table2` 的数据：

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

执行以下语句将您的 MinIO 存储桶 `bucket_minio` 的 `input` 文件夹中存储的 `file1.csv` 的数据加载到 `table1` 中：

```SQL
LOAD LABEL test_db.label_brokerloadtest_401
(
    DATA INFILE("s3://bucket_minio/input/file1.csv")
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

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 查询作业结果。此功能从 v3.1 起支持。有关详细信息，请参阅本主题的“[查看加载作业](#view-a-load-job)”部分。

确认加载作业成功后，可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询 `table1` 的数据：

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

执行以下语句将您的 MinIO 存储桶 `bucket_minio` 的 `input` 文件夹中存储的所有数据文件（`file1.csv` 和 `file2.csv`）的数据加载到 `table1` 中：

```SQL
LOAD LABEL test_db.label_brokerloadtest_402
(
    DATA INFILE("s3://bucket_minio/input/*")
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

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 查询作业结果。此功能从 v3.1 起支持。有关详细信息，请参阅本主题的“[查看加载作业](#view-a-load-job)”部分。

确认加载作业成功后，可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询 `table1` 的数据：

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
8 rows in set (0.01 sec)
```

### 将多个数据文件加载到多个表中

#### 示例

执行以下语句，将您的 MinIO 存储桶 `bucket_minio` 的 `input` 文件夹中存储的所有数据文件（`file1.csv` 和 `file2.csv`）的数据分别加载到 `table1` 和 `table2` 中：

```SQL
LOAD LABEL test_db.label_brokerloadtest_403
(
    DATA INFILE("s3://bucket_minio/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("s3://bucket_minio/input/file2.csv")
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
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 查询作业结果。此功能从 v3.1 起支持。有关详细信息，请参阅本主题的“[查看加载作业](#view-a-load-job)”部分。

确认加载作业成功后，可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询 `table1` 和 `table2` 的数据：

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
```SQL
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
);
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 查询数据

提交加载作业后，您可以使用 `SELECT * FROM information_schema.loads` 查询作业结果。此功能自 v3.1 起支持。更多信息，请参见本主题的“[查看加载作业](#view-a-load-job)”部分。

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

使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句查询 `information_schema` 数据库中 `loads` 表中一个或多个加载作业的结果。此功能自 v3.1 版本起支持。

示例 1：查询 `test_db` 数据库执行的加载作业的结果。在查询语句中，指定最多返回两个结果，并且结果必须按创建时间（`CREATE_TIME`）降序排列。

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'test_db'
ORDER BY create_time DESC
LIMIT 2\G
```

返回结果如下：

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

示例 2：查询 `test_db` 数据库上执行的加载作业（标签为 `label_brokerload_unqualifiedtest_82`）的结果：

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'test_db' AND label = 'label_brokerload_unqualifiedtest_82'\G
```

返回结果如下：

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

有关返回结果中字段的信息，请参阅 [信息架构 > 负载](../reference/information_schema/loads.md)。

## 取消加载作业

当加载作业未处于 **CANCELLED** 或 **FINISHED** 阶段时，您可以使用 [CANCEL LOAD](../sql-reference/sql-statements/data-manipulation/CANCEL_LOAD.md) 语句取消该作业。

例如，您可以执行以下语句取消数据库 `test_db` 中标签为 `label1` 的加载作业：

```SQL
CANCEL LOAD
FROM test_db
WHERE LABEL = "label1";
```

## 作业拆分和并发运行

Broker Load 作业可以拆分为一个或多个同时运行的任务。加载作业中的任务在单个事务中运行。它们必须全部成功或失败。StarRocks 根据您在 `LOAD` 语句中声明的 `data_desc` 拆分每个加载作业：

- 如果声明多个 `data_desc` 参数，每个参数指定一个不同的表，则会生成一个任务来加载每个表的数据。

- 如果声明多个 `data_desc` 参数，每个参数为同一个表指定不同的分区，则会生成一个任务来加载每个分区的数据。

此外，每项任务还可以进一步拆分为一个或多个实例，这些实例均匀分布到并且并发运行在您的 StarRocks 集群的 BE 上。StarRocks 根据以下 [FE 配置](../administration/FE_configuration.md#fe-configuration-items) 拆分每个任务：

- `min_bytes_per_broker_scanner`：每个实例处理的最小数据量。默认大小为 64 MB。
- `load_parallel_instance_num`：单个 BE 上每个加载作业允许的并发实例数。默认数量为 1。

  您可以使用以下公式来计算单个任务中的实例数：

  **单个任务中的实例数量** = min(单个任务要加载的数据量/`min_bytes_per_broker_scanner`，`load_parallel_instance_num` x BE数量)

在大多数情况下，每个加载作业只声明一个 `data_desc`，每个加载作业只分割为一个任务，并且该任务被分割为与 BE 数量相同的实例数。

## 故障排除

请参阅 [Broker Load 常见问题解答](../faq/loading/Broker_load_faq.md)。