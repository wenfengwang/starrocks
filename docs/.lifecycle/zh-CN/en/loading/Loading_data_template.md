---
displayed_sidebar: "Chinese"
unlisted: true
---

# 从\<SOURCE\>模板加载数据

## 模板说明

### 关于样式的说明

技术文档通常在各处都有指向其他文档的链接。当您查看本文档时，您可能会注意到页面上几乎没有链接，而几乎所有链接都在文档底部的**更多信息**部分。不需要将每个关键字都链接到另一页，请假设读者知道`CREATE TABLE`是什么意思，如果不知道可以在搜索栏中找到。在文档中放置一个注释告诉读者还有其他选项，并且详细信息在**更多信息**部分描述，这样需要信息的人就知道他们可以***稍后***阅读，完成手头的任务后。

### 模板

此模板基于从Amazon S3加载数据的过程，其中的某些部分对于从其他来源加载可能不适用。请专注于此模板的流程，不必担心包括每个部分；流程旨在是：


#### 介绍

介绍性文本，让读者知道如果他们按照此指南操作，最终结果将会是什么。在S3文档的情况下，最终结果是“从S3中以异步方式或同步方式加载数据”。

#### 为什么？

- 解决商业问题的描述和方法的优势和劣势（如果有的话）


#### 数据流或其他图表

图表或图片可能会有所帮助。如果您正在描述一个复杂的技术并且图像有助于理解，那么请使用图像。如果您正在描述一个产生可视化结果的技术（例如使用Superset分析数据），那么一定要包含最终产品的图像。

如果流程不明显，请使用数据流程图。当一个命令导致StarRocks运行多个进程并组合这些进程的输出，然后操作数据，这可能是需要描述数据流程的时候。在此模板中，描述了两种加载数据的方法。其中一种简单，没有数据流程部分；另一种更复杂（StarRocks正在处理复杂的工作，而不是用户！），复杂选项包括数据流程部分。

#### 带验证部分的示例

请注意，示例应该放在语法细节和其他深入的技术细节之前。许多读者会查阅文档，寻找他们能够复制、粘贴和修改的特定技术。

如果可能的话，给出一个可以工作并包含要使用的数据集的示例。此模板中的示例使用存储在S3中的数据集，任何拥有AWS账户并且可以使用密钥和秘钥进行验证的人都可以使用。通过提供数据集，示例对于读者更有价值，因为他们可以完全体验所描述的技术。

确保示例按照书面顺序正常工作。这意味着两件事：

1. 您按照给出的顺序运行了命令
2. 您包括了必要的先决条件。例如，如果您的示例涉及数据库`foo`，那么可能需要在前面加上`CREATE DATABASE foo;`，`USE foo;`。

验证非常重要。如果您所描述的流程包括几个步骤，则在某些情况下应包含验证步骤；这有助于避免读者在最后才意识到在第10步中有错误。在此示例中**检查进度**和`DESCRIBE user_behavior_inferred;`步骤是用于验证的。

#### 更多信息

在模板的末尾，有一个放置相关信息的位置，包括主体中提到的可选信息的链接。

### 嵌入模板的注释

模板注释故意使用与我们格式化文档注释不同的格式，以便在您工作时引起注意。请在使用过程中删除粗体斜体注释：

```markdown
***注：描述性文本***
```

## 最后，模板的开始

***注：如果有多个推荐选择，请在介绍中告诉读者这一点。例如，当从S3加载时，存在同步加载和异步加载的选项：***

StarRocks提供两种从S3加载数据的选项：

1. 使用Broker Load进行异步加载

2. 使用`FILES()`表函数进行同步加载


***注：告诉读者为什么选择一个选择而不是另一个选择：***

通常对小数据集使用`FILES()`表函数进行同步加载，对大数据集使用Broker Load进行异步加载。这两种方法有不同的优势，并且以下进行描述。

> **注意**
>
> 您只能以对StarRocks表具有INSERT权限的用户身份加载数据到StarRocks表中。如果您没有INSERT权限，请按照[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)提供的说明授予您连接到StarRocks集群的用户INSERT权限。

## 使用Broker Load

异步Broker Load流程处理与S3的连接，拉取数据并将数据存储在StarRocks中。

### Broker Load的优势

- Broker Load支持数据转换、UPSERT和DELETE操作。
- Broker Load在后台运行，客户端无需保持连接以继续作业。
- Broker Load适用于长时间运行的作业，超时时间默认为4小时。
- 除了Parquet和ORC文件格式外，Broker Load还支持CSV文件。

### 数据流程

***注：涉及多个组件或步骤的流程可能通过图表更容易理解。此示例包括一张图表，以帮助描述用户选择Broker Load选项时发生的步骤。***

![Broker Load的工作流程](../assets/broker_load_how-to-work_zh.png)

1. 用户创建一个加载作业。
2. 前端（FE）创建一个查询计划，并将计划分发到后端节点（BE）。
3. 后端（BE）节点从源中拉取数据，并将数据加载到StarRocks中。

### 典型示例

创建一个表，启动一个拉取S3中Parquet文件的加载进程，并验证数据加载的进度和成功情况。

