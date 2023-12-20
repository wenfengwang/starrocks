---
displayed_sidebar: English
---

# 通过加载更改数据

从'../assets/commonMarkdown/insertPrivNote.md'导入InsertPrivNote

StarRocks提供的[主键表](../table_design/table_types/primary_key_table.md)允许您通过执行[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)或[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)作业来更改StarRocks表中的数据。这些数据更改包括插入、更新和删除操作。但是，主键表不支持使用[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)或[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)语句来更改数据。

StarRocks还支持部分更新和条件更新。

<InsertPrivNote />


本主题以CSV数据为例，描述了如何通过加载来更改StarRocks表中的数据。支持的数据文件格式取决于您选择的加载方法。

> **注意**
> 对于CSV数据，您可以使用不超过50字节的UTF-8字符串作为文本分隔符，例如逗号（,）、制表符或竖线（|）。

## 实施

StarRocks提供的主键表支持UPSERT和DELETE操作，并且不区分INSERT操作和UPDATE操作。

当您创建加载作业时，StarRocks支持在作业创建语句或命令中添加一个名为__op的字段。__op字段用于指定您想要执行的操作类型。

> **注意**
> 创建表时，您无需在该表中添加名为 `__op` 的列。

根据您选择的加载方法，定义__op字段的方法有所不同：

- 如果选择Stream Load，请使用columns参数来定义__op字段。

- 如果选择Broker Load，请使用SET子句来定义__op字段。

- 如果选择Routine Load，请使用COLUMNS列来定义__op字段。

您可以根据想要进行的数据更改来决定是否添加__op字段。如果不添加__op字段，操作类型默认为UPSERT。主要的数据更改场景如下：

- 如果您要加载的数据文件只涉及UPSERT操作，那么不需要添加__op字段。

- 如果您要加载的数据文件只涉及DELETE操作，那么必须添加__op字段，并将操作类型指定为DELETE。

- 如果您要加载的数据文件涉及UPSERT和DELETE操作，那么必须添加__op字段，并确保数据文件中包含的列的值为0或1。值为0表示执行UPSERT操作，值为1表示执行DELETE操作。

## 使用说明

- 确保您的数据文件中的每一行都有相同数量的列。

- 涉及数据更改的列必须包括主键列。

## 先决条件

### Broker Load

请参见从[HDFS加载数据](../loading/hdfs_load.md)或从[cloud_storage_load.md](../loading/cloud_storage_load.md)中的“背景信息”部分。

### Routine Load

如果您选择Routine Load，请确保在Apache Kafka®集群中创建了主题。假设您已经创建了四个主题：topic1、topic2、topic3和topic4。

## 基本操作

