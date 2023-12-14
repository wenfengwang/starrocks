---
displayed_sidebar: "Chinese"
---

# 从GCS加载数据

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocks提供从GCS加载数据的两种选项：

1. 使用Broker Load进行异步加载
2. 使用`FILES()`表函数进行同步加载

通常情况下，较小的数据集使用`FILES()`表函数进行同步加载，而较大的数据集常常使用Broker Load进行异步加载。这两种方法具有不同的优势，下面将对其进行描述。

<InsertPrivNote />

## 收集连接细节

> **注意**
>
> 示例中使用的是服务账户密钥认证。页面底部提供了其他认证方法的链接。
>
> 本指南使用的是StarRocks托管的数据集。该数据集可被任何经过身份验证的GCP用户读取，因此您可以使用自己的凭据来读取下文使用的Parquet文件。


从GCS加载数据需要具备以下信息：

- GCS存储桶
- GCS对象键（对象名），如果需要访问存储桶中的特定对象。请注意，对象键可以包括前缀，如果您的GCS对象存储在子文件夹中。有关完整语法，请参阅**更多信息**。
- GCS区域
- 服务账户访问密钥和秘钥

## 使用Broker Load

异步的Broker Load流程负责与GCS建立连接、提取数据并将数据存储到StarRocks中。

### Broker Load的优势

- 在加载过程中，Broker Load支持数据转换、UPSERT和DELETE操作。
- Broker Load在后台运行，客户端无需保持连接以便任务继续进行。
- Broker Load适用于运行时间较长的任务，默认超时时间为4小时。
- 除了Parquet和ORC文件格式外，Broker Load还支持CSV文件。

### 数据流

![Broker Load的工作流程](../assets/broker_load_how-to-work_en.png)

1. 用户创建一个加载作业。
2. 前端（FE）创建查询计划并将计划分发给后端节点（BE）。
3. 后端（BE）节点从源头获取数据，并将数据加载到StarRocks中。

### 典型示例

创建一个表，启动一个加载进程以从GCS提取Parquet文件，并验证数据加载的进度和成功情况。

> **注意**
>
> 示例中使用的是Parquet格式的样本数据集，如果您想加载CSV或ORC文件，相关信息在本页面底部有链接。

#### 创建表

为您的表创建一个数据库：

```SQL
CREATE DATABASE IF NOT EXISTS project;
USE project;
```

创建一个表。此模式匹配了托管在StarRocks账户中的GCS存储桶中的样本数据集。

```SQL
DROP TABLE IF EXISTS user_behavior;

CREATE TABLE `user_behavior` (
    `UserID` int(11),
    `ItemID` int(11),
    `CategoryID` int(11),
    `BehaviorType` varchar(65533),
    `Timestamp` datetime
) ENGINE=OLAP 
DUPLICATE KEY(`UserID`)
DISTRIBUTED BY HASH(`UserID`)
PROPERTIES (
    "replication_num" = "1"
);
```

> **注意**
>
> 本文档中的示例将`replication_num`属性设置为`1`，以便在简单的单个BE系统上运行。如果您使用三个或更多个BE，则请删除DDL的`PROPERTIES`部分。

#### 启动Broker Load

此作业包含四个主要部分：

- `LABEL`：在查询`LOAD`作业的状态时使用的字符串。
- `LOAD`声明：源URI、目标表和源数据格式。
- `BROKER`：源头的连接细节。
- `PROPERTIES`：超时值和要应用于此作业的其他属性。

> **注意**
>
> 示例中使用的数据集托管在StarRocks账户的GCS存储桶中。由于该对象可被任何经过GCP身份验证的用户读取，任何有效的服务账户电子邮件、秘钥和密钥都可以使用，因此请将下面的命令中的占位符替换为您自己的凭据。

```SQL
LOAD LABEL user_behavior
(
    DATA INFILE("gs://starrocks-samples/user_behavior_ten_million_rows.parquet")
    INTO TABLE user_behavior
    FORMAT AS "parquet"
 )
 WITH BROKER
 (
 
    "gcp.gcs.service_account_email" = "sampledatareader@xxxxx-xxxxxx-000000.iam.gserviceaccount.com",
    "gcp.gcs.service_account_private_key_id" = "baaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
    "gcp.gcs.service_account_private_key" = "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----"
 )
PROPERTIES
(
    "timeout" = "72000"
);
```

#### 检查进度

查询`information_schema.loads`表以跟踪进度。如果您有多个`LOAD`作业正在运行，可以根据作业关联的`LABEL`进行筛选。以下输出显示了加载作业`user_behavior`的两个条目。第一条记录显示`CANCELLED`状态；向输出末尾滚动，您会看到`listPath failed`。第二条记录显示成功，并带有有效的AWS IAM访问密钥和秘钥。

```SQL
SELECT * FROM information_schema.loads;
```

```SQL
SELECT * FROM information_schema.loads WHERE LABEL = 'user_behavior';
```