> **注意**
>
> 示例使用了Parquet格式的样本数据集，如果要加载CSV或ORC文件，则底部链接有相关信息。

#### 创建一个表

为您的表创建一个数据库：

```SQL
CREATE DATABASE IF NOT EXISTS project;
USE project;
```

创建一个表。此架构与托管在StarRocks账户S3存储桶中的样本数据集匹配。

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
> 示例使用IAM用户身份验证。页面底部链接提供了其他身份验证方法的信息。

从S3中加载数据需要以下信息：

- S3存储桶
- 如果访问存储桶中的特定对象，则需要S3对象键（对象名称）。请注意，如果您的S3对象存储在子文件夹中，对象键可以包括前缀。完整的语法在**更多信息**部分有链接。
- S3区域
- 访问密钥和秘钥

#### 启动Broker Load

此作业具有四个主要部分：

- `LABEL`：在查询`LOAD`作业的状态时使用的字符串。
- `LOAD`声明：源URI、目标表和源数据格式。
- `BROKER`：源的连接详细信息。
- `PROPERTIES`：超时值和应用于此作业的任何其他属性。

> **注意**
>
> 示例中使用了一个存储在StarRocks账户S3存储桶中的数据集。以下命令中将`AAA`和`BBB`替换为您的凭据。

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

#### 检查进展

查询`information_schema.loads`表来跟踪进度。如果有多个`LOAD`作业在运行，您可以根据作业相关联的`LABEL`进行筛选。在下面的输出中，对于负载作业`user_behavior`有两个条目。第一条记录显示状态为`CANCELLED`；滚动到输出末尾时，您会看到`listPath failed`。第二条记录显示成功，具有有效的AWS IAM访问密钥和秘钥。

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

您也可以在此时检查数据的一个子集。

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

## 使用`FILES()`表函数

### `FILES()`优势

`FILES()`可以推断Parquet数据列的数据类型，并为StarRocks表生成模式。这提供了直接从S3查询文件的能力，并使用`SELECT`，或者基于Parquet文件模式进行自动创建一个表的能力。

> **注意**
>
> 模式推断是3.1版本的新功能，仅适用于Parquet格式，尚不支持嵌套类型。

### 典型示例

有三个示例使用`FILES()`表函数：

- 直接从S3查询数据
- 使用模式推断创建和加载表
- 手动创建表，然后加载数据

> **注意**
>
> 这些示例中使用的数据集托管在StarRocks帐户的S3存储桶中。任何有效的`aws.s3.access_key`和`aws.s3.secret_key`均可使用，因为该对象可被任何AWS经过身份验证的用户读取。在下面的命令中，将`AAA`和`BBB`的凭据替换为您的凭据。

#### 直接从S3查询

使用`FILES()`直接从S3查询可以在创建表之前很好地预览数据的内容。例如：

- 获取数据集的预览而不存储数据。
- 查询最小和最大值，决定使用什么数据类型。
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
> 注意，列名由Parquet文件提供。

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

这是前面示例的延续，前一个查询被包装在`CREATE TABLE`中，以使用模式推断自动创建表。使用`FILES()`表函数与Parquet文件时，不需要列名和类型即可创建表，因为Parquet格式包含列名和类型，而StarRocks将推断出模式。

> **注意**
>
> 当使用模式推断时，`CREATE TABLE`的语法不允许设置副本数，因此在创建表之前设置它。下面的示例适用于具有单个副本的系统：
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
> 将推断模式与手工创建的模式进行比较：
>
> - 数据类型
> - 可为空性
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

- 列数据类型、可空设置或默认值
- 关键类型和列
- 分布
- 等等

> **注意**
>
> 创建最有效的表结构需要了解数据的使用方式和列的内容。此文档不涵盖表设计，页面底部有“更多信息”链接。

在此示例中，我们创建了一个表，基于对表将如何查询和Parquet文件中的数据的了解。

- 通过查询S3中的文件，发现`Timestamp`列包含与`datetime`数据类型匹配的数据，因此在以下DDL中指定了列类型。
- 通过查询S3中的数据，可以发现数据集中没有空值，因此DDL不将任何列设置为可为空。
- 基于对预期查询类型的了解，排序键和分桶列设置为`UserID`列（对于这些数据，您的用例可能与此不同，您可能决定为排序键使用`ItemID`或者除了`UserID`之外还使用`ItemID`）。

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

创建表后，您可以使用`INSERT INTO`…`SELECT FROM FILES()`将其加载：

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
- 了解经纪加载如何在加载期间支持数据转换，请参阅[加载时进行数据转换](../loading/Etl_in_loading.md)和[加载时更改数据](../loading/Load_to_Primary_Key_tables.md)。
- 本文档仅涵盖了基于IAM用户的身份验证。有关其他选项，请参阅[对AWS资源进行身份验证](../integrations/authenticate_to_aws_resources.md)。
- [AWS CLI命令参考](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3/index.html)详细介绍了S3 URI。
- 了解更多关于[表设计](../table_design/StarRocks_table_design.md)的信息。
- 经纪加载提供了比上述示例中更多的配置和使用选项，详细信息请参阅[经纪加载](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。