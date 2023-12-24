---
displayed_sidebar: English
unlisted: true
---

# 从 \<SOURCE\> 模板加载数据

## 模板说明

### 关于风格的说明

技术文档通常到处都有指向其他文档的链接。当您查看此文档时，您可能会注意到页面中的链接很少，并且几乎所有链接都位于文档底部的 **更多信息** 部分。并非每个关键字都需要链接到另一个页面，请假设读者知道 `CREATE TABLE` 意味着什么，如果他们不知道，他们可以点击搜索栏并找到答案。在文档中弹出一个注释，告诉读者还有其他选项，详细信息在 **更多信息** 部分有描述是可以的；这让需要信息的人知道他们可以在***完成***手头的任务后阅读它。

### 模板

此模板基于从 Amazon S3 加载数据的过程，其中部分内容不适用于从其他来源加载。请专注于此模板的流程，不要担心包含每个部分；该流程应为：

#### 介绍

介绍性文本，让读者知道如果他们遵循本指南，最终结果会是什么。对于 S3 文档，最终结果是“以异步方式或同步方式从 S3 加载数据”。

#### 为什么？

- 描述使用该技术解决的业务问题
- 所描述方法的优点和缺点（如果有）

#### 数据流或其他关系图

图表或图像可能会有所帮助。如果您描述的技术很复杂，并且图像有帮助，请使用一种技术。如果您描述的是产生视觉效果的技术（例如，使用超集来分析数据），那么一定要包括最终产品的图像。

如果流不明显，请使用数据流图。当一个命令导致 StarRocks 运行多个进程并合并这些进程的输出，然后操作数据时，可能是时候描述数据流了。在此模板中，描述了两种加载数据的方法。其中一个很简单，没有数据流部分；另一个更复杂（StarRocks 处理的是复杂的工作，而不是用户！），复杂选项包括数据流部分。

#### 带有验证部分的示例

请注意，示例应出现在语法细节和其他深层技术细节之前。许多读者会来到文档中寻找他们可以复制、粘贴和修改的特定技术。

如果可能，请给出一个可行的示例，并包含要使用的数据集。此模板中的示例使用存储在 S3 中的数据集，拥有 AWS 账户并可以使用密钥和密钥进行身份验证的任何人都可以使用该数据集。通过提供数据集，这些示例对读者更有价值，因为他们可以充分体验所描述的技术。

确保示例按书面形式工作。这意味着两件事：

1. 您已按显示的顺序运行命令
2. 您已包含必要的先决条件。例如，如果你的示例引用了数据库 `foo`，那么你可能需要在它前面加上 `CREATE DATABASE foo;`， `USE foo;`。

验证是如此重要。如果您描述的过程包括几个步骤，那么只要应该完成某件事，就包括一个验证步骤；这有助于避免读者读到最后并意识到他们在第 10 步中有错别字。在此示例中，**检查进度** 和 `DESCRIBE user_behavior_inferred;` 步骤用于验证。

#### 更多信息

在模板的末尾，有一个位置可以放置指向相关信息的链接，包括指向您在正文中提到的可选信息的链接。

### 模板中嵌入的注释

模板注释的格式有意与我们格式化文档注释的方式不同，以便在您使用模板时引起您的注意。请在进行时删除粗斜体注释：

```markdown
***Note: descriptive text***
```

## 最后，模板的开始

***注意：如果有多个推荐选项，请在介绍中告诉读者这一点。例如，从 S3 加载时，有一个同步加载和异步加载选项：***

StarRocks 提供了两种从 S3 加载数据的选项：

1. 使用 Broker Load 进行异步加载
2. 使用 `FILES()` 表函数进行同步加载 

***注意：告诉读者为什么他们会选择一个选项而不是另一个选项：***

小型数据集通常使用 `FILES()` 表函数进行同步加载，大型数据集通常使用 Broker Load 进行异步加载。这两种方法具有不同的优点，如下所述。

> **注意**
>
> 您只能以对 StarRocks 表具有 INSERT 权限的用户身份将数据加载到 StarRocks 表中。如果您没有 INSERT 权限，请按照 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 中的说明，将 INSERT 权限授予您用于连接到 StarRocks 集群的用户。

## 使用 Broker Load

异步 Broker Load 进程负责与 S3 建立连接、拉取数据以及将数据存储在 StarRocks 中。

