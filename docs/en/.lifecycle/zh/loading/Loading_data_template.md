---
displayed_sidebar: English
unlisted: true
---

# 从 \<SOURCE\> 模板加载数据

## 模板说明

### 关于风格的注释

技术文档通常包含指向其他文档的链接。当您查看本文档时，可能会注意到页面中的链接较少，且几乎所有链接都位于文档底部的**更多信息**部分。不是每个关键词都需要链接到另一个页面，请假设读者知道 `CREATE TABLE` 的含义，如果他们不了解，可以点击搜索栏查找。在文档中提示读者有其他选项，详细信息在**更多信息**部分描述；这样，需要该信息的人就知道他们可以在完成当前任务**之后**阅读它。

### 模板

本模板基于从 Amazon S3 加载数据的流程，其中某些部分可能不适用于从其他来源加载。请专注于本模板的流程，不必担心包含每一个部分；流程的目的是：

#### 介绍

介绍性文本，让读者知道如果他们遵循本指南，最终结果将是什么。在 S3 文档的案例中，最终结果是“以异步或同步方式从 S3 加载数据”。

#### 为什么？

- 描述使用该技术解决的业务问题
- 所描述方法的优点和缺点（如果有）

#### 数据流图或其他图形

图表或图像可能有帮助。如果您描述的技术复杂且图像有助于理解，那么请使用图像。如果您描述的技术产生了视觉效果（例如，使用 Superset 分析数据），那么一定要包含最终产品的图像。

如果流程不是显而易见的，请使用数据流图。当一个命令导致 StarRocks 运行多个进程并组合这些进程的输出，然后处理数据时，可能需要一个数据流的描述。在本模板中，描述了两种加载数据的方法。一种是简单的，没有数据流部分；另一种更复杂（StarRocks 处理复杂工作，而不是用户！），复杂的选项包括数据流部分。

#### 示例及验证部分

注意，示例应在语法细节和其他深入技术细节之前给出。许多读者会查阅文档以找到他们可以复制、粘贴和修改的特定技术。

如果可能，请提供一个包含数据集的示例，该示例可以正常工作。本模板中的示例使用存储在 S3 中的数据集，任何拥有 AWS 账户并能够使用密钥和密钥对进行身份验证的人都可以使用。提供数据集使示例对读者更有价值，因为他们可以完整体验所描述的技术。

确保示例按照编写的方式正常工作。这意味着两件事：

1. 您已按照呈现的顺序运行命令
2. 您已包括必要的先决条件。例如，如果您的示例引用了数据库 `foo`，那么您可能需要在前面加上 `CREATE DATABASE foo;`, `USE foo;`。

验证非常重要。如果您描述的过程包括多个步骤，那么在应该完成某事时包括一个验证步骤；这有助于避免读者在完成后才意识到他们在第 10 步中打错了字。在本示例中，“**检查进度**”和 `DESCRIBE user_behavior_inferred;` 步骤用于验证。

#### 更多信息

在模板的末尾，有一个地方可以放置相关信息的链接，包括您在正文中提到的可选信息的链接。

### 模板中嵌入的注释

模板注释的格式故意与我们格式化文档注释的方式不同，以便在您使用模板时引起您的注意。请在进行时删除粗体斜体注释：

```markdown
***注：描述性文本***
```

## 最后，开始模板

***注：如果有多个推荐选择，请在简介中告诉读者。例如，在从 S3 加载时，有同步加载和异步加载的选项：***

StarRocks 提供了两种从 S3 加载数据的选项：

1. 使用 Broker Load 进行异步加载
2. 使用 `FILES()` 表函数进行同步加载

***注：告诉读者为什么他们会选择一种方法而不是另一种：***

小型数据集通常使用 `FILES()` 表函数同步加载，而大型数据集通常使用 Broker Load 异步加载。这两种方法各有不同的优点，下面将进行描述。

> **注意**
> 您只能以对 StarRocks 表具有 INSERT 权限的用户身份将数据加载到这些 StarRocks 表中。如果您没有 INSERT 权限，请按照 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 中提供的说明向用于连接到您的 StarRocks 集群的用户授予 INSERT 权限。

## 使用 Broker Load

异步 Broker Load 过程负责建立与 S3 的连接、拉取数据并将数据存储在 StarRocks 中。

### Broker Load 的优势

- Broker Load 在加载过程中支持数据转换、UPSERT 和 DELETE 操作。
- Broker Load 在后台运行，客户端不需要保持连接即可继续作业。
- Broker Load 适用于长时间运行的作业，默认超时时间为 4 小时。
- 除了 Parquet 和 ORC 文件格式，Broker Load 还支持 CSV 文件。

### 数据流

***注***：涉及多个组件或步骤的流程可能更容易通过图表理解。***此示例包括一个图表，有助于描述用户选择 Broker Load 选项时发生的步骤。***

