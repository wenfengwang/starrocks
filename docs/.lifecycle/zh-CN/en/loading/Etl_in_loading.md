---
displayed_sidebar: "中文"
---

# 在加载时转换数据

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocks支持在加载时对数据进行转换。

此功能支持[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)和[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)，但不支持[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)。

<InsertPrivNote />

本主题以CSV数据为例，描述了如何在加载时提取和转换数据。根据您选择的加载方法，支持的数据文件格式有所不同。

> **注意**
>
> 对于CSV数据，您可以使用UTF-8字符串（例如逗号（,）、制表符或竖线（|）），其长度不超过50个字节，作为文本分隔符。

## 场景

将数据文件加载到StarRocks表中时，数据文件的数据可能无法完全映射到StarRocks表的数据上。在这种情况下，您无需在将其加载到StarRocks表之前提取或转换数据。StarRocks可以帮助您在加载期间提取和转换数据：

- 跳过不需要加载的列。
  
  您可以跳过不需要加载的列。此外，如果数据文件的列顺序与StarRocks表的列顺序不同，您可以在数据文件和StarRocks表之间创建列映射。

- 筛选掉不想加载的行。
  
  您可以根据筛选条件指定StarRocks筛选掉不想加载的行。

- 从原始列生成新列。
  
  生成的列是从数据文件的原始列计算出来的特殊列。您可以将生成的列映射到StarRocks表的列上。

- 从文件路径提取分区字段值。
  
  如果数据文件是由Apache Hive™生成的，您可以从文件路径中提取分区字段值。

## 先决条件

### Broker Load

请参见[从HDFS加载数据](../loading/hdfs_load.md)或[从云存储加载数据](../loading/cloud_storage_load.md)中的“背景信息”部分。

### Routine Load

如果选择[Routine Load](./RoutineLoad.md)，请确保在Apache Kafka®集群中已创建主题。假设您已创建了两个主题：`topic1`和`topic2`。

## 数据示例

1. 在本地文件系统中创建数据文件。

   a. 创建名为`file1.csv`的数据文件。文件包括四列，依次表示用户ID、用户性别、事件日期和事件类型。

      ```Plain
      354,female,2020-05-20,1
      465,male,2020-05-21,2
      576,female,2020-05-22,1
      687,male,2020-05-23,2
      ```

   b. 创建名为`file2.csv`的数据文件。文件仅包括一列，表示日期。

      ```Plain
      2020-05-20
      2020-05-21
      2020-05-22
      2020-05-23
      ```

