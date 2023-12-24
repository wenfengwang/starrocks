---
displayed_sidebar: English
---

# 通过加载更改数据

从 '../table_design/table_types/primary_key_table.md' 导入 InsertPrivNote。

StarRocks 提供的[主键表](../table_design/table_types/primary_key_table.md)允许您通过运行[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)或[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)作业对 StarRocks 表进行数据更改。这些数据更改包括插入、更新和删除。但是，主键表不支持使用[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)或[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)更改数据。

StarRocks 还支持部分更新和条件更新。

<InsertPrivNote />

本主题以 CSV 数据为例，介绍如何通过加载方式对 StarRocks 表进行数据修改。支持的数据文件格式因您选择的加载方法而异。

> **注意**
>
> 对于 CSV 数据，您可以使用长度不超过 50 个字节的 UTF-8 字符串（例如逗号（,）、制表符或竖线（|））作为文本分隔符。

## 实现

StarRocks 提供的主键表支持 UPSERT 和 DELETE 操作，并不区分 INSERT 和 UPDATE 操作。

在创建加载作业时，StarRocks 支持向作业创建语句或命令中添加名为 `__op` 的字段。该 `__op` 字段用于指定要执行的操作类型。

> **注意**
>
> 创建表时，无需向该表添加名为 `__op` 的列。

定义 `__op` 字段的方法因您选择的加载方法而异：

- 如果选择 Stream Load（流加载），请使用 `columns` 参数定义 `__op` 字段。

- 如果选择 Broker Load，请使用 SET 子句定义 `__op` 字段。

- 如果选择 Routine Load（例程加载），请使用 `COLUMNS` 列定义 `__op` 字段。

您可以根据要进行的数据更改来决定是否添加 `__op` 字段。如果不添加该 `__op` 字段，则操作类型默认为 UPSERT。主要数据变更场景如下：

- 如果要加载的数据文件仅涉及 UPSERT 操作，则无需添加 `__op` 字段。

- 如果要加载的数据文件仅涉及 DELETE 操作，则必须添加 `__op` 字段并将操作类型指定为 DELETE。

- 如果要加载的数据文件同时涉及 UPSERT 和 DELETE 操作，则必须添加 `__op` 字段，并确保数据文件包含的列中的值为 `0` 或 `1`。值为 `0` 表示 UPSERT 操作，值为 `1` 表示 DELETE 操作。

## 使用说明

- 确保数据文件中的每一行都具有相同的列数。

- 涉及数据更改的列必须包含主键列。

## 先决条件

### Broker Load

请参阅[从 HDFS 加载数据](../loading/hdfs_load.md)或[从云存储加载数据](../loading/cloud_storage_load.md)中的“背景信息”部分。

### Routine Load

如果选择 Routine Load（例程加载），请确保在 Apache Kafka® 集群中创建主题。假设您已创建了四个主题：`topic1`、`topic2`、`topic3` 和 `topic4`。

## 基本操作