```plaintext
JOB_ID|LABEL                                      |DATABASE_NAME|STATE    |PROGRESS           |TYPE  |PRIORITY|SCAN_ROWS|FILTERED_ROWS|UNSELECTED_ROWS|SINK_ROWS|ETL_INFO|TASK_INFO                                           |CREATE_TIME        |ETL_START_TIME     |ETL_FINISH_TIME    |LOAD_START_TIME    |LOAD_FINISH_TIME   |JOB_DETAILS                                                                                                                                                                                                                                                    |ERROR_MSG                             |TRACKING_URL|TRACKING_SQL|REJECTED_RECORD_PATH|
------+-------------------------------------------+-------------+---------+-------------------+------+--------+---------+-------------+---------------+---------+--------+----------------------------------------------------+-------------------+-------------------+-------------------+-------------------+-------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------+------------+------------+--------------------+
 10121|user_behavior                              |project      |CANCELLED|ETL:N/A; LOAD:N/A  |BROKER|NORMAL  |        0|            0|              0|        0|        |resource:N/A; timeout(s):72000; max_filter_ratio:0.0|2023-08-10 14:59:30|                   |                   |                   |2023-08-10 14:59:34|{"All backends":{},"FileNumber":0,"FileSize":0,"InternalTableLoadBytes":0,"InternalTableLoadRows":0,"ScanBytes":0,"ScanRows":0,"TaskNumber":0,"Unfinished backends":{}}                                                                                        |type:ETL_RUN_FAIL; msg:listPath failed|            |            |                    |
 10106|user_behavior                              |project      |FINISHED |ETL:100%; LOAD:100%|BROKER|NORMAL  | 86953525|            0|              0| 86953525|        |resource:N/A; timeout(s):72000; max_filter_ratio:0.0|2023-08-10 14:50:15|2023-08-10 14:50:19|2023-08-10 14:50:19|2023-08-10 14:50:19|2023-08-10 14:55:10|{"All backends":{"a5fe5e1d-d7d0-4826-ba99-c7348f9a5f2f":[10004]},"FileNumber":1,"FileSize":1225637388,"InternalTableLoadBytes":2710603082,"InternalTableLoadRows":86953525,"ScanBytes":1225637388,"ScanRows":86953525,"TaskNumber":1,"Unfinished backends":{"a5|                                      |            |            |                    |
```

此时，您也可以检查部分数据。

```SQL
SELECT * from user_behavior LIMIT 10;
```

```plaintext
UserID|ItemID|CategoryID|BehaviorType|Timestamp          |
------+------+----------+------------+-------------------+
171146| 68873|   3002561|pv          |2017-11-30 07:11:14|
171146|146539|   4672807|pv          |2017-11-27 09:51:41|
171146|146539|   4672807|pv          |2017-11-27 14:08:33|
171146|214198|   1320293|pv          |2017-11-25 22:38:27|
171146|260659|   4756105|pv          |2017-11-30 05:11:25|
171146|267617|   4565874|pv          |2017-11-27 14:01:25|
171146|329115|   2858794|pv          |2017-12-01 02:10:51|
171146|458604|   1349561|pv          |2017-11-25 22:49:39|
## 使用`FILES()`表函数

### `FILES()`的优势

`FILES()`可以推断Parquet数据列的数据类型并为StarRocks表生成模式。这提供了直接从S3进行`SELECT`查询或者基于Parquet文件模式让StarRocks自动为您创建表的功能。

> **注意**

>

> 模式推断是version 3.1的新功能，仅为Parquet格式提供，并且尚不支持嵌套类型。

### 典型示例

有三个使用`FILES()`表函数的示例：

- 直接从S3查询数据

- 使用模式推断创建和加载表
- 手动创建表然后加载数据


#### 直接从S3查询

使用`FILES()`直接从S3查询可以在创建表之前很好地预览数据集的内容。例如：

- 获取数据集的预览而无需存储数据。
- 查询最小和最大值，然后决定使用什么数据类型。
- 检查空值。

```sql
SELECT * FROM FILES(
    "path" = "gs://starrocks-samples/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "gcp.gcs.service_account_email" = "sampledatareader@xxxxx-xxxxxx-000000.iam.gserviceaccount.com",
    "gcp.gcs.service_account_private_key_id" = "baaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
    "gcp.gcs.service_account_private_key" = "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----"
) LIMIT 10;
```

> **注意**
>
> 注意列名由Parquet文件提供。

```plaintext
UserID|ItemID |CategoryID|BehaviorType|Timestamp          |
------+-------+----------+------------+-------------------+
     1|2576651|    149192|pv          |2017-11-25 01:21:25|
     1|3830808|   4181361|pv          |2017-11-25 07:04:53|
     1|4365585|   2520377|pv          |2017-11-25 07:49:06|
     1|4606018|   2735466|pv          |2017-11-25 13:28:01|
     1| 230380|    411153|pv          |2017-11-25 21:22:22|
     1|3827899|   2920476|pv          |2017-11-26 16:24:33|
     1|3745169|   2891509|pv          |2017-11-26 19:44:31|
     1|1531036|   2920476|pv          |2017-11-26 22:02:12|
     1|2266567|   4145813|pv          |2017-11-27 00:11:11|
     1|2951368|   1080785|pv          |2017-11-27 02:47:08|