2. 在StarRocks数据库`test_db`中创建表。

   > **注意**
   >
   > 从v2.5.7开始，StarRocks在创建表或添加分区时可以自动设置桶数（BUCKETS）。您无需手动设置桶数。有关详细信息，请参见[确定桶数](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   a. 创建名为`table1`的表，包括三列：`event_date`、`event_type`和`user_id`。

      ```SQL
      MySQL [test_db]> CREATE TABLE table1
      (
          `event_date` DATE COMMENT "事件日期",
          `event_type` TINYINT COMMENT "事件类型",
          `user_id` BIGINT COMMENT "用户ID"
      )
      DISTRIBUTED BY HASH(user_id);
      ```

   b. 创建名为`table2`的表，包括四列：`date`、`year`、`month`和`day`。

      ```SQL
      MySQL [test_db]> CREATE TABLE table2
      (
          `date` DATE COMMENT "日期",
          `year` INT COMMENT "年",
          `month` TINYINT COMMENT "月",
          `day` TINYINT COMMENT "日"
      )
      DISTRIBUTED BY HASH(date);
      ```

3. 将`file1.csv`和`file2.csv`上传到您的HDFS集群的`/user/starrocks/data/input/`路径，将`file1.csv`的数据发布到Kafka集群的`topic1`，并将`file2.csv`的数据发布到Kafka集群的`topic2`。

## 跳过不需要加载的列

要将数据文件加载到StarRocks表中，可能需要跳过一些无法映射到StarRocks表的列。在这种情况下，StarRocks支持仅加载可以从数据文件映射到StarRocks表的列。

此功能支持从以下数据源加载数据：

- 本地文件系统

- HDFS和云存储
  
  > **注意**
  >
  > 本部分以HDFS为例。

- Kafka

在大多数情况下，CSV文件的列未命名。对于某些CSV文件，第一行由列名称组成，但StarRocks会将第一行的内容处理为普通数据而不是列名称。因此，当加载CSV文件时，您必须在作业创建语句或命令中**按顺序**临时命名CSV文件的列。这些临时命名的列会**按名称**映射到StarRocks表的列。请注意以下关于数据文件列的几点：

- 可以映射到StarRocks表并且通过StarRocks表列名称临时命名的列的数据会直接加载。

- 无法映射到StarRocks表的列会被忽略，这些列的数据不会加载。

- 如果某些列可以映射到StarRocks表的列但在作业创建语句或命令中没有被临时命名，加载作业将报告错误。

本部分以`file1.csv`和`table1`为例。`file1.csv`的四列按顺序临时命名为`user_id`、`user_gender`、`event_date`和`event_type`。在`file1.csv`的临时命名列中，`user_id`、`event_date`和`event_type`可以映射到`table1`的特定列，而`user_gender`无法映射到`table1`的任何列。因此，`user_id`、`event_date`和`event_type`加载到`table1`，而`user_gender`不会被加载。

### 加载数据

#### 从本地文件系统加载数据

如果`file1.csv`存储在本地文件系统中，请运行以下命令创建[Stream Load](../loading/StreamLoad.md)作业：

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "column_separator:," \
    -H "columns: user_id, user_gender, event_date, event_type" \
    -T file1.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
```

> **注意**
>
> 如果选择Stream Load，您必须使用`columns`参数按名称临时命名数据文件的列，以在数据文件和StarRocks表之间创建列映射。

有关详细语法和参数描述，请参见[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。

#### 从HDFS集群加载数据

如果`file1.csv`存储在您的HDFS集群中，请执行以下语句创建[Broker Load](../loading/hdfs_load.md)作业：

```SQL
LOAD LABEL test_db.label1
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/file1.csv")
    INTO TABLE `table1`
    FORMAT AS "csv"
    COLUMNS TERMINATED BY ","
    (user_id, user_gender, event_date, event_type)
)
WITH BROKER "broker1";
```

> **注意**
>
> 如果选择Broker Load，您必须使用`column_list`参数按名称临时命名数据文件的列，以在数据文件和StarRocks表之间创建列映射。

有关详细语法和参数描述，请参见[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

#### 从Kafka集群加载数据

如果将 `file1.csv` 的数据发布到您的 Kafka 集群的 `topic1`，请执行以下语句创建 [例行加载](../loading/RoutineLoad.md) 作业：

```SQL
CREATE ROUTINE LOAD test_db.table101 ON table1
    COLUMNS TERMINATED BY ",",
    COLUMNS(user_id, user_gender, event_date, event_type)
FROM KAFKA
(
    "kafka_broker_list" = "<kafka_broker_host>:<kafka_broker_port>",
    "kafka_topic" = "topic1",
    "property.kafka_default_offsets" = "OFFSET_BEGINNING"
);
```

> **注意**
>
> 如果选择例行加载，则必须使用 `COLUMNS` 参数来暂时命名数据文件的列，以在数据文件和 StarRocks 表之间创建列映射。

有关详细的语法和参数描述，请参见 [CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

### 查询数据

在来自您的本地文件系统、HDFS 集群或 Kafka 集群的数据加载完成后，请查询 `table1` 的数据，以验证加载是否成功：

```SQL
MySQL [test_db]> SELECT * FROM table1;
+------------+------------+---------+
| event_date | event_type | user_id |
+------------+------------+---------+
| 2020-05-22 |          1 |     576 |
| 2020-05-20 |          1 |     354 |
| 2020-05-21 |          2 |     465 |
| 2020-05-23 |          2 |     687 |
+------------+------------+---------+
4 rows in set (0.01 sec)
```

## 过滤掉不需要加载的行

当您将数据文件加载到 StarRocks 表时，可能不希望加载数据文件的特定行。在这种情况下，您可以使用 WHERE 子句指定要加载的行。StarRocks 将过滤掉所有不符合 WHERE 子句中指定的筛选条件的行。

此功能支持从以下数据源加载数据：

- 本地文件系统

- HDFS 和云存储
  > **注意**
  >
  > 本节以 HDFS 为例。

- Kafka

本节以 `file1.csv` 和 `table1` 为例。如果要仅从 `file1.csv` 加载事件类型为 `1` 的行到 `table1`，您可以使用 WHERE 子句指定筛选条件 `event_type = 1`。

### 加载数据

#### 从本地文件系统加载数据

如果 `file1.csv` 存储在您的本地文件系统中，请运行以下命令创建 [流加载](../loading/StreamLoad.md) 作业：

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "column_separator:," \
    -H "columns: user_id, user_gender, event_date, event_type" \
    -H "where: event_type=1" \
    -T file1.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
```