### Broker Load 的优势

- Broker Load 支持加载期间的数据转换、UPSERT 和 DELETE 操作。
- Broker Load 在后台运行，客户端无需保持连接即可继续作业。
- 对于长时间运行的作业，Broker Load 是首选，默认超时为 4 小时。
- 除了 Parquet 和 ORC 文件格式外，Broker Load 还支持 CSV 文件。

### 数据流

***注意：使用图表可能更容易理解涉及多个组件或步骤的流程。此示例包括一个关系图，该关系图有助于描述用户选择 Broker Load 选项时发生的步骤。***

![Broker Load 的工作流](../assets/broker_load_how-to-work_en.png)

1. 用户创建加载作业。
2. 前端（FE）创建查询计划，并将计划分发给后端节点（BE）。
3. 后端节点从源端拉取数据，并将数据加载到 StarRocks 中。

### 典型案例

创建一个表，启动从 S3 拉取 Parquet 文件的加载过程，并验证数据加载的进度和成功。

> **注意**
>
> 这些示例使用 Parquet 格式的示例数据集，如果要加载 CSV 或 ORC 文件，则该信息链接在本页底部。

#### 创建表

为表创建数据库：

```SQL
CREATE DATABASE IF NOT EXISTS project;
USE project;
```

创建表。该架构与 StarRocks 账号托管的 S3 存储桶中的示例数据集匹配。

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

#### 收集连接详细信息

> **注意**
>
> 这些示例使用基于 IAM 用户的身份验证。其他身份验证方法可用，并在本页底部提供链接。

从 S3 加载数据需要具备以下条件：

- S3 存储桶
- S3 对象键（对象名称），如果访问存储桶中的特定对象。请注意，如果您的 S3 对象存储在子文件夹中，则对象键可以包含前缀。完整语法在详细信息中链接****。
- S3 区域
- 访问密钥和密钥

#### 启动 Broker Load

这项工作有四个主要部分：

- `LABEL`：查询作业状态时使用的字符串 `LOAD` 。
- `LOAD` declaration：源 URI、目标表和源数据格式。
- `BROKER`：源的连接详细信息。
- `PROPERTIES`：超时值和要应用于此作业的任何其他属性。

> **注意**
>
> 这些示例中使用的数据集托管在 StarRocks 账号的 S3 存储桶中。任何有效 `aws.s3.access_key` 且 `aws.s3.secret_key` 可以使用，因为任何经过 AWS 身份验证的用户都可以读取该对象。在以下命令中，将 `AAA` 和 `BBB` 替换为您的凭据。

```SQL
LOAD LABEL user_behavior
(
    DATA INFILE("s3://starrocks-datasets/user_behavior_sample_data.parquet")
    INTO TABLE user_behavior
    FORMAT AS "parquet"
 )
 WITH BROKER
 (
    "aws.s3.enable_ssl" = "true",
    "aws.s3.use_instance_profile" = "false",
    "aws.s3.region" = "us-east-1",
    "aws.s3.access_key" = "AAAAAAAAAAAAAAAAAAAA",
    "aws.s3.secret_key" = "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB"
 )
PROPERTIES
(
    "timeout" = "72000"
);
```

#### 检查进度

查询 `information_schema.loads` 表以跟踪进度。如果正在运行多个作业，可以筛选 `LOAD` 与该作业关联的 `LABEL`。在下面的输出中，有两个用于加载作业的条目 `user_behavior`。第一条记录显示的状态为 `CANCELLED`；滚动到输出的末尾，您会看到 `listPath failed`。第二条记录显示使用有效的 AWS IAM 访问密钥和密钥成功。

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

此时，您还可以检查数据的子集。

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

## 使用 `FILES()` 表函数

### `FILES()` 的优势

`FILES()` 可以推断 Parquet 数据列的数据类型，并为 StarRocks 表生成模式。这提供了直接从 S3 使用 `SELECT` 查询文件的能力，或者根据 Parquet 文件模式让 StarRocks 自动为您创建表。

> **注意**
>
> 模式推断是 3.1 版本中的新功能，仅适用于 Parquet 格式，尚不支持嵌套类型。

### 典型示例

使用 `FILES()` 表函数有三个示例：

- 直接从 S3 查询数据
- 使用模式推断创建和加载表
- 手动创建表，然后加载数据