![Broker Load 工作流程](../assets/broker_load_how-to-work_en.png)

1. 用户创建加载作业。
2. 前端（FE）创建查询计划并将计划分发到后端节点（BE）。
3. 后端（BE）节点从源拉取数据并将数据加载到 StarRocks 中。

### 典型示例

创建一个表，启动一个加载过程，从 S3 拉取 Parquet 文件，并验证数据加载的进度和成功。

> **注意**
> 这些示例使用的是 Parquet 格式的样本数据集，如果您想加载 CSV 或 ORC 文件，相关信息链接在本页底部。

#### 创建表

为您的表创建一个数据库：

```SQL
CREATE DATABASE IF NOT EXISTS project;
USE project;
```

创建一个表。此架构与 StarRocks 账户托管的 S3 存储桶中的样本数据集相匹配。

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

#### 收集连接详情

> **注意**
> 这些示例使用基于 IAM 用户的身份验证。其他身份验证方法可用，并在本页底部链接。

从 S3 加载数据需要有：

- S3 桶
- S3 对象键（如果访问桶中的特定对象）。注意，如果您的 S3 对象存储在子文件夹中，则对象键可以包含前缀。完整语法链接在**更多信息**中。
- S3 区域
- 访问密钥和密钥

#### 启动 Broker Load

这项工作主要包括四个部分：

- `LABEL`：查询 LOAD 作业状态时使用的字符串。
- `LOAD` 声明：源 URI、目标表和源数据格式。
- `BROKER`：源的连接详情。
- `PROPERTIES`：超时值和适用于此作业的任何其他属性。

> **注意**
> 这些示例中使用的数据集托管在 StarRocks 账户的 S3 存储桶中。任何有效的 `aws.s3.access_key` 和 `aws.s3.secret_key` 都可以使用，因为任何经过 AWS 身份验证的用户都可以读取该对象。在下面的命令中，将您的凭证替换为 `AAA` 和 `BBB`。

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

查询 `information_schema.loads` 表以跟踪进度。如果您有多个 `LOAD` 作业正在运行，您可以根据作业的 `LABEL` 进行筛选。在下面的输出中，有两个关于 `user_behavior` 的加载作业条目。第一条记录显示状态为 `CANCELLED`；滚动到输出的末尾，您会看到 `listPath failed`。第二条记录显示使用有效的 AWS IAM 访问密钥和密钥成功。

```SQL
SELECT * FROM information_schema.loads;
```

```SQL
SELECT * FROM information_schema.loads WHERE LABEL = 'user_behavior';
```

```plaintext
JOB_ID|LABEL                                      |DATABASE_NAME|STATE    |PROGRESS           |TYPE  |PRIORITY|SCAN_ROWS|FILTERED_ROWS|UNSELECTED_ROWS|SINK_ROWS|ETL_INFO|TASK_INFO                                           |CREATE_TIME        |ETL_START_TIME     |ETL_FINISH_TIME    |LOAD_START_TIME    |LOAD_FINISH_TIME   |JOB_DETAILS                                                                                                                                                                                                                                                    |ERROR_MSG                             |TRACKING_URL|TRACKING_SQL|REJECTED_RECORD_PATH|
------+-------------------------------------------+-------------+---------+-------------------+------+--------+---------+-------------+---------------+---------+--------+----------------------------------------------------+-------------------+-------------------+-------------------+-------------------+-------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------+------------+------------+--------------------+
```
```plaintext
10121|user_behavior                              |project      |CANCELLED|ETL:N/A; LOAD:N/A  |BROKER|NORMAL  |        0|            0|              0|        0|        |resource:N/A; timeout(s):72000; max_filter_ratio:0.0|2023-08-10 14:59:30|                   |                   |                   |2023-08-10 14:59:34|{"All backends":{},"FileNumber":0,"FileSize":0,"InternalTableLoadBytes":0,"InternalTableLoadRows":0,"ScanBytes":0,"ScanRows":0,"TaskNumber":0,"Unfinished backends":{}}                                                                                        |type:ETL_RUN_FAIL; msg:listPath failed|            |            |                    |
10106|user_behavior                              |project      |FINISHED |ETL:100%; LOAD:100%|BROKER|NORMAL  | 86953525|            0|              0| 86953525|        |resource:N/A; timeout(s):72000; max_filter_ratio:0.0|2023-08-10 14:50:15|2023-08-10 14:50:19|2023-08-10 14:50:19|2023-08-10 14:50:19|2023-08-10 14:55:10|{"All backends":{"a5fe5e1d-d7d0-4826-ba99-c7348f9a5f2f":[10004]},"FileNumber":1,"FileSize":1225637388,"InternalTableLoadBytes":2710603082,"InternalTableLoadRows":86953525,"ScanBytes":1225637388,"ScanRows":86953525,"TaskNumber":1,"Unfinished backends":{"a5|                                      |            |            |                    |
```