有关详细的语法和参数描述，请参见 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。

#### 从 HDFS 集群加载数据

如果 `file1.csv` 存储在您的 HDFS 集群中，请执行以下语句创建 [Broker Load](../loading/hdfs_load.md) 作业：

```SQL
LOAD LABEL test_db.label2
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/file1.csv")
    INTO TABLE `table1`
    FORMAT AS "csv"
    COLUMNS TERMINATED BY ","
    (user_id, user_gender, event_date, event_type)
    WHERE event_type = 1
)
WITH BROKER "broker1";
```

有关详细的语法和参数描述，请参见 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

#### 从 Kafka 集群加载数据

如果将 `file1.csv` 的数据发布到您的 Kafka 集群的 `topic1`，请执行以下语句创建 [例行加载](../loading/RoutineLoad.md) 作业：

```SQL
CREATE ROUTINE LOAD test_db.table102 ON table1
COLUMNS TERMINATED BY ",",
COLUMNS (user_id, user_gender, event_date, event_type)
WHERE event_type = 1
FROM KAFKA
(
    "kafka_broker_list" = "<kafka_broker_host>:<kafka_broker_port>",
    "kafka_topic" = "topic1",
    "property.kafka_default_offsets" = "OFFSET_BEGINNING"
);
```

有关详细的语法和参数描述，请参见 [CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

### 查询数据

在来自您的本地文件系统、HDFS 集群或 Kafka 集群的数据加载完成后，请查询 `table1` 的数据，以验证加载是否成功：

```SQL
MySQL [test_db]> SELECT * FROM table1;
+------------+------------+---------+
| event_date | event_type | user_id |
+------------+------------+---------+
| 2020-05-20 |          1 |     354 |
| 2020-05-22 |          1 |     576 |
+------------+------------+---------+
2 rows in set (0.01 sec)
```

## 从原始列生成新列

当您将数据文件加载到 StarRocks 表时，数据文件的一些数据可能需要在加载到 StarRocks 表之前进行转换。在这种情况下，可以在作业创建命令或语句中使用函数或表达式来实现数据转换。

此功能支持从以下数据源加载数据：

- 本地文件系统

- HDFS 和云存储
  > **注意**
  >
  > 本节以 HDFS 为例。

- Kafka

本节以 `file2.csv` 和 `table2` 为例。`file2.csv` 由表示日期的一个列组成。您可以使用 [year](../sql-reference/sql-functions/date-time-functions/year.md)、[month](../sql-reference/sql-functions/date-time-functions/month.md) 和 [day](../sql-reference/sql-functions/date-time-functions/day.md) 函数从 `file2.csv` 中提取每个日期中的年、月和日，并将提取的数据加载到 `table2` 的 `year`、`month` 和 `day` 列中。 

### 加载数据

#### 从本地文件系统加载数据

如果 `file2.csv` 存储在您的本地文件系统中，请运行以下命令创建 [流加载](../loading/StreamLoad.md) 作业：

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "column_separator:," \
    -H "columns:date,year=year(date),month=month(date),day=day(date)" \
    -T file2.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table2/_stream_load
```

> **注意**
>
> - 在 `columns` 参数中，您必须首先临时命名**数据文件的所有列**，然后临时命名您希望从数据文件的原始列生成的新列。如前面的示例所示，`file2.csv` 的唯一列被临时命名为 `date`，然后调用 `year=year(date)`、`month=month(date)` 和 `day=day(date)` 函数生成三个新列，它们被临时命名为 `year`、`month` 和 `day`。
>
> - Stream Load 不支持 `column_name = function(column_name)` 形式，但支持 `column_name= function(column_name)` 形式。

有关详细的语法和参数描述，请参见 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。

#### 从 HDFS 集群加载数据

如果 `file2.csv` 存储在您的 HDFS 集群中，请执行以下语句创建 [Broker Load](../loading/hdfs_load.md) 作业：

```SQL
LOAD LABEL test_db.label3
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/file2.csv")
    INTO TABLE `table2`
    FORMAT AS "csv"
    COLUMNS TERMINATED BY ","
    (date)
    SET(year=year(date), month=month(date), day=day(date))
)
WITH BROKER "broker1";
```

> **注意**
> 必须首先使用`column_list`参数临时命名数据文件的**所有列**，然后使用SET子句临时命名您想要从数据文件的原始列生成的新列。如前面的示例所示，`file2.csv`的唯一列在`column_list`参数中临时命名为`date`，然后在SET子句中调用`year=year(date)`、`month=month(date)`和`day=day(date)`函数来生成三个新列，这些新列在此时被临时命名为`year`、`month`和`day`。

有关详细的语法和参数描述，请参见[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

#### 从Kafka集群加载数据

如果`file2.csv`的数据发布到您的Kafka集群的`topic2`，请执行以下语句来创建[例行加载](../loading/RoutineLoad.md)作业：

```SQL
CREATE ROUTINE LOAD test_db.table201 ON table2
    COLUMNS TERMINATED BY ",",
    COLUMNS(date,year=year(date),month=month(date),day=day(date))