> **注意**
>
> 这些示例中使用的数据集托管在 StarRocks 账户的 S3 存储桶中。任何有效的 `aws.s3.access_key` 和 `aws.s3.secret_key` 都可以使用，因为任何经过 AWS 认证的用户都可以读取该对象。在以下命令中，将 `AAA` 和 `BBB` 替换为您的凭据。

#### 直接从 S3 查询

使用 `FILES()` 直接从 S3 查询可以很好地预览数据集的内容。例如：

- 在不存储数据的情况下预览数据集。
- 查询最小值和最大值，并决定要使用的数据类型。
- 检查空值。

```sql
SELECT * FROM FILES(
    "path" = "s3://starrocks-datasets/user_behavior_sample_data.parquet",
    "format" = "parquet",
    "aws.s3.region" = "us-east-1",
    "aws.s3.access_key" = "AAAAAAAAAAAAAAAAAAAA",
    "aws.s3.secret_key" = "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB"
) LIMIT 10;
```

> **注意**
>
> 请注意，列名称由 Parquet 文件提供。

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

这是上一个示例的延续；前一个查询被包装在 `CREATE TABLE` 中，以使用模式推断自动创建表。使用 Parquet 文件作为格式时，创建表不需要列名和类型，因为 Parquet 格式包含列名和类型，StarRocks 将推断模式。

> **注意**
>
> 使用模式推断时，`CREATE TABLE` 的语法不允许设置副本数，因此请在创建表之前设置。以下示例适用于具有单个副本的系统：
>
> `ADMIN SET FRONTEND CONFIG ('default_replication_num' ="1");`

```sql
CREATE DATABASE IF NOT EXISTS project;
USE project;

CREATE TABLE `user_behavior_inferred` AS
SELECT * FROM FILES(
    "path" = "s3://starrocks-datasets/user_behavior_sample_data.parquet",
    "format" = "parquet",
    "aws.s3.region" = "us-east-1",
    "aws.s3.access_key" = "AAAAAAAAAAAAAAAAAAAA",
    "aws.s3.secret_key" = "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB"
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
> - 关键字段

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

- 列数据类型、可为 null 的设置或默认值
- 键类型和列
- 分配
- 等。

> **注意**
>
> 创建最有效的表结构需要了解如何使用数据以及列的内容。本文档不涉及表格设计， ** 页面末尾**有一个更多信息的链接。

在此示例中，我们将基于有关如何查询表和 Parquet 文件中的数据的知识创建一个表。通过直接在 S3 中查询文件，可以获得 Parquet 文件中数据的知识。

- 由于在 S3 中查询文件指示`Timestamp`该列包含与数据类型匹配的数据`datetime`，因此在以下 DDL 中指定了列类型。
- 通过查询 S3 中的数据，您可以发现数据集中没有空值，因此 DDL 不会将任何列设置为可为 null。
- 根据对预期查询类型的了解，排序键和存储列将设置为该列 `UserID` （此数据的用例可能不同，您可以决定使用排序键`ItemID`的补充或代替 `UserID` 排序键：

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

创建表后，您可以使用 `INSERT INTO` ... `SELECT FROM FILES()`：

```SQL
INSERT INTO user_behavior_declared
  SELECT * FROM FILES(
    "path" = "s3://starrocks-datasets/user_behavior_sample_data.parquet",
    "format" = "parquet",
    "aws.s3.region" = "us-east-1",
    "aws.s3.access_key" = "AAAAAAAAAAAAAAAAAAAA",
    "aws.s3.secret_key" = "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB"
);
```

## 更多信息

- 有关同步和异步数据加载的更多详细信息，请参阅数据加载文档[概述](../loading/Loading_intro.md)。
- 了解 Broker Load 在加载过程中如何支持数据转换，请参阅[加载时的数据转换](../loading/Etl_in_loading.md)和[加载后更改数据](../loading/Load_to_Primary_Key_tables.md)。
- 本文档仅涵盖基于 IAM 用户的身份验证。有关其他选项，请参阅[身份验证到 AWS 资源](../integrations/authenticate_to_aws_resources.md)。
- [AWS CLI 命令参考](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3/index.html)详细介绍了 S3 URI。
- 了解更多关于[表格设计](../table_design/StarRocks_table_design.md)的信息。
- Broker Load 提供了比上述示例更多的配置和使用选项，详细内容请参见[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)