本节提供了如何通过加载来更改StarRocks表中的数据的示例。有关详细的语法和参数描述，请参见[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)和[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

### UPSERT

如果您要加载的数据文件只涉及UPSERT操作，那么不需要添加__op字段。

> **注意**
> 如果您添加了`__op`字段：
- 您可以将操作类型指定为UPSERT。

- 您可以留空__op字段，因为操作类型默认为UPSERT。

#### 数据示例

1. 准备一个数据文件。

   a. 在本地文件系统中创建一个名为example1.csv的CSV文件。该文件包含三列，分别代表用户ID、用户名和用户分数。

   ```Plain
   101,Lily,100
   102,Rose,100
   ```

   b. 将example1.csv的数据发布到您的Kafka集群的topic1。

2. 准备StarRocks表。

   a. 在您的StarRocks数据库test_db中创建一个名为table1的主键表。该表包含三列：id、name和score，其中id是主键。

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
      > 从v2.5.7版本开始，StarRocks在创建表或添加分区时可以自动设置存储桶（BUCKETS）的数量。您不再需要手动设置存储桶的数量。有关详细信息，请参阅[determine the number of buckets](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   b. 向table1插入一条记录。

   ```SQL
   INSERT INTO table1 VALUES
       (101, 'Lily',80);
   ```

#### 加载数据

运行一个加载作业，将example1.csv中id为101的记录更新到table1，并将id为102的记录插入到table1。

- 执行Stream Load作业。

-   如果您不想包含__op字段，请执行以下命令：

    ```Bash
    curl --location-trusted -u <username>:<password> \
        -H "Expect:100-continue" \
        -H "label:label1" \
        -H "column_separator:," \
        -T example1.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
    ```

-   如果您想包含__op字段，请执行以下命令：

    ```Bash
    curl --location-trusted -u <username>:<password> \
        -H "Expect:100-continue" \
        -H "label:label2" \
        -H "column_separator:," \
        -H "columns:__op ='upsert'" \
        -T example1.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
    ```

- 执行Broker Load作业。

-   如果您不想包含__op字段，请执行以下命令：

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

-   如果您想包含__op字段，请执行以下命令：

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

- 执行Routine Load作业。

-   如果您不想包含__op字段，请执行以下命令：

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

-   如果您想包含__op字段，请执行以下命令：

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

加载完成后，查询table1的数据以验证加载是否成功：

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

如上所示的查询结果，example1.csv中id为101的记录已经更新到table1，而id为102的记录已经插入到table1。

### DELETE

如果您要加载的数据文件只涉及DELETE操作，那么必须添加__op字段，并将操作类型指定为DELETE。

#### 数据示例

1. 准备一个数据文件。

   a. 在本地文件系统中创建一个名为example2.csv的CSV文件。该文件包含三列，分别代表用户ID、用户名和用户分数。

   ```Plain
   101,Jack,100
   ```

   b. 将example2.csv的数据发布到您的Kafka集群的topic2。

2. 准备StarRocks表。

   a. 在您的StarRocks数据库test_db中创建一个名为table2的主键表。该表包含三列：id、name和score，其中id是主键。

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
      > 从v2.5.7版本开始，StarRocks在创建表或添加分区时可以自动设置存储桶（BUCKETS）的数量。您不再需要手动设置存储桶的数量。有关详细信息，请参阅[determine the number of buckets](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   b. 向table2插入两条记录。

   ```SQL
   INSERT INTO table2 VALUES
   (101, 'Jack', 100),
   (102, 'Bob', 90);
   ```

#### 加载数据

运行一个加载作业，从table2中删除example2.csv中id为101的记录。

- 执行Stream Load作业。

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label3" \
      -H "column_separator:," \
      -H "columns:__op='delete'" \
      -T example2.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table2/_stream_load
  ```

- 执行Broker Load作业。

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

- 执行Routine Load作业。

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

加载完成后，查询table2的数据以验证加载是否成功：

```SQL
SELECT * FROM table2;
+------+------+-------+
| id   | name | score |
+------+------+-------+
|  102 | Bob  |    90 |
+------+------+-------+
1 row in set (0.00 sec)
```

如上所示的查询结果，example2.csv中id为101的记录已经从table2删除。

### UPSERT和DELETE

如果您要加载的数据文件同时涉及UPSERT和DELETE操作，那么必须添加__op字段，并确保数据文件中包含的列的值为0或1。值为0表示执行UPSERT操作，值为1表示执行DELETE操作。

#### 数据示例

1. 准备一个数据文件。

   a. 在本地文件系统中创建一个名为example3.csv的CSV文件。该文件包含四列，分别代表用户ID、用户名、用户分数和操作类型。

   ```Plain
   101,Tom,100,1
   102,Sam,70,0
   103,Stan,80,0
   ```

   b. 将example3.csv的数据发布到您的Kafka集群的topic3。

2. 准备StarRocks表。

   a. 在您的StarRocks数据库test_db中创建一个名为table3的主键表。该表包含三列：id、name和score，其中id是主键。

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
      > 从v2.5.7版本开始，StarRocks可以自动设置存储桶（BUCKETS）的数量当您创建表或添加分区。您不再需要手动设置存储桶的数量。有关详细信息，请参阅[determine the number of buckets](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   b. 向table3插入两条记录。

   ```SQL
   INSERT INTO table3 VALUES
       (101, 'Tom', 100),
       (102, 'Sam', 90);
   ```

#### 加载数据

运行一个加载作业，从table3中删除example3.csv中id为101的记录，更新id为102的记录，并将id为103的记录插入到table3。

- 执行Stream Load作业：

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
    > 在上面的示例中，`example3.csv`中表示操作类型的第四列暂时命名为`temp`，并且通过使用`columns`参数将`__op`字段映射到`temp`列。这样，StarRocks可以根据`example3.csv`第四列的值是`0`还是`1`来决定执行UPSERT操作还是DELETE操作。

- 执行Broker Load作业：

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

- 执行Routine Load作业：

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

加载完成后，查询table3的数据以验证加载是否成功：

```SQL
SELECT * FROM table3;
+------+------+-------+
| id   | name | score |
+------+------+-------+
|  102 | Sam  |    70 |
|  103 | Stan |    80 |
+------+------+-------+
2 rows in set (0.01 sec)
```

如上所示的查询结果，example3.csv中id为101的记录已经从table3删除，id为102的记录已更新到table3，而id为103的记录已插入到table3。

## 部分更新

从v2.2版本开始，StarRocks支持只更新主键表的指定列。本节以CSV为例，介绍了如何进行部分更新。

> **注意**
> 当执行部分更新时，如果要更新的行不存在，StarRocks将插入一条新行，并为因没有数据更新而为空的字段填充默认值。

### 数据示例

1. 准备一个数据文件。

   a. 在本地文件系统中创建一个名为example4.csv的CSV文件。该文件包含两列，分别代表用户ID和用户名。

   ```Plain
   101,Lily
   102,Rose
   103,Alice
   ```

   b. 将example4.csv的数据发布到您的Kafka集群的topic4。

2. 准备StarRocks表。

   a. 在您的StarRocks数据库test_db中创建一个名为table4的主键表。该表包含三列：id、name和score，其中id是主键。

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
      > 从v2.5.7版本开始，StarRocks可以在创建表或添加分区时自动设置存储桶（BUCKETS）的数量。您不再需要手动设置存储桶的数量。有关详细信息，请参阅[determine the number of buckets](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   b. 向table4插入一条记录。

   ```SQL
   INSERT INTO table4 VALUES
       (101, 'Tom',80);
   ```

### 加载数据

执行加载操作，将example4.csv中的两列数据更新到table4的id和name列。

- 执行Stream Load作业：

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
    > 如果选择**Stream Load**，必须将`partial_update`参数设置为`true`以启用部分更新功能。此外，您必须使用`columns`参数来指定要更新的列。

- 执行Broker Load作业：

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
    > 如果选择**Broker Load**，必须将`partial_update`参数设置为`true`以启用部分更新功能。此外，您必须使用`column_list`参数来指定要更新的列。

- 执行Routine Load作业：

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
    > 如果选择**Broker Load**，必须将`partial_update`参数设置为`true`以启用部分更新功能。此外，您必须使用`COLUMNS`参数来指定要更新的列。

### 查询数据

加载完成后，查询table4的数据以验证加载是否成功：

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

如上所示的查询结果，example4.csv中id为101的记录已更新到table4，而id为102和103的记录已插入到table4。

## 条件更新

从StarRocks v2.5版本开始，主键表支持条件更新。您可以指定一个非主键列作为确定更新是否生效的条件。这样，只有当源数据记录在指定列中的值大于或等于目标数据记录时，源数据到目标数据的更新才会生效。

条件更新功能旨在解决数据混乱问题。如果源数据是无序的，您可以使用此功能确保新数据不会被旧数据覆盖。

> **注意**
- 同一批数据不能指定不同列作为更新条件。
- DELETE操作不支持条件更新。
- 在v3.1.3之前的版本中，不能同时使用部分更新和条件更新。从v3.1.3版本开始，StarRocks支持同时使用部分更新和条件更新。

### 数据示例

1. 准备一个数据文件。

   a. 在本地文件系统中创建一个名为example5.csv的CSV文件。该文件包含三列，分别代表用户ID、版本和用户分数。

   ```Plain
   101,1,100
   102,3,100
   ```

   b. 将example5.csv的数据发布到您的Kafka集群的topic5。

2. 准备StarRocks表。

   a. 在您的StarRocks数据库test_db中创建一个名为table5的主键表。该表包含三列：id、version和score，其中id是主键。

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
      > 从v2.5.7版本开始，StarRocks在创建表或添加分区时可以自动设置存储桶（BUCKETS）的数量。您不再需要手动设置存储桶的数量。有关详细信息，请参阅[determine the number of buckets](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   b. 向table5插入一条记录。

   ```SQL
   INSERT INTO table5 VALUES
       (101, 2, 80),
       (102, 2, 90);
   ```

### 加载数据

执行加载操作，将example5.csv中id分别为101和102的记录更新到table5，并指定更新只在每条记录的version值大于或等于当前版本值时生效。

- 执行Stream Load作业：

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label10" \
      -H "column_separator:," \
      -H "merge_condition:version" \
      -T example5.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table5/_stream_load
  ```

- 执行Routine Load作业：

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

- 执行Broker Load作业：

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

加载完成后，查询table5的数据以验证加载是否成功：

```SQL
SELECT * FROM table5;
+------+------+-------+
| id   | version | score |
+------+------+-------+
|  101 |       2 |   80 |
|  102 |       3 |  100 |
+------+------+-------+
2 rows in set (0.02 sec)
```

如上所示的查询结果，example5.csv中id为101的记录未更新到table5，而id为102的记录已插入到table5。