FROM KAFKA
(
    "kafka_broker_list" = "<kafka_broker_host>:<kafka_broker_port>",
    "kafka_topic" = "topic2",
    "property.kafka_default_offsets" = "OFFSET_BEGINNING"
);
```

> **注意**
>
> 在`COLUMNS`参数中，必须首先临时命名数据文件的**所有列**，然后临时命名您想要从数据文件的原始列生成的新列。如前面的示例所示，`file2.csv`的唯一列临时命名为`date`，然后调用`year=year(date)`、`month=month(date)`和`day=day(date)`函数来生成三个新列，这些新列在此时被临时命名为`year`、`month`和`day`。

有关详细的语法和参数描述，请参见[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

### 查询数据

在从您的本地文件系统、HDFS集群或Kafka集群加载数据完成后，查询`table2`的数据以验证加载是否成功：

```SQL
MySQL [test_db]> SELECT * FROM table2;
+------------+------+-------+------+
| date       | year | month | day  |
+------------+------+-------+------+
| 2020-05-20 | 2020 |  5    | 20   |
| 2020-05-21 | 2020 |  5    | 21   |
| 2020-05-22 | 2020 |  5    | 22   |
| 2020-05-23 | 2020 |  5    | 23   |
+------------+------+-------+------+
4 rows in set (0.01 sec)
```

## 从文件路径提取分区字段值

如果您指定的文件路径包含分区字段，可以使用`文件路径中的列作为`参数指定您想要从文件路径中提取的分区字段。文件路径中的分区字段等效于数据文件中的列。`文件路径中的列作为`参数仅在从HDFS集群加载数据时受支持。

例如，假设您要加载从Hive生成的以下四个数据文件：

```Plain
/user/starrocks/data/input/date=2020-05-20/data
1,354
/user/starrocks/data/input/date=2020-05-21/data
2,465
/user/starrocks/data/input/date=2020-05-22/data
1,576
/user/starrocks/data/input/date=2020-05-23/data
2,687
```

这四个数据文件存储在您的HDFS集群的`/user/starrocks/data/input/`路径中。每个数据文件都是按照分区字段`date`进行分区的，并且由两列组成，分别表示事件类型和用户ID。

### 从HDFS集群加载数据

执行以下语句创建[Broker Load](../loading/hdfs_load.md)作业，从`/user/starrocks/data/input/`文件路径提取`date`分区字段值，并使用通配符(*)指定要加载到`table1`的文件路径中的所有数据文件：

```SQL
LOAD LABEL test_db.label4
(
    DATA INFILE("hdfs://<fe_host>:<fe_http_port>/user/starrocks/data/input/date=*/*")
    INTO TABLE `table1`
    FORMAT AS "csv"
    COLUMNS TERMINATED BY ","
    (event_type, user_id)
    COLUMNS FROM PATH AS (date)
    SET(event_date = date)
)
WITH BROKER "broker1";
```

> **注意**
>
> 在前面的示例中，指定文件路径中的`date`分区字段等效于`table1`中的`event_date`列。因此，您需要使用SET子句将文件路径中的`date`分区字段映射到`event_date`列。如果指定文件路径中的分区字段与StarRocks表的某个列具有相同的名称，则不需要使用SET子句创建映射。

有关详细的语法和参数描述，请参见[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

### 查询数据

在从您的HDFS集群加载数据完成后，查询`table1`的数据以验证加载是否成功：

```SQL
MySQL [test_db]> SELECT * FROM table1;
+------------+------------+---------+
| event_date | event_type | user_id |
+------------+------------+---------+
| 2020-05-22 |          1 |     576 |
| 2020-05-20 |          1 |     354 |
| 2020-05-21 |          2 |     465 |
| 2020-05-23 |          2 |     687 |
+------------+------------+---------+
4 rows in set (0.01 sec)
```