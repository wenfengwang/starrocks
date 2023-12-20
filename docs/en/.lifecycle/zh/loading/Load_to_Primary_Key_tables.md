---
displayed_sidebar: English
---

# 通过加载改变数据

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

[主键表](../table_design/table_types/primary_key_table.md) 提供的 StarRocks 允许您通过运行 [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 或 [Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) 作业来对 StarRocks 表进行数据更改。这些数据更改包括插入、更新和删除。然而，主键表不支持使用 [Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) 或 [INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md) 来更改数据。

StarRocks 还支持部分更新和条件更新。

<InsertPrivNote />


本主题以 CSV 数据为例，描述了如何通过加载来对 StarRocks 表进行数据更改。支持的数据文件格式根据您选择的加载方法而有所不同。

> **注意**
> 对于 CSV 数据，您可以使用长度不超过 50 字节的 UTF-8 字符串作为文本分隔符，例如逗号 (,)、制表符或竖线 (|)。

## 实现

StarRocks 提供的主键表支持 UPSERT 和 DELETE 操作，并且不区分 INSERT 操作和 UPDATE 操作。

当您创建加载作业时，StarRocks 支持在作业创建语句或命令中添加名为 `__op` 的字段。`__op` 字段用于指定您想要执行的操作类型。

> **注意**
> 当您创建表时，您不需要在该表中添加名为 `__op` 的列。

定义 `__op` 字段的方法取决于您选择的加载方法：

- 如果您选择 Stream Load，请使用 `columns` 参数定义 `__op` 字段。

- 如果您选择 Broker Load，请使用 SET 子句定义 `__op` 字段。

- 如果您选择 Routine Load，请使用 `COLUMNS` 参数定义 `__op` 字段。

您可以根据您想要进行的数据更改来决定是否添加 `__op` 字段。如果您不添加 `__op` 字段，操作类型默认为 UPSERT。主要的数据更改场景如下：

- 如果您要加载的数据文件只涉及 UPSERT 操作，您不需要添加 `__op` 字段。

- 如果您要加载的数据文件只涉及 DELETE 操作，您必须添加 `__op` 字段并指定操作类型为 DELETE。

- 如果您要加载的数据文件涉及 UPSERT 和 DELETE 操作，您必须添加 `__op` 字段，并确保数据文件中包含值为 `0` 或 `1` 的列。值为 `0` 表示 UPSERT 操作，值为 `1` 表示 DELETE 操作。

## 使用说明

- 确保您的数据文件中的每一行都有相同数量的列。

- 涉及数据更改的列必须包括主键列。

## 先决条件

### Broker Load

请参阅 [从 HDFS 加载数据](../loading/hdfs_load.md) 或 [从云存储加载数据](../loading/cloud_storage_load.md) 中的“背景信息”部分。

### Routine Load

如果您选择 Routine Load，请确保在您的 Apache Kafka® 集群中创建了主题。假设您已经创建了四个主题：`topic1`、`topic2`、`topic3` 和 `topic4`。

## 基本操作

本节提供了如何通过加载对 StarRocks 表进行数据更改的示例。有关详细语法和参数描述，请参阅 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 和 [CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

### UPSERT

如果您要加载的数据文件只涉及 UPSERT 操作，您不需要添加 `__op` 字段。

> **注意**
> 如果您添加了 `__op` 字段：
- 您可以将操作类型指定为 UPSERT。

- 您可以留空 `__op` 字段，因为操作类型默认为 UPSERT。

#### 数据示例

1. 准备数据文件。

   a. 在您的本地文件系统中创建一个名为 `example1.csv` 的 CSV 文件。该文件由三列组成，依次代表用户 ID、用户名和用户分数。

   ```Plain
   101,Lily,100
   102,Rose,100
   ```

   b. 将 `example1.csv` 的数据发布到您的 Kafka 集群的 `topic1`。

2. 准备 StarRocks 表。

   a. 在您的 StarRocks 数据库 `test_db` 中创建一个名为 `table1` 的主键表。该表由三列组成：`id`、`name` 和 `score`，其中 `id` 是主键。

   ```SQL
   CREATE TABLE `table1`
   (
       `id` int(11) NOT NULL COMMENT "user ID",
       `name` varchar(65533) NOT NULL COMMENT "user name",
       `score` int(11) NOT NULL COMMENT "user score"
   )
   ENGINE=OLAP
   PRIMARY KEY(`id`)
   DISTRIBUTED BY HASH(`id`);
   ```

      > **注意**
      > 从 v2.5.7 版本开始，StarRocks 在创建表或添加分区时可以自动设置桶（BUCKETS）的数量。您不再需要手动设置桶的数量。有关详细信息，请参阅[确定桶的数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   b. 向 `table1` 插入一条记录。

   ```SQL
   INSERT INTO table1 VALUES
       (101, 'Lily', 80);
   ```

#### 加载数据

运行加载作业，将 `example1.csv` 中 `id` 为 `101` 的记录更新到 `table1`，并将 `example1.csv` 中 `id` 为 `102` 的记录插入到 `table1`。

- 运行 Stream Load 作业。

-   如果您不想包含 `__op` 字段，请运行以下命令：

    ```Bash
    curl --location-trusted -u <username>:<password> \
        -H "Expect:100-continue" \
        -H "label:label1" \
        -H "column_separator:," \
        -T example1.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
    ```

-   如果您想包含 `__op` 字段，请运行以下命令：

    ```Bash
    curl --location-trusted -u <username>:<password> \
        -H "Expect:100-continue" \
        -H "label:label2" \
        -H "column_separator:," \
        -H "columns:__op='upsert'" \
        -T example1.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
    ```

- 运行 Broker Load 作业。

-   如果您不想包含 `__op` 字段，请运行以下命令：

    ```SQL
    LOAD LABEL test_db.label1
    (
        data infile("hdfs://<hdfs_host>:<hdfs_port>/example1.csv")
        into table table1
        columns terminated by ","
        format as "csv"
    )
    with broker "broker1";
    ```

-   如果您想包含 `__op` 字段，请运行以下命令：

    ```SQL
    LOAD LABEL test_db.label2
    (
        data infile("hdfs://<hdfs_host>:<hdfs_port>/example1.csv")
        into table table1
        columns terminated by ","
        format as "csv"
        set (__op = 'upsert')
    )
    with broker "broker1";
    ```

- 运行 Routine Load 作业。

-   如果您不想包含 `__op` 字段，请运行以下命令：

    ```SQL
    CREATE ROUTINE LOAD test_db.table1 ON table1
    COLUMNS TERMINATED BY ",",
    COLUMNS (id, name, score)
    PROPERTIES
    (
        "desired_concurrent_number" = "3",
        "max_batch_interval" = "20",
        "max_batch_rows" = "250000",
        "max_error_number" = "1000"
    )
    FROM KAFKA
    (
        "kafka_broker_list" = "<kafka_broker_host>:<kafka_broker_port>",
        "kafka_topic" = "test1",
        "property.kafka_default_offsets" = "OFFSET_BEGINNING"
    );
    ```

-   如果您想包含 `__op` 字段，请运行以下命令：

    ```SQL
    CREATE ROUTINE LOAD test_db.table1 ON table1
    COLUMNS TERMINATED BY ",",
    COLUMNS (id, name, score, __op='upsert')
    PROPERTIES
    (
        "desired_concurrent_number" = "3",
        "max_batch_interval" = "20",
        "max_batch_rows" = "250000",
        "max_error_number" = "1000"
    )
    FROM KAFKA
    (
        "kafka_broker_list" = "<kafka_broker_host>:<kafka_broker_port>",
        "kafka_topic" = "test1",
        "property.kafka_default_offsets" = "OFFSET_BEGINNING"
    );
    ```
2行记录集（0.01秒）
```

如上查询结果所示，example3.csv中id为101的记录已从table3中删除，id为102的记录已更新到table3，id为103的记录已插入到table3。

## 部分更新

自v2.2版本起，StarRocks支持仅更新主键表的指定列。本节以CSV为例，描述如何执行部分更新。

> **注意**
> 当执行部分更新时，如果要更新的行不存在，StarRocks将插入一条新行，并在那些因未插入数据更新而为空的字段中填充默认值。

### 数据示例

1. 准备一个数据文件。

   a. 在您的本地文件系统中创建一个名为`example4.csv`的CSV文件。该文件包含两列，分别代表用户ID和用户名。

   ```Plain
   101,Lily
   102,Rose
   103,Alice
   ```

   b. 将`example4.csv`的数据发布到您的Kafka集群的`topic4`。

2. 准备一个StarRocks表。

   a. 在您的StarRocks数据库`test_db`中创建一个名为`table4`的主键表。该表包含三列：`id`、`name`和`score`，其中`id`是主键。

   ```SQL
   CREATE TABLE `table4`
   (
       `id` int(11) NOT NULL COMMENT "用户ID",
       `name` varchar(65533) NOT NULL COMMENT "用户名",
       `score` int(11) NOT NULL COMMENT "用户分数"
   )
   ENGINE=OLAP
   PRIMARY KEY(`id`)
   DISTRIBUTED BY HASH(`id`);
   ```

      > **注意**
      > 从v2.5.7版本开始，StarRocks在创建表或添加分区时可以自动设置桶（BUCKETS）的数量。您不再需要手动设置桶的数量。有关详细信息，请参阅[确定桶的数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   b. 向`table4`插入一条记录。

   ```SQL
   INSERT INTO table4 VALUES
       (101, 'Tom', 80);
   ```

### 加载数据

运行加载作业，将`example4.csv`中的两列数据更新到`table4`的`id`和`name`列。

- 运行Stream Load作业：

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label7" -H "column_separator:," \
      -H "partial_update:true" \
      -H "columns:id,name" \
      -T example4.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table4/_stream_load
  ```

    > **注意**
    > 如果选择Stream Load，您必须将`partial_update`参数设置为`true`以启用部分更新功能。此外，您必须使用`columns`参数来指定您想要更新的列。

- 运行Broker Load作业：

  ```SQL
  LOAD LABEL test_db.table4
  (
      data infile("hdfs://<hdfs_host>:<hdfs_port>/example4.csv")
      into table table4
      format as "csv"
      (id, name)
  )
  WITH BROKER "broker1"
  PROPERTIES
  (
      "partial_update" = "true"
  );
  ```

    > **注意**
    > 如果选择Broker Load，您必须将`partial_update`参数设置为`true`以启用部分更新功能。此外，您必须使用`column_list`参数来指定您想要更新的列。

- 运行Routine Load作业：

  ```SQL
  CREATE ROUTINE LOAD test_db.table4 ON table4
  COLUMNS (id, name),
  COLUMNS TERMINATED BY ','
  PROPERTIES
  (
      "partial_update" = "true"
  )
  FROM KAFKA
  (
      "kafka_broker_list" ="<kafka_broker_host>:<kafka_broker_port>",
      "kafka_topic" = "test4",
      "property.kafka_default_offsets" ="OFFSET_BEGINNING"
  );
  ```

    > **注意**
    > 如果选择Routine Load，您必须将`partial_update`参数设置为`true`以启用部分更新功能。此外，您必须使用`COLUMNS`参数来指定您想要更新的列。

### 查询数据

加载完成后，查询`table4`的数据以验证加载是否成功：

```SQL
SELECT * FROM table4;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|  102 | Rose  |     0 |
|  101 | Lily  |    80 |
|  103 | Alice |     0 |
+------+-------+-------+
3行记录集（0.01秒）
```

如上查询结果所示，`example4.csv`中id为101的记录已更新到`table4`，id为102和103的记录已插入到`table4`。

## 条件更新

从StarRocks v2.5版本开始，主键表支持条件更新。您可以指定非主键列作为条件，以确定更新是否可以生效。因此，只有当源数据记录中指定列的值大于或等于目标数据记录中的值时，从源数据记录到目标数据记录的更新才会生效。

条件更新功能旨在解决数据错序问题。如果源数据是无序的，您可以使用此功能确保新数据不会被旧数据覆盖。

> **注意**
- 您不能为同一批数据指定不同的列作为更新条件。
- DELETE操作不支持条件更新。
- 在v3.1.3之前的版本中，部分更新和条件更新不能同时使用。从v3.1.3版本开始，StarRocks支持同时使用部分更新和条件更新。

### 数据示例

1. 准备一个数据文件。

   a. 在您的本地文件系统中创建一个名为`example5.csv`的CSV文件。该文件包含三列，分别代表用户ID、版本和用户分数。

   ```Plain
   101,1,100
   102,3,100
   ```

   b. 将`example5.csv`的数据发布到您的Kafka集群的`topic5`。

2. 准备一个StarRocks表。

   a. 在您的StarRocks数据库`test_db`中创建一个名为`table5`的主键表。该表包含三列：`id`、`version`和`score`，其中`id`是主键。

   ```SQL
   CREATE TABLE `table5`
   (
       `id` int(11) NOT NULL COMMENT "用户ID", 
       `version` int NOT NULL COMMENT "版本",
       `score` int(11) NOT NULL COMMENT "用户分数"
   )
   ENGINE=OLAP
   PRIMARY KEY(`id`) DISTRIBUTED BY HASH(`id`);
   ```

      > **注意**
      > 从v2.5.7版本开始，StarRocks在创建表或添加分区时可以自动设置桶（BUCKETS）的数量。您不再需要手动设置桶的数量。有关详细信息，请参阅[确定桶的数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   b. 向`table5`插入记录。

   ```SQL
   INSERT INTO table5 VALUES
       (101, 2, 80),
       (102, 2, 90);
   ```

### 加载数据

运行加载作业，将`example5.csv`中id为101和102的记录分别更新到`table5`，并指定只有当每条记录的`version`值大于或等于它们当前的`version`值时，更新才生效。

- 运行Stream Load作业：

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label10" \
      -H "column_separator:," \
      -H "merge_condition:version" \
      -T example5.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table5/_stream_load
  ```

- 运行Routine Load作业：

  ```SQL
  CREATE ROUTINE LOAD test_db.table5 ON table5
  COLUMNS (id, version, score),
  COLUMNS TERMINATED BY ','
  PROPERTIES
  (
      "merge_condition" = "version"
  )
  FROM KAFKA
  (
      "kafka_broker_list" ="<kafka_broker_host>:<kafka_broker_port>",
      "kafka_topic" = "topic5",
      "property.kafka_default_offsets" ="OFFSET_BEGINNING"
  );
  ```

- 运行Broker Load作业：

  ```SQL
  LOAD LABEL test_db.table5
  ( DATA INFILE ("s3://xxx.csv")
    INTO TABLE table5 COLUMNS TERMINATED BY "," FORMAT AS "CSV"
  )
  WITH BROKER
  PROPERTIES
  (
      "merge_condition" = "version"
  );
  ```

### 查询数据

加载完成后，查询`table5`的数据以验证加载是否成功：

```SQL
SELECT * FROM table5;
+------+------+-------+
| id   | version | score |
+------+------+-------+
|  101 |       2 |   80 |
|  102 |       3 |  100 |
+------+------+-------+
2行记录集（0.02秒）
```

如上查询结果所示，`example5.csv`中id为101的记录未更新到`table5`，而id为102的记录已插入到`table5`。
```markdown
2 rows in set (0.01 sec)
```

如上查询结果所示，`example3.csv`中`id`为`101`的记录已从`table3`中删除，`id`为`102`的记录已更新到`table3`，而`id`为`103`的记录已插入到`table3`中。

## 部分更新

从v2.2开始，StarRocks支持仅更新主键表的指定列。本节以CSV为例介绍如何进行部分更新。

> **注意**
> 当您执行部分更新时，如果要更新的行不存在，StarRocks会插入一个新行，并在因为没有数据更新而为空的字段中填充默认值。

### 数据示例

1. 准备一个数据文件。

   a. 在本地文件系统中创建一个名为`example4.csv`的CSV文件。该文件由两列组成，依次表示用户ID和用户名。

   ```Plain
   101,Lily
   102,Rose
   103,Alice
   ```

   b. 将`example4.csv`的数据发布到Kafka集群的`topic4`。

2. 准备一个StarRocks表。

   a. 在StarRocks数据库`test_db`中创建一个名为`table4`的主键表。该表由三列组成：`id`、`name`和`score`，其中`id`是主键。

   ```SQL
   CREATE TABLE `table4`
   (
       `id` int(11) NOT NULL COMMENT "用户ID",
       `name` varchar(65533) NOT NULL COMMENT "用户名",
       `score` int(11) NOT NULL COMMENT "用户分数"
   )
   ENGINE=OLAP
   PRIMARY KEY(`id`)
   DISTRIBUTED BY HASH(`id`) ON `id`;
   ```

      > **注意**
      > 从v2.5.7开始，StarRocks可以在创建表或添加分区时自动设置存储桶（BUCKETS）的数量。您不再需要手动设置存储桶的数量。有关详细信息，请参阅[确定桶数](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   b. 向`table4`中插入一条记录。

   ```SQL
   INSERT INTO table4 VALUES
       (101, 'Tom', 80);
   ```

### 加载数据

运行加载作业，将`example4.csv`中的数据更新到`table4`的`id`和`name`列。

- 运行Stream Load作业：

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label7" -H "column_separator:," \
      -H "partial_update:true" \
      -H "columns:id,name" \
      -T example4.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table4/_stream_load
  ```

    > **注意**
    > 如果选择Stream Load，您必须将`partial_update`参数设置为`true`以启用部分更新功能。此外，您必须使用`columns`参数来指定您希望更新的列。

- 运行Broker Load作业：

  ```SQL
  LOAD LABEL test_db.label4
  (
      DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/example4.csv")
      INTO TABLE table4
      COLUMNS TERMINATED BY ","
      FORMAT AS "csv"
      (id, name)
  )
  WITH BROKER "broker1"
  PROPERTIES
  (
      "partial_update" = "true"
  );
  ```

    > **注意**
    > 如果选择Broker Load，您必须将`partial_update`参数设置为`true`以启用部分更新功能。此外，您必须使用`column_list`参数来指定您希望更新的列。

- 运行Routine Load作业：

  ```SQL
  CREATE ROUTINE LOAD test_db.label4 ON table4
  COLUMNS (id, name),
  COLUMNS TERMINATED BY ','
  PROPERTIES
  (
      "partial_update" = "true"
  )
  FROM KAFKA
  (
      "kafka_broker_list" = "<kafka_broker_host>:<kafka_broker_port>",
      "kafka_topic" = "test4",
      "property.kafka_default_offsets" = "OFFSET_BEGINNING"
  );
  ```

    > **注意**
    > 如果选择Routine Load，您必须将`partial_update`参数设置为`true`以启用部分更新功能。此外，您必须使用`COLUMNS`参数来指定您希望更新的列。

### 查询数据

加载完成后，查询`table4`的数据以验证加载是否成功：

```SQL
SELECT * FROM table4;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|  102 | Rose  |     0 |
|  101 | Lily  |    80 |
|  103 | Alice |     0 |
+------+-------+-------+
3 rows in set (0.01 sec)
```

如上查询结果所示，`example4.csv`中`id`为`101`的记录已更新到`table4`中，而`id`为`102`和`103`的记录已插入到`table4`中。

## 条件更新

从StarRocks v2.5开始，主键表支持条件更新。您可以指定非主键列作为判断更新是否生效的条件。这样，只有当源数据记录在指定列中的值大于或等于目标数据记录时，从源记录到目标记录的更新才会生效。

条件更新功能旨在解决数据混乱问题。如果源数据是无序的，您可以使用此功能来确保新数据不会被旧数据覆盖。

> **注意**
- 同一批数据不能指定不同的列作为更新条件。
- DELETE操作不支持条件更新。
- 在v3.1.3之前的版本中，部分更新和条件更新不能同时使用。从v3.1.3开始，StarRocks支持同时使用部分更新和条件更新。

### 数据示例

1. 准备一个数据文件。

   a. 在本地文件系统中创建一个名为`example5.csv`的CSV文件。该文件由三列组成，依次表示用户ID、版本和用户分数。

   ```Plain
   101,1,100
   102,3,100
   ```

   b. 将`example5.csv`的数据发布到Kafka集群的`topic5`。

2. 准备一个StarRocks表。

   a. 在StarRocks数据库`test_db`中创建一个名为`table5`的主键表。该表由三列组成：`id`、`version`和`score`，其中`id`是主键。

   ```SQL
   CREATE TABLE `table5`
   (
       `id` int(11) NOT NULL COMMENT "用户ID",
       `version` int NOT NULL COMMENT "版本",
       `score` int(11) NOT NULL COMMENT "用户分数"
   )
   ENGINE=OLAP
   PRIMARY KEY(`id`) DISTRIBUTED BY HASH(`id`) ON `id`;
   ```

      > **注意**
      > 从v2.5.7开始，StarRocks可以在创建表或添加分区时自动设置存储桶（BUCKETS）的数量。您不再需要手动设置存储桶的数量。有关详细信息，请参阅[确定桶数](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   b. 向`table5`中插入记录。

   ```SQL
   INSERT INTO table5 VALUES
       (101, 2, 80),
       (102, 2, 90);
   ```

### 加载数据

运行加载作业，将`example5.csv`中`id`分别为`101`和`102`的记录更新到`table5`中，并指定只有当两条记录的`version`值大于或等于它们当前的`version`值时，更新才生效。

- 运行Stream Load作业：

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label10" \
      -H "column_separator:," \
      -H "merge_condition:version" \
      -T example5.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table5/_stream_load
  ```

- 运行Routine Load作业：

  ```SQL
  CREATE ROUTINE LOAD test_db.label5 ON table5
  COLUMNS (id, version, score),
  COLUMNS TERMINATED BY ','
  PROPERTIES
  (
      "merge_condition" = "version"
  )
  FROM KAFKA
  (
      "kafka_broker_list" = "<kafka_broker_host>:<kafka_broker_port>",
      "kafka_topic" = "topic5",
      "property.kafka_default_offsets" = "OFFSET_BEGINNING"
  );
  ```

- 运行Broker Load作业：

  ```SQL
  LOAD LABEL test_db.label5
  ( DATA INFILE ("s3://xxx.csv")
    INTO TABLE table5
    COLUMNS TERMINATED BY ","
    FORMAT AS "CSV"
  )
  WITH BROKER
  PROPERTIES
  (
      "merge_condition" = "version"
  );
  ```

### 查询数据

加载完成后，查询`table5`的数据以验证加载是否成功：

```SQL
SELECT * FROM table5;
+------+------+-------+
| id   | version | score |
+------+------+-------+
|  101 |       2 |    80 |
|  102 |       3 |   100 |
+------+------+-------+
2 rows in set (0.02 sec)
```

如上查询结果所示，`example5.csv`中`id`为`101`的记录未更新到`table5`，而`id`为`102`的记录已更新到`table5`中。
```
```markdown
      "merge_condition" = "version"
  );
  ```

### 查询数据

加载完成后，查询`table5`的数据，以验证加载是否成功：

```SQL
SELECT * FROM table5;
+------+------+-------+
| id   | version | score |
+------+------+-------+
|  101 |       2 |   80 |
|  102 |       3 |  100 |
+------+------+-------+
2行在集合中 (0.02秒)
```

如上查询结果所示，`example5.csv`中`id`为`101`的记录没有更新到`table5`，而`id`为`102`的记录已经被插入到`table5`中。