```

#### 使用模式推断创建表

这是前面示例的延续；前面的查询封装在`CREATE TABLE`中，以使用模式推断自动创建表。当使用`FILES()`表函数与Parquet文件时，不需要指定列名和类型来创建表，因为Parquet格式包括列名和类型，而StarRocks会推断模式。

> **注意**
>
> 在使用模式推断时，`CREATE TABLE`的语法不允许设置副本数，因此在创建表之前设置它。下面的示例适用于具有单个副本的系统：
>
> `ADMIN SET FRONTEND CONFIG ('default_replication_num' ="1");`

```sql
CREATE DATABASE IF NOT EXISTS project;
USE project;

CREATE TABLE `user_behavior_inferred` AS
SELECT * FROM FILES(
    "path" = "gs://starrocks-samples/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "gcp.gcs.service_account_email" = "sampledatareader@xxxxx-xxxxxx-000000.iam.gserviceaccount.com",
    "gcp.gcs.service_account_private_key_id" = "baaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
    "gcp.gcs.service_account_private_key" = "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----"
);
```

```SQL
DESCRIBE user_behavior_inferred;
```

```plaintext
Field       |Type            |Null|Key  |Default|Extra|
------------+----------------+----+-----+-------+-----+
UserID      |bigint          |YES |true |       |     |
ItemID      |bigint          |YES |true |       |     |
CategoryID  |bigint          |YES |true |       |     |
BehaviorType|varchar(1048576)|YES |false|       |     |
Timestamp   |varchar(1048576)|YES |false|       |     |
```

> **注意**
>
> 将推断的模式与手动创建的模式进行比较：
>
> - 数据类型
> - 可为空
> - 键字段

```SQL
SELECT * from user_behavior_inferred LIMIT 10;
```

```plaintext
UserID|ItemID|CategoryID|BehaviorType|Timestamp          |
------+------+----------+------------+-------------------+
171146| 68873|   3002561|pv          |2017-11-30 07:11:14|
171146|146539|   4672807|pv          |2017-11-27 09:51:41|
171146|146539|   4672807|pv          |2017-11-27 14:08:33|
171146|214198|   1320293|pv          |2017-11-25 22:38:27|
171146|260659|   4756105|pv          |2017-11-30 05:11:25|
171146|267617|   4565874|pv          |2017-11-27 14:01:25|
171146|329115|   2858794|pv          |2017-12-01 02:10:51|
171146|458604|   1349561|pv          |2017-11-25 22:49:39|
171146|458604|   1349561|pv          |2017-11-27 14:03:44|
171146|478802|    541347|pv          |2017-12-02 04:52:39|
```

#### 加载到现有表中

您可能希望自定义要插入的表，例如：

- 列数据类型、可空设置或默认值
- 键类型和列
- 分布
- 等等

> **注意**
>
> 创建最有效的表结构需要了解数据的使用方式和列的内容。本文档不涵盖表设计，页面底部有**更多信息**的链接。

在这个示例中，我们基于对将要查询表以及Parquet文件中的数据的了解来创建表。

- 由于S3中的文件查询表明`Timestamp`列包含与`datetime`数据类型匹配的数据，因此下面的DDL中指定了列类型。
- 通过查询S3中的数据，您可以发现数据集中没有空值，因此DDL不设置任何列为可空。
- 基于对预期查询类型的了解，将排序键和分桶列设置为列`UserID` (您的用例可能对此数据有不同的决定，您可能决定对排序键使用`ItemID`以及/或者替代`UserID`):

```SQL
CREATE TABLE `user_behavior_declared` (
    `UserID` int(11),
    `ItemID` int(11),
    `CategoryID` int(11),
    `BehaviorType` varchar(65533),
    `Timestamp` datetime
) ENGINE=OLAP 
DUPLICATE KEY(`UserID`)
DISTRIBUTED BY HASH(`UserID`)
PROPERTIES (
    "replication_num" = "1"
);
```

创建表之后，您可以使用`INSERT INTO` … `SELECT FROM FILES()`加载数据：

```SQL
INSERT INTO user_behavior_declared
  SELECT * FROM FILES(
    "path" = "gs://starrocks-samples/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "gcp.gcs.service_account_email" = "sampledatareader@xxxxx-xxxxxx-000000.iam.gserviceaccount.com",
```
    "gcp.gcs.service_account_private_key_id" = "baaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
    "gcp.gcs.service_account_private_key" = "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----"
);
```

## 更多信息

- 此文档仅涵盖了服务账号密钥认证。有关其他选项，请参阅[认证到GCS资源](../integrations/authenticate_to_gcs.md)。
- 有关同步和异步数据加载的详细信息，请参阅[数据加载概述](../loading/Loading_intro.md)文档。
- 了解经纪人加载在加载过程中支持的数据转换，详情请参阅[加载时进行数据转换](../loading/Etl_in_loading.md)和[加载到主键表中进行数据更改](../loading/Load_to_Primary_Key_tables.md)。
- 了解更多关于[表设计](../table_design/StarRocks_table_design.md)的信息。
- 经纪人加载提供的配置和使用选项远远多于上述示例中提供的，详细信息请参阅[经纪人加载](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。