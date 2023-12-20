---
displayed_sidebar: English
unlisted: true
---

# 从\<SOURCE\>模板加载数据

## 模板说明

### 关于风格的说明

技术文档通常包含大量指向其他文档的链接。当您查看本文档时，您可能会注意到，页面中的链接不多，且几乎所有的链接都位于文档底部的**更多信息**部分。不是每个关键词都需要链接到另一个页面，请假设读者知道`CREATE TABLE`的含义，如果他们不了解，可以点击搜索栏查找。在文档中添加注释，告诉读者还有其他选项，并在**更多信息**部分中详细描述，这样需要这些信息的人就知道，他们可以在完成当前任务后再去阅读***稍后***。

### 模板

此模板是基于从Amazon S3加载数据的流程而设计的，其中一些部分可能不适用于从其他来源加载。请关注模板的流程，并不必担心包含每一个部分；流程应该是这样的：

#### 引言

介绍性文字，让读者明白如果他们遵循本指南，最终将得到什么结果。在S3文档的案例中，最终结果是“以异步或同步的方式从S3加载数据”。

#### 为什么？

- 描述使用该技术解决的商业问题
- 所描述方法的优势和劣势（如果有）

#### 数据流程图或其他图形

图表或图像可能有帮助。如果您描述的技术复杂且图像有助于理解，那么就使用图像。如果您描述的技术会产生某种视觉效果（例如，使用Superset分析数据），那么肯定要包含最终产品的图像。

如果流程不太明显，请使用数据流程图。当一个命令导致StarRocks运行多个进程，结合这些进程的输出，然后处理数据时，可能需要一个数据流程的描述。在这个模板中，描述了两种加载数据的方法。一种是简单的，没有数据流程部分；另一种更复杂（是StarRocks在处理复杂工作，而不是用户！），复杂的选项包括数据流程部分。

#### 带验证部分的示例

请注意，示例应该在语法细节和其他深入的技术细节之前提供。很多读者会查找文档，寻找他们可以复制、粘贴和修改的特定技术。

如果可能，请提供一个包含数据集的工作示例。本模板中的示例使用的数据集存储在S3中，任何拥有AWS账户并可以通过密钥和密钥秘密进行身份验证的用户都可以使用。通过提供数据集，示例对读者更有价值，因为他们可以完全体验所描述的技术。

确保示例按照编写的内容工作。这意味着两件事：

1. 您已按照呈现的顺序运行了命令
2. 您已包括了必要的先决条件。例如，如果示例提到了数据库foo，那么您可能需要先执行CREATE DATABASE foo; 和 USE foo;。

验证非常重要。如果您描述的过程包含多个步骤，那么每当应该完成某项操作时，就包含一个验证步骤；这有助于避免读者在完成后意识到他们在第10步中打错了字。在这个示例中 **检查进度** 和 `DESCRIBE user_behavior_inferred;` 步骤是为了验证。

#### 更多信息

在模板的末尾，有一个位置用于放置相关信息的链接，包括在正文中提到的可选信息的链接。

### 模板中嵌入的注释

模板注释的格式故意与我们格式化文档注释的方式不同，目的是在您使用模板时引起您的注意。请在进行时删除粗体斜体的注释：

```markdown
***Note: descriptive text***
```

## 最后，开始模板

***注意***：如果有多个推荐选项，请在引言中告诉读者。例如，当从S3加载时，有同步加载和异步加载的选项：

StarRocks提供了两种从S3加载数据的方式：

1. 使用Broker Load进行异步加载
2. 使用FILES()表函数进行同步加载

***注意: 告诉读者为什么他们会选择一种方法而不是另一种:***

小型数据集通常使用FILES()表函数进行同步加载，而大型数据集通常使用Broker Load进行异步加载。这两种方法各有不同的优点，下面将分别介绍。

> **注意**
> 您只能以拥有**INSERT**权限的用户身份将数据加载到StarRocks表中。如果您没有**INSERT**权限，请按照[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)中提供的指导将**INSERT**权限授予用于连接到您的StarRocks集群的用户。

## 使用代理加载

异步Broker Load过程负责建立与S3的连接、提取数据并将数据存储在StarRocks中。

### Broker Load的优势

- Broker Load在加载过程中支持数据转换、UPSERT和DELETE操作。
- Broker Load在后台运行，客户端无需保持连接即可继续作业。
- Broker Load适用于长时间运行的作业，默认超时时间为4小时。
- 除了Parquet和ORC文件格式，Broker Load还支持CSV文件。

### 数据流

***注意: 涉及多个组件或步骤的过程可能更容易通过图表理解。这个示例包括一个图表，帮助描述用户选择Broker Load选项时发生的步骤。***

![Workflow of Broker Load](../assets/broker_load_how-to-work_en.png)

1. 用户创建一个加载作业。
2. 前端（FE）创建查询计划并将计划分发给后端节点（BE）。
3. 后端（BE）节点从源头提取数据并将数据加载到StarRocks中。

### 典型示例

创建一个表，启动一个加载过程，从S3拉取Parquet文件，并验证数据加载的进度和成功情况。