此时，您还可以检查数据的子集。

```SQL
SELECT * FROM user_behavior LIMIT 10;
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

`FILES()` 能够推断 Parquet 数据的列数据类型并生成 StarRocks 表的架构。这提供了直接从 S3 使用 `SELECT` 查询文件的能力，或者让 StarRocks 根据 Parquet 文件的架构自动为您创建表。

> **注意**
> 模式推断是在 3.1 版本中的新特性，目前仅支持 Parquet 格式，并且尚不支持嵌套类型。

### 典型示例

这里有三个使用 `FILES()` 表函数的示例：

- 直接从 S3 查询数据
- 使用架构推断创建并加载表
- 手动创建表然后加载数据

> **注意**
> 这些示例中使用的数据集托管在 StarRocks 账户的 S3 桶中。任何有效的 `aws.s3.access_key` 和 `aws.s3.secret_key` 都可以使用，因为任何经过 AWS 认证的用户都可以读取该对象。请在下面的命令中用您的凭证替换 `AAA` 和 `BBB`。

#### 直接从 S3 查询

使用 `FILES()` 直接从 S3 查询可以在创建表之前很好地预览数据集的内容。例如：

- 在不存储数据的情况下预览数据集。
- 查询最小值和最大值以决定使用哪些数据类型。
- 检查是否有空值。

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
> 注意列名是由 Parquet 文件提供的。

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

这是前面示例的延续；前面的查询被包裹在 `CREATE TABLE` 中，以使用架构推断自动创建表。当使用 `FILES()` 表函数与 Parquet 文件一起时，不需要指定列名和类型来创建表，因为 Parquet 格式包含了列名和类型，StarRocks 会推断出架构。

> **注意**
> 使用架构推断时，`CREATE TABLE` 的语法不允许设置副本数量，所以在创建表之前设置它。以下示例适用于单副本系统：
> `ADMIN SET FRONTEND CONFIG ('default_replication_num' = "1");`

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
> 比较推断的架构与手动创建的架构：
- 数据类型
- 是否可为空
- 键字段

```SQL
SELECT * FROM user_behavior_inferred LIMIT 10;
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

您可能希望自定义您要插入的表，例如：

- 列数据类型、是否可为空的设置或默认值
- 键类型和列
- 分布方式
- 等等。

> **注意**
> 创建最高效的表结构需要了解数据的使用方式和列的内容。本文档不涉及表设计，页面末尾的 **更多信息** 中有相关链接。

在这个示例中，我们根据对表查询方式的了解以及 Parquet 文件中的数据知识创建了一个表。可以通过直接在 S3 中查询文件来获得 Parquet 文件中数据的知识。

- 由于 S3 中的文件查询表明 `Timestamp` 列包含与 `datetime` 数据类型匹配的数据，因此在以下 DDL 中指定了列类型。
- 通过查询 S3 中的数据，您可以发现数据集中没有空值，因此 DDL 没有将任何列设置为可为空。
- 根据对预期查询类型的了解，排序键和分桶列设置为 `UserID` 列（您的用例可能会有所不同，您可能决定使用 `ItemID` 作为排序键的补充或替代 `UserID`）：

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

创建表后，您可以使用 `INSERT INTO ... SELECT FROM FILES()` 来加载它：

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

- 有关同步和异步数据加载的更多详细信息，请参阅[数据加载概述](../loading/Loading_intro.md)文档。
- 了解 Broker Load 在加载期间如何支持数据转换，请参阅[加载时数据转换](../loading/Etl_in_loading.md)和[通过加载更改数据](../loading/Load_to_Primary_Key_tables.md)。
- 本文档仅涵盖了基于 IAM 用户的认证。有关其他选项，请参阅[认证到 AWS 资源](../integrations/authenticate_to_aws_resources.md)。
- [AWS CLI 命令参考](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3/index.html)详细介绍了 S3 URI。
- 了解更多关于[表设计](../table_design/StarRocks_table_design.md)的信息。
- Broker Load 提供了比上述示例中更多的配置和使用选项，详细信息请参阅[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。
- 有关同步和异步数据加载的更多详细信息，请参阅[数据加载概述](../loading/Loading_intro.md)文档。
- 了解 Broker Load 在加载期间如何支持数据转换，请参阅[在加载时转换数据](../loading/Etl_in_loading.md)和[通过加载变更数据](../loading/Load_to_Primary_Key_tables.md)。
- 本文档仅涵盖了基于 IAM 用户的身份验证。有关其他选项，请参阅[对 AWS 资源进行身份验证](../integrations/authenticate_to_aws_resources.md)。
- [AWS CLI 命令参考](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3/index.html)详细介绍了 S3 URI。
- 了解更多有关[表设计](../table_design/StarRocks_table_design.md)的信息。