---
displayed_sidebar: English
---

# 从 GCS 加载数据

从 '../assets/commonMarkdown/insertPrivNote.md' 导入 InsertPrivNote

StarRocks 提供了两种从 GCS 加载数据的方式：

1. 使用 Broker Load 进行异步加载
2. 使用 FILES() 表函数进行同步加载

小型数据集通常使用 FILES() 表函数同步加载，而大型数据集通常使用 Broker Load 异步加载。这两种方法各有不同的优势，下面将分别进行介绍。

<InsertPrivNote />


## 收集连接信息

> **注意**
> 示例中使用的是服务账户密钥认证方式。还有其他认证方法可用，具体信息请参见本页面底部的[链接](https://www.example.com)。
> 本指南中使用的是 **StarRocks** 托管的数据集。任何通过 **GCP** 认证的用户都可以读取该数据集，因此您可以使用自己的凭证来读取下文中用到的 **Parquet** 文件。

从 GCS 加载数据需要具备以下信息：

- GCS 存储桶
- GCS 对象键（对象名称）如果要访问存储桶中的特定对象。请注意，如果您的 GCS 对象存储在子文件夹中，对象键可以包含前缀。完整的语法请参见**更多信息**部分的链接。
- GCS 区域
- 服务账户的访问密钥和密钥

## 使用 Broker Load

Broker Load 是一个异步过程，它负责建立与 GCS 的连接、拉取数据以及将数据存储到 StarRocks 中。

### Broker Load 的优势

- Broker Load 在加载过程中支持数据转换、UPSERT 和 DELETE 操作。
- Broker Load 在后台运行，客户端不需要保持连接即可完成作业。
- Broker Load 适合长时间运行的作业，默认超时时间为 4 小时。
- 除了 Parquet 和 ORC 文件格式，Broker Load 还支持 CSV 文件。

### 数据流程

![Workflow of Broker Load](../assets/broker_load_how-to-work_en.png)

1. 用户创建一个加载作业。
2. 前端（FE）创建查询计划并将该计划分发到后端节点（BE）。
3. 后端节点（BE）从源头拉取数据并将数据加载到 StarRocks 中。

### 典型示例

创建一个表，启动一个加载作业来从 GCS 拉取 Parquet 文件，并验证数据加载的进度和成功情况。

> **注意**
> 示例使用的是 Parquet 格式的示例数据集，如果您想要加载 CSV 或 ORC 文件，相关信息请参见本页面底部的链接。

#### 创建表

为您的表创建一个数据库：

```SQL
CREATE DATABASE IF NOT EXISTS project;
USE project;
```

创建一个表。此表结构与 StarRocks 账户托管的 GCS 存储桶中的示例数据集相匹配。

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
> 本文档中的示例将属性 `replication_num` 设置为 `1`，以便可以在单个后端系统上运行。如果您使用的是三个或更多的后端节点，请移除 DDL 中的 `PROPERTIES` 部分。

#### 启动 Broker Load

这项作业包含四个主要部分：

- LABEL：在查询 LOAD 作业状态时使用的字符串标识。
- LOAD 声明：源 URI、目标表和源数据格式。
- BROKER：源的连接详情。
- PROPERTIES：超时值和应用于此作业的其他属性。

> **注意**
> 这些示例中使用的数据集托管在一个StarRocks账户的GCS存储桶中。任何有效的服务账户电子邮件、密钥和密钥都可以使用，因为任何经过GCP认证的用户都可以读取该对象。请将下面命令中的占位符替换为您的凭据。

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

查询 information_schema.loads 表以追踪进度。如果您有多个 LOAD 作业在运行，您可以根据作业的 LABEL 进行过滤。在下面的输出中，有两个名为 user_behavior 的加载作业条目。第一条记录显示状态为 CANCELLED；滚动到输出的末尾，您会看到 listPath 失败。第二条记录显示使用有效的 AWS IAM 访问密钥和密钥成功。

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
171146|458604|   1349561|pv          |2017-11-27 14:03:44|
171146|478802|    541347|pv          |2017-12-02 04:52:39|
```

## 使用 FILES() 表函数

### FILES() 的优点

FILES() 可以推断 Parquet 数据的列数据类型，并为 StarRocks 表生成架构。这使您能够直接使用 SELECT 从 S3 查询文件，或者让 StarRocks 根据 Parquet 文件的架构自动为您创建表。

> **注意**
> 架构推断是一个新的特性，仅在 3.1 版本中提供，仅适用于 Parquet 格式，暂时不支持嵌套类型。

### 典型示例

使用 FILES() 表函数有三个示例：

- 直接从 S3 查询数据
- 使用架构推断创建和加载表
- 手动创建表然后加载数据

#### 直接从 S3 查询

使用 FILES() 直接从 S3 查询可以在创建表之前预览数据集的内容。例如：

- 无需存储数据即可预览数据集。
- 查询最小值和最大值，以决定使用哪些数据类型。
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
> 请注意，列名由 Parquet 文件提供。

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

#### 使用架构推断创建表

这是前一个示例的延续；之前的查询被包含在 CREATE TABLE 中，以使用架构推断自动创建表。当使用 FILES() 表函数与 Parquet 文件一起时，创建表不需要指定列名和类型，因为 Parquet 格式已经包含了列名和类型，StarRocks 会推断出架构。

> **注意**
> 使用架构推断时 `CREATE TABLE` 的语法不允许设置副本数量，请在创建表之前设置。以下示例适用于单副本系统：
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
> 比较推断出的**架构**和手动创建的**架构**：
- 数据类型
- 是否可为空
- 关键字段

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

您可能希望自定义插入的表，例如：

- 列数据类型、是否可为空的设置或默认值
- 键类型和列
- 分布
- 等等。

> **注意**
> 创建最高效的表结构需要了解数据的使用方式和列内容。本文档没有涉及表设计，**更多信息**请参见页面底部的链接。

在本示例中，我们根据对表查询方式和 Parquet 文件中数据的了解创建了一个表。通过直接在 S3 中查询文件，可以了解 Parquet 文件中的数据信息。

- 由于在 S3 中对文件的查询显示 Timestamp 列包含与 datetime 数据类型匹配的数据，因此在下面的 DDL 中指定了列类型。
- 通过在 S3 中查询数据，我们发现数据集中没有空值，因此 DDL 没有将任何列设置为可为空。
- 根据预期的查询类型知识，将排序键和分桶列设置为 UserID 列（您的用例可能会有所不同，您可能决定使用 ItemID 作为排序键的补充或替代 UserID）：

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

创建表后，您可以使用 INSERT INTO ... SELECT FROM FILES() 来加载数据：

```SQL
INSERT INTO user_behavior_declared
  SELECT * FROM FILES(
    "path" = "gs://starrocks-samples/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "gcp.gcs.service_account_email" = "sampledatareader@xxxxx-xxxxxx-000000.iam.gserviceaccount.com",
    "gcp.gcs.service_account_private_key_id" = "baaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
    "gcp.gcs.service_account_private_key" = "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----"
);
```

## 更多信息

- 本文档仅介绍了服务账户密钥认证。有关其他选项，请参阅[验证 GCS 资源](../integrations/authenticate_to_gcs.md)。
- 有关同步和异步数据加载的更多详细信息，请参阅[数据加载概述](../loading/Loading_intro.md)文档。
- 了解 Broker Load 如何在加载期间支持数据转换，请参阅[Transform data at loading](../loading/Etl_in_loading.md)和[Change data through loading](../loading/Load_to_Primary_Key_tables.md)。
- 了解更多关于[table design](../table_design/StarRocks_table_design.md)的信息。
- Broker Load 提供了比上述示例中更多的配置和使用选项，详细信息请参阅 [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 文档。