> **注意**
> 这些示例使用Parquet格式的样本数据集，如果您想加载CSV或ORC文件，该信息在本页底部有链接。

#### 创建一个表

为您的表创建一个数据库：

```SQL
CREATE DATABASE IF NOT EXISTS project;
USE project;
```

创建一个表。此架构与StarRocks账户托管的S3存储桶中的样本数据集相匹配。

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
> 这些示例使用基于IAM用户的认证。其他认证方法可用，并在本页底部有链接。

从S3加载数据需要有：

- S3存储桶
- S3对象键（对象名称），如果访问存储桶中的特定对象。请注意，如果您的S3对象存储在子文件夹中，对象键可以包含前缀。完整语法在**更多信息**中有链接。
- S3区域
- 访问密钥和密钥秘密

#### 启动Broker Load

这项工作有四个主要部分：

- LABEL：查询LOAD作业状态时使用的字符串。
- LOAD声明：源URI、目标表和源数据格式。
- BROKER：源的连接详情。
- PROPERTIES：超时值和适用于此作业的任何其他属性。

> **注意**
> 这些示例中使用的数据集托管在一个StarRocks账户的S3存储桶中。任何有效的`aws.s3.access_key`和`aws.s3.secret_key`都可以使用，因为任何经过AWS认证的用户都可以读取该对象。在下面的命令中用您的凭据替换`AAA`和`BBB`。

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

查询information_schema.loads表以跟踪进度。如果您有多个LOAD作业在运行，您可以过滤与作业相关的LABEL。在下面的输出中，有两条针对load job user_behavior的记录。第一条记录显示状态为CANCELLED；滚动到输出的末尾，您会看到listPath失败。第二条记录显示使用有效的AWS IAM访问密钥和秘密成功。

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

您也可以在这个时候检查数据的一个子集。

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

## 使用FILES()表函数

### FILES()的优点

FILES()可以推断Parquet数据的列类型，并为StarRocks表生成架构。这提供了直接从S3使用SELECT查询文件的能力，或让StarRocks根据Parquet文件的架构自动为您创建表。

> **注意**
> 架构推断是新功能3.1版本的新功能，仅针对Parquet格式提供，目前还不支持嵌套类型。

### 典型示例

有三个使用FILES()表函数的示例：

- 直接从S3查询数据
- 使用架构推断创建和加载表
- 手动创建表然后加载数据

> **注意**
> 这些示例中使用的数据集托管在一个StarRocks账户的S3存储桶中。任何有效的`aws.s3.access_key`和`aws.s3.secret_key`都可以使用，因为任何经过AWS认证的用户都可以读取该对象。在下面的命令中用您的凭据替换`AAA`和`BBB`。

#### 直接从S3查询

使用FILES()直接从S3查询可以在创建表之前很好地预览数据集内容。例如：

- 无需存储数据即可预览数据集。
- 查询最小值和最大值，并决定使用何种数据类型。
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
> 请注意，列名由Parquet文件提供。

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

这是之前示例的延续；之前的查询被包含在CREATE TABLE中，以使用架构推断自动创建表。当使用FILES()表函数与Parquet文件一起使用时，创建表不需要指定列名和类型，因为Parquet格式已包含列名和类型，StarRocks将推断出架构。

> **注意**
> 使用架构推断时`CREATE TABLE`的语法不允许设置副本数量，因此在创建表之前设置。以下示例适用于单副本系统：
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
> 比较推断出的架构和手动创建的架构：
- 数据类型
- 是否可为空
- 键字段

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

您可能想要自定义插入的表，例如：

- 列数据类型、是否可为空的设置或默认值
- 键类型和列
- 分布
- 等等。

> **注意**
> 创建最有效的表结构需要了解数据的使用方式和列内容。本文档不涵盖表设计，页面末尾的**更多信息**链接中有相关内容。

在此示例中，我们根据查询表的方式和Parquet文件中的数据知识创建了一个表。通过直接在S3中查询文件，可以获取Parquet文件中的数据知识。

- 由于S3文件的查询显示Timestamp列包含与datetime数据类型匹配的数据，因此在下面的DDL中指定了列类型。
- 通过在S3中查询数据，发现数据集中没有空值，因此DDL没有将任何列设置为可为空。
- 根据对预期查询类型的了解，将排序键和分桶列设置为UserID列（您的用例可能不同，您可能决定使用ItemID作为排序键，或者UserID和ItemID一起使用）：

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

创建表后，您可以使用INSERT INTO ... SELECT FROM FILES()来加载数据：

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
- 了解Broker Load在加载期间如何支持数据转换，请参阅[在加载时转换数据](../loading/Etl_in_loading.md)和[通过加载更改数据](../loading/Load_to_Primary_Key_tables.md)。
- 本文档仅介绍了基于IAM用户的认证。有关其他选项，请参阅[认证AWS资源](../integrations/authenticate_to_aws_resources.md)。
- [AWS CLI命令参考](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3/index.html)详细介绍了S3 URI。
- 了解更多关于[table design](../table_design/StarRocks_table_design.md)的信息。
- Broker Load提供了比上述示例更多的配置和使用选项，详细信息请参阅[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。