本节提供了如何通过加载方式对 StarRocks 表进行数据更改的示例。有关详细的语法和参数描述，请参阅[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)和[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

### UPSERT

如果要加载的数据文件仅涉及 UPSERT 操作，则无需添加 `__op` 字段。

> **注意**
>
> 如果添加 `__op` 字段：
>
> - 您可以将操作类型指定为 UPSERT。
>
> - 您可以将 `__op` 字段留空，因为操作类型默认为 UPSERT。

#### 数据示例

1. 准备数据文件。

   a. 在本地文件系统中创建名为 `example1.csv` 的 CSV 文件。该文件由三列组成，分别表示用户 ID、用户名和用户分数。

      ```Plain
      101,Lily,100
      102,Rose,100
      ```

   b. 将 `example1.csv` 的数据发布到 Kafka 集群的 `topic1`。

2. 准备一个 StarRocks 表。

   a. 在 StarRocks 数据库 `test_db` 中创建一个名为 `table1` 的主键表。该表由三列组成：`id`、`name` 和 `score`，其中 `id` 是主键。

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
      >
      > 从 v2.5.7 开始，StarRocks 可以在您创建表或添加分区时自动设置存储桶数量（BUCKETS）。您不再需要手动设置存储桶数量。有关详细信息，请参阅[确定存储桶数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   b. 向 `table1` 插入一条记录。

      ```SQL
      INSERT INTO table1 VALUES
          (101, 'Lily',80);
      ```

#### 加载数据

运行加载作业，将 `example1.csv` 中 `id` 为 `101` 的记录更新到 `table1` 中，并将 `example1.csv` 中 `id` 为 `102` 的记录插入到 `table1` 中。

- 运行流加载作业。
  
  - 如果不想包含 `__op` 字段，请运行以下命令：

    ```Bash
    curl --location-trusted -u <username>:<password> \
        -H "Expect:100-continue" \
        -H "label:label1" \
        -H "column_separator:," \
        -T example1.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
    ```

  - 如果要包含 `__op` 字段，请运行以下命令：

    ```Bash
    curl --location-trusted -u <username>:<password> \
        -H "Expect:100-continue" \
        -H "label:label2" \
        -H "column_separator:," \
        -H "columns:__op ='upsert'" \
        -T example1.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
    ```

- 运行 Broker Load 作业。

  - 如果不想包含 `__op` 字段，请运行以下命令：

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

  - 如果要包含 `__op` 字段，请运行以下命令：

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

- 运行例行加载作业。

  - 如果不想包含 `__op` 字段，请运行以下命令：

    ```SQL
    CREATE ROUTINE LOAD test_db.table1 ON table1
    COLUMNS TERMINATED BY ",",
    COLUMNS (id, name, score)
    PROPERTIES
    (
        "desired_concurrent_number" = "3",
        "max_batch_interval" = "20",
        "max_batch_rows"= "250000",
        "max_error_number" = "1000"
    )
    FROM KAFKA
    (
        "kafka_broker_list" ="<kafka_broker_host>:<kafka_broker_port>",
        "kafka_topic" = "test1",
        "property.kafka_default_offsets" ="OFFSET_BEGINNING"
    );
    ```

  - 如果要包含 `__op` 字段，请运行以下命令：

    ```SQL
    CREATE ROUTINE LOAD test_db.table1 ON table1
    COLUMNS TERMINATED BY ",",
    COLUMNS (id, name, score, __op ='upsert')
    PROPERTIES
    (
        "desired_concurrent_number" = "3",
        "max_batch_interval" = "20",
        "max_batch_rows"= "250000",
        "max_error_number" = "1000"
    )
    FROM KAFKA
    (

        "kafka_broker_list" ="<kafka_broker_host>:<kafka_broker_port>",
        "kafka_topic" = "test1",
        "property.kafka_default_offsets" ="OFFSET_BEGINNING"
    );
    ```

#### 查询数据

加载完成后，请查询 `table1` 的数据，以验证加载是否成功：

```SQL
SELECT * FROM table1;
+------+------+-------+
| id   | name | score |
+------+------+-------+
|  101 | Lily |   100 |
|  102 | Rose |   100 |
+------+------+-------+
2 rows in set (0.02 sec)
```

如上述查询结果所示，`example1.csv` 中 `id` 为 `101` 的记录已更新到 `table1`，而 `example1.csv` 中 `id` 为 `102` 的记录已插入到 `table1`。

### 删除

如果要加载的数据文件仅涉及 DELETE 操作，则必须添加 `__op` 字段，并指定操作类型为 DELETE。

#### 数据示例

1. 准备数据文件。

   a. 在本地文件系统中创建名为 `example2.csv` 的 CSV 文件。该文件由三列组成，依次表示用户 ID、用户名和用户分数。

      ```Plain
      101,Jack,100
      ```

   b. 将 `example2.csv` 的数据发布到 Kafka 集群的 `topic2`。

2. 准备 StarRocks 表。

   a. 在 StarRocks 表 `test_db` 中创建一个名为 `table2` 的主键表。该表由三列组成：`id`、`name` 和 `score`，其中 `id` 是主键。

      ```SQL
      CREATE TABLE `table2`
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
      >
      > 从 v2.5.7 版本开始，StarRocks 在创建表或添加分区时可以自动设置存储桶（BUCKETS）的数量。您无需再手动设置存储桶的数量。有关详细信息，请参见 [确定存储桶数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   b. 向 `table2` 中插入两条记录。

      ```SQL
      INSERT INTO table2 VALUES
      (101, 'Jack', 100),
      (102, 'Bob', 90);
      ```

#### 加载数据

运行加载作业，从 `table2` 中删除 `example2.csv` 中 `id` 为 `101` 的记录。

- 运行流加载作业。

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label3" \
      -H "column_separator:," \
      -H "columns:__op='delete'" \
      -T example2.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table2/_stream_load
  ```

- 运行 Broker 加载作业。

  ```SQL
  LOAD LABEL test_db.label3
  (
      data infile("hdfs://<hdfs_host>:<hdfs_port>/example2.csv")
      into table table2
      columns terminated by ","
      format as "csv"
      set (__op = 'delete')
  )
  with broker "broker1";  
  ```

- 运行常规加载作业。

  ```SQL
  CREATE ROUTINE LOAD test_db.table2 ON table2
  COLUMNS(id, name, score, __op = 'delete')
  PROPERTIES
  (
      "desired_concurrent_number" = "3",
      "max_batch_interval" = "20",
      "max_batch_rows"= "250000",
      "max_error_number" = "1000"
  )
  FROM KAFKA
  (
      "kafka_broker_list" ="<kafka_broker_host>:<kafka_broker_port>",
      "kafka_topic" = "test2",
      "property.kafka_default_offsets" ="OFFSET_BEGINNING"
  );
  ```

#### 查询数据

加载完成后，请查询 `table2` 的数据，以验证加载是否成功：

```SQL
SELECT * FROM table2;
+------+------+-------+
| id   | name | score |
+------+------+-------+
|  102 | Bob  |    90 |
+------+------+-------+
1 row in set (0.00 sec)
```

如上述查询结果所示，`example2.csv` 中 `id` 为 `101` 的记录已从 `table2` 中删除。

### UPSERT 和 DELETE

如果要加载的数据文件同时涉及 UPSERT 和 DELETE 操作，则必须添加 `__op` 字段，并确保数据文件包含一列，其值为 `0` 或 `1`。值为 `0` 表示 UPSERT 操作，值为 `1` 表示 DELETE 操作。

#### 数据示例

1. 准备数据文件。

   a. 在本地文件系统中创建名为 `example3.csv` 的 CSV 文件。该文件由四列组成，依次表示用户 ID、用户名、用户分数和操作类型。

      ```Plain
      101,Tom,100,1
      102,Sam,70,0
      103,Stan,80,0
      ```

   b. 将 `example3.csv` 的数据发布到 Kafka 集群的 `topic3`。

2. 准备 StarRocks 表。

   a. 在 StarRocks 数据库 `test_db` 中创建一个名为 `table3` 的主键表。该表由三列组成：`id`、`name` 和 `score`，其中 `id` 是主键。

      ```SQL
      CREATE TABLE `table3`
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
      >
      > 从 v2.5.7 版本开始，StarRocks 在创建表或添加分区时可以自动设置存储桶（BUCKETS）的数量。您无需再手动设置存储桶的数量。有关详细信息，请参见 [确定存储桶数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   b. 向 `table3` 中插入两条记录。

      ```SQL
      INSERT INTO table3 VALUES
          (101, 'Tom', 100),
          (102, 'Sam', 90);
      ```

#### 加载数据

运行加载作业，从 `table3` 中删除 `example3.csv` 中 `id` 为 `101` 的记录，更新 `example3.csv` 中 `id` 为 `102` 的记录到 `table3`，并将 `example3.csv` 中 `id` 为 `103` 的记录插入到 `table3`。

- 运行流加载作业：

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label4" \
      -H "column_separator:," \
      -H "columns: id, name, score, temp, __op = temp" \
      -T example3.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table3/_stream_load
  ```

  > **注意**
  >
  > 在上述示例中，`example3.csv` 中表示操作类型的第四列被临时命名为 `temp`，并使用 `columns` 参数将 `__op` 字段映射到 `temp` 列。因此，StarRocks 可以根据 `example3.csv` 中第四列的值为 `0` 或 `1` 来决定执行 UPSERT 还是 DELETE 操作。

- 运行 Broker 加载作业：

  ```Bash
  LOAD LABEL test_db.label4
  (
      data infile("hdfs://<hdfs_host>:<hdfs_port>/example1.csv")
      into table table1
      columns terminated by ","
      format as "csv"
      (id, name, score, temp)
      set (__op=temp)
  )
  with broker "broker1";
  ```

- 运行常规加载作业：

  ```SQL
  CREATE ROUTINE LOAD test_db.table3 ON table3
  COLUMNS(id, name, score, temp, __op = temp)
  PROPERTIES
  (
      "desired_concurrent_number" = "3",
      "max_batch_interval" = "20",
      "max_batch_rows"= "250000",
      "max_error_number" = "1000"
  )
  FROM KAFKA
  (
      "kafka_broker_list" = "<kafka_broker_host>:<kafka_broker_port>",
      "kafka_topic" = "test3",
      "property.kafka_default_offsets" = "OFFSET_BEGINNING"
  );
  ```

#### 查询数据

加载完成后，请查询 `table3` 的数据，以验证加载是否成功：

```SQL
SELECT * FROM table3;
+------+------+-------+
| id   | name | score |
+------+------+-------+
|  102 | Sam  |    70 |
|  103 | Stan |    80 |
+------+------+-------+
2 rows in set (0.01 sec)
如前面的查询结果所示，在`example3.csv`中`id`为`101`的记录已从`table3`中删除，`id`为`102`的记录已从`example3.csv`更新到`table3`，而`id`为`103`的记录已从`example3.csv`插入到`table3`中。

## 部分更新

从v2.2开始，StarRocks支持仅更新主键表的指定列。本节以CSV为例，介绍如何执行部分更新。

> **注意**
>
> 当执行部分更新时，如果要更新的行不存在，StarRocks会插入新行，并在未插入数据更新的空字段中填充默认值。

### 数据示例

1. 准备数据文件。

   a. 在本地文件系统中创建名为`example4.csv`的CSV文件。该文件由两列组成，依次表示用户ID和用户名。

      ```Plain
      101,Lily
      102,Rose
      103,Alice
      ```

   b. 将`example4.csv`的数据发布到Kafka集群的`topic4`中。

2. 准备一个StarRocks表。

   a. 在StarRocks数据库`test_db`中创建一个名为`table4`的主键表。该表由三列组成：`id`、`name`和`score`，其中`id`是主键。

      ```SQL
      CREATE TABLE `table4`
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
      >
      > 从v2.5.7开始，StarRocks在创建表或添加分区时可以自动设置存储桶（BUCKETS）的数量。您不再需要手动设置存储桶的数量。有关详细信息，请参见[确定存储桶数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   b. 向`table4`中插入一条记录。

      ```SQL
      INSERT INTO table4 VALUES
          (101, 'Tom',80);
      ```

### 加载数据

运行加载，将`example4.csv`中的两列数据更新到`table4`的`id`和`name`列中。

- 运行流加载作业：

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
  >
  > 如果选择流加载，您必须将`partial_update`参数设置为`true`以启用部分更新功能。此外，您还必须使用`columns`参数指定要更新的列。

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
  >
  > 如果选择Broker Load，您必须将`partial_update`参数设置为`true`以启用部分更新功能。此外，您还必须使用`column_list`参数指定要更新的列。

- 运行例行加载作业：

  ```SQL
  CREATE ROUTINE LOAD test_db.table4 on table4
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
  >
  > 如果选择Broker Load，您必须将`partial_update`参数设置为`true`以启用部分更新功能。此外，您还必须使用`COLUMNS`参数指定要更新的列。

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

如上述查询结果所示，`example4.csv`中`id`为`101`的记录已更新到`table4`，而`example4.csv`中`id`为`102`和`103`的记录已插入到`table4`。

## 有条件的更新

从StarRocks v2.5开始，主键表支持有条件的更新。您可以指定非主键列作为条件，以确定更新是否生效。因此，仅当源数据记录在指定列中的值大于或等于目标数据记录中的值时，从源记录到目标记录的更新才会生效。

条件更新功能旨在解决数据混乱问题。如果源数据无序，可以使用此功能来确保新数据不会被旧数据覆盖。

> **注意**
>
> - 不能为同一批数据指定不同的列作为更新条件。
> - DELETE操作不支持有条件的更新。
> - 在v3.1.3之前的版本中，无法同时使用部分更新和有条件的更新。从v3.1.3开始，StarRocks支持同时使用部分更新和有条件的更新。

### 数据示例

1. 准备数据文件。

   a.a. 在本地文件系统中创建名为`example5.csv`的CSV文件。该文件由三列组成，依次表示用户ID、版本和用户分数。

      ```Plain
      101,1,100
      102,3,100
      ```

   b. 将`example5.csv`的数据发布到Kafka集群的`topic5`中。

2. 准备一个StarRocks表。

   a. 在StarRocks数据库`test_db`中创建一个名为`table5`的主键表。该表由三列组成：`id`、`version`和`score`，其中`id`是主键。

      ```SQL
      CREATE TABLE `table5`
      (
          `id` int(11) NOT NULL COMMENT "user ID", 
          `version` int NOT NULL COMMENT "version",
          `score` int(11) NOT NULL COMMENT "user score"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`) DISTRIBUTED BY HASH(`id`);
      ```

      > **注意**
      >
      > 从v2.5.7开始，StarRocks在创建表或添加分区时可以自动设置存储桶（BUCKETS）的数量。您不再需要手动设置存储桶的数量。有关详细信息，请参见[确定存储桶数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   b. 向`table5`中插入记录。

      ```SQL
      INSERT INTO table5 VALUES
          (101, 2, 80),
          (102, 2, 90);
      ```

### 加载数据

运行加载，将`example5.csv`中`id`值分别为`101`和`102`的记录更新到`table5`中，并指定仅当两条记录中的`version`值大于或等于其当前值时，更新才会生效。

- 运行流加载作业：

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label10" \
      -H "column_separator:," \
      -H "merge_condition:version" \
      -T example5.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table5/_stream_load
  ```

- 运行例行加载作业：

  ```SQL
  CREATE ROUTINE LOAD test_db.table5 on table5
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

加载完成后，请查询 `table5` 的数据，以验证加载是否成功：

```SQL
SELECT * FROM table5;
+------+------+-------+
| id   | version | score |
+------+------+-------+
|  101 |       2 |   80 |
|  102 |       3 |  100 |
+------+------+-------+
2 行记录 (0.02 秒)
```

如上查询结果所示，`example5.csv` 中 `id` 为 `101` 的记录未更新到 `table5`，而 `example5.csv` 中 `id` 为 `102` 的记录已插入到 `table5`。