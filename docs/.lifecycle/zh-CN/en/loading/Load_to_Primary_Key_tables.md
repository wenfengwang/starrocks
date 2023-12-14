---
displayed_sidebar：“英语”
---

# 通过加载改变数据

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

通过 StarRocks 提供的[主键表](../table_design/table_types/primary_key_table.md)，您可以通过运行 [流加载](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)，[Broker加载](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)，或者 [创建例程加载](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) 作业来对 StarRocks 表进行数据更改。这些数据更改包括插入、更新和删除。但是，主键表不支持使用 [Spark加载](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) 或 [INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md) 来更改数据。

StarRocks 还支持部分更新和条件更新。

<InsertPrivNote />

本主题以CSV数据为例，描述了如何通过加载对StarRocks表进行数据更改。所支持的数据文件格式取决于您选择的加载方法。

> **注意**
>
> 对于CSV数据，您可以使用不超过50字节的UTF-8字符串（例如逗号（,）、制表符或竖线（|））作为文本分隔符。

## 实施

由 StarRocks 提供的主键表支持 UPSERT 和 DELETE 操作，并且不区分 INSERT 操作和 UPDATE 操作。

当您创建一个加载作业时，StarRocks 支持在作业创建语句或命令中添加名为 `__op` 的字段。`__op` 字段用于指定您要执行的操作类型。

> **注意**
>
> 当您创建表时，不需要向该表添加名为 `__op` 的列。

根据您选择的加载方法，定义 `__op` 字段的方法各不相同：

- 如果选择流加载，请使用 `columns` 参数定义 `__op` 字段。

- 如果选择Broker加载，请使用 SET 子句定义 `__op` 字段。

- 如果选择例程加载，请使用 `COLUMNS` 列定义 `__op` 字段。

您可以根据您希望进行的数据更改决定是否添加 `__op` 字段。如果不添加 `__op` 字段，操作类型默认为 UPSERT。主要的数据更改场景如下所示：

- 如果要加载的数据文件仅涉及 UPSERT 操作，则无需添加 `__op` 字段。

- 如果要加载的数据文件仅涉及 DELETE 操作，则必须添加 `__op` 字段，并将操作类型指定为 DELETE。

- 如果要加载的数据文件涉及 UPSERT 和 DELETE 操作，则必须添加 `__op` 字段，并确保数据文件包含一个其值为 `0` 或 `1` 的列。值 `0` 表示 UPSERT 操作，值 `1` 表示 DELETE 操作。

## 使用注意事项

- 确保数据文件中的每一行具有相同数量的列。

- 参与数据更改的列必须包括主键列。

## 先决条件

### Broker加载

请参见[从HDFS加载数据](../loading/hdfs_load.md)和[从云存储加载数据](../loading/cloud_storage_load.md)中的“背景信息”部分。

### 例程加载

如果选择例程加载，请确保在您的 Apache Kafka® 集群中创建了主题。假设您已经创建了四个主题：`topic1`，`topic2`，`topic3`和`topic4`。

## 基本操作

本节提供了通过加载对 StarRocks 表进行数据更改的示例。有关详细的语法和参数说明，请参见 [流加载](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)，[Broker加载](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 和 [创建例程加载](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

### UPSERT

如果要加载的数据文件仅涉及 UPSERT 操作，则无需添加 `__op` 字段。

> **注意**
>
> 如果添加 `__op` 字段：
>
> - 您可以将操作类型指定为UPSERT。
>
> - 您可以将 `__op` 字段保留为空，因为默认的操作类型为 UPSERT。

#### 数据示例

1. 准备一个数据文件。

   a. 在您的本地文件系统中创建名为 `example1.csv` 的 CSV 文件。该文件包括三列，依次表示用户 ID、用户名和用户分数。

      ```Plain
      101,Lily,100
      102,Rose,100
      ```

   b. 将 `example1.csv` 的数据发布到您的Kafka集群的`topic1`。

2. 准备一个 StarRocks 表。

   a. 在您的StarRocks数据库`test_db`中创建一个名为`table1`的主键表。该表包括三列：`id`、`name`和`score`，其中 `id` 是主键。

      ```SQL
      CREATE TABLE `table1`
      (
          `id` int(11) NOT NULL COMMENT "用户ID",
          `name` varchar(65533) NOT NULL COMMENT "用户姓名",
          `score` int(11) NOT NULL COMMENT "用户分数"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`)
      DISTRIBUTED BY HASH(`id`);
      ```

      > **注意**
      >
      > 自 v2.5.7 起，StarRocks 在您创建表或添加分区时可以自动设置桶的数量（BUCKETS）。您不再需要手动设置桶的数量。有关详细信息，请参见[确定桶的数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   b. 向 `table1` 插入一条记录。

      ```SQL
      INSERT INTO table1 VALUES
          (101, 'Lily',80);
      ```

#### 加载数据

运行一个加载作业，以更新 `example1.csv` 中 `id` 为 `101` 的记录到 `table1` 中，并把 `example1.csv` 中 `id` 为 `102` 的记录插入到 `table1` 中。

- 运行流加载作业。

  - 如果不想包括 `__op` 字段，请运行以下命令：

    ```Bash
    curl --location-trusted -u <username>:<password> \
        -H "Expect:100-continue" \
        -H "label:label1" \
        -H "column_separator:," \
        -T example1.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
    ```

  - 如果想包括 `__op` 字段，请运行以下命令：

    ```Bash
    curl --location-trusted -u <username>:<password> \
        -H "Expect:100-continue" \
        -H "label:label2" \
        -H "column_separator:," \
        -H "columns:__op ='upsert'" \
        -T example1.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
    ```

- 运行 Broker 加载作业。

  - 如果不想包括 `__op` 字段，请运行以下命令：

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

  - 如果想包括 `__op` 字段，请运行以下命令：

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

- 运行例程加载作业。

  - 如果不想包括 `__op` 字段，请运行以下命令：

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

  - 如果想包括 `__op` 字段，请运行以下命令：

    ```SQL
    CREATE ROUTINE LOAD test_db.table1 ON table1
    列终止于",",
    列(id, name, score, __op ='upsert')
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

加载完成后，查询`table1`的数据以验证加载是否成功：

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

正如上述查询结果所示，`example1.csv`中`id`为`101`的记录已被更新到`table1`，`example1.csv`中`id`为`102`的记录已被插入到`table1`。

### DELETE

如果要加载的数据文件仅涉及删除操作，则必须添加`__op`字段并指定操作类型为DELETE。

#### 数据示例

1. 准备数据文件。

   a. 在本地文件系统中创建名为`example2.csv`的CSV文件。该文件包含三列，依次代表用户ID，用户名和用户得分。

      ```Plain
      101,Jack,100
      ```

   b. 将`example2.csv`的数据发布到您的Kafka集群的`topic2`中。

2. 准备StarRocks表。

   a. 在StarRocks表`test_db`中创建名为`table2`的主键表。该表包含三列：`id`，`name`和`score`，其中`id`为主键。

      ```SQL
      CREATE TABLE `table2`
            (
          `id` int(11) NOT NULL COMMENT "用户ID",
          `name` varchar(65533) NOT NULL COMMENT "用户名",
          `score` int(11) NOT NULL COMMENT "用户得分"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`)
      DISTRIBUTED BY HASH(`id`);
      ```

      > **注意**
      >
      > 自v2.5.7起，StarRocks在创建表或添加分区时可以自动设置存储桶的数量(BUCKETS)。您无需手动设置存储桶的数量。有关详细信息，请参见[确定存储桶的数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   b. 向`table2`中插入两条记录。

      ```SQL
      INSERT INTO table2 VALUES
      (101, 'Jack', 100),
      (102, 'Bob', 90);
      ```

#### 加载数据

运行一个加载作业，将`example2.csv`中`id`为`101`的记录从`table2`中删除。

- 运行流式加载作业。

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label3" \
      -H "column_separator:," \
      -H "columns:__op='delete'" \
      -T example2.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table2/_stream_load
  ```

- 运行Broker加载作业。

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

- 运行例行加载作业。

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

加载完成后，查询`table2`的数据以验证加载是否成功：

```SQL
SELECT * FROM table2;
+------+------+-------+
| id   | name | score |
+------+------+-------+
|  102 | Bob  |    90 |
+------+------+-------+
1 row in set (0.00 sec)
```

如上述查询结果所示，`example2.csv`中`id`为`101`的记录已从`table2`中被删除。

### UPSERT和DELETE

如果要加载的数据文件涉及UPSERT和DELETE操作，必须添加`__op`字段，并确保数据文件包含一个值为`0`或`1`的列。值为`0`表示UPSERT操作，值为`1`表示DELETE操作。

#### 数据示例

1. 准备数据文件。

   a. 在本地文件系统中创建名为`example3.csv`的CSV文件。该文件包含四列，依次代表用户ID，用户名，用户得分和操作类型。

      ```Plain
      101,Tom,100,1
      102,Sam,70,0
      103,Stan,80,0
      ```

   b. 将`example3.csv`的数据发布到您的Kafka集群的`topic3`中。

2. 准备StarRocks表。

   a. 在StarRocks数据库`test_db`中创建名为`table3`的主键表。该表包含三列：`id`，`name`和`score`，其中`id`为主键。

      ```SQL
      CREATE TABLE `table3`
      (
          `id` int(11) NOT NULL COMMENT "用户ID",
          `name` varchar(65533) NOT NULL COMMENT "用户名",
          `score` int(11) NOT NULL COMMENT "用户得分"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`)
      DISTRIBUTED BY HASH(`id`);
      ```

      > **注意**
      >
      > 自v2.5.7起，StarRocks在创建表或添加分区时可以自动设置存储桶的数量(BUCKETS)。您无需手动设置存储桶的数量。有关详细信息，请参见[确定存储桶的数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   b. 向`table3`中插入两条记录。

      ```SQL
      INSERT INTO table3 VALUES
          (101, 'Tom', 100),
          (102, 'Sam', 90);
      ```

#### 加载数据

运行一个加载作业，从`table3`中删除`example3.csv`中`id`为`101`的记录，更新`example3.csv`中`id`为`102`的记录并将其更新到`table3`，并将`example3.csv`中`id`为`103`的记录插入`table3`。

- 运行流式加载作业：

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
  > 在上述示例中，代表`example3.csv`中操作类型的第四列临时命名为`temp`，并使用`columns`参数将`__op`字段映射到`temp`列。因此，StarRocks可以根据`example3.csv`的第四列中的值是`0`还是`1`来决定是执行UPSERT操作还是DELETE操作。

- 运行Broker加载作业：

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

- 运行例行加载作业：

  ```SQL
  CREATE ROUTINE LOAD test_db.table3 ON table3
```SQL
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

#### 查询数据

在加载完成后，查询 `table3` 的数据以验证加载是否成功：

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

如上述查询结果所示，在 `example3.csv` 中 `id` 为 `101` 的记录已从 `table3` 中删除， `example3.csv` 中 `id` 为 `102` 的记录已更新到 `table3`， `example3.csv` 中 `id` 为 `103` 的记录已插入到 `table3`。

## 部分更新

自 v2.2 以来，StarRocks 支持仅更新主键表的指定列。本部分以 CSV 为例，描述如何执行部分更新操作。

> **注意**
>
> 在执行部分更新时，如果要更新的行不存在，StarRocks 会插入新行，并在因无数据更新而在其中填充默认值的字段中填充空值。

### 数据示例

1. 准备一个数据文件。

   a. 在您的本地文件系统中创建名为 `example4.csv` 的 CSV 文件。该文件由两列组成，依次表示用户 ID 和用户名。

      ```Plain
      101,Lily
      102,Rose
      103,Alice
      ```

   b. 将 `example4.csv` 的数据发布到您的 Kafka 集群的 `topic4` 中。

2. 准备一个 StarRocks 表。

   a. 在 StarRocks 数据库 `test_db` 中创建名为 `table4` 的主键表。该表由三列组成：`id`、`name` 和 `score`，其中 `id` 是主键。

      ```SQL
      CREATE TABLE `table4`
      (
          `id` int(11) NOT NULL COMMENT "用户 ID",
          `name` varchar(65533) NOT NULL COMMENT "用户姓名",
          `score` int(11) NOT NULL COMMENT "用户分数"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`)
      DISTRIBUTED BY HASH(`id`);
      ```

      > **注意**
      >
      > 自 v2.5.7 起，StarRocks 可以在创建表或添加分区时自动设置存储桶的数量 (BUCKETS)。您无须再手动设置存储桶的数量。有关详细信息，请参阅[确定存储桶数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   b. 将一条记录插入到 `table4`。

      ```SQL
      INSERT INTO table4 VALUES
          (101, 'Tom',80);
      ```

### 加载数据

运行加载操作，将 `example4.csv` 中的数据在 `table4` 的 `id` 和 `name` 列中更新。

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
  > 如果选择流加载，您必须将 `partial_update` 参数设置为 `true` 以启用部分更新功能。此外，您必须使用 `columns` 参数指定要更新的列。

- 运行 Broker 加载作业：

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
  > 如果选择 Broker 加载，您必须将 `partial_update` 参数设置为 `true` 以启用部分更新功能。此外，您必须使用 `column_list` 参数指定要更新的列。

- 运行常规加载作业：

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
  > 如果选择 Broker 加载，您必须将 `partial_update` 参数设置为 `true` 以启用部分更新功能。此外，您必须使用 `COLUMNS` 参数指定要更新的列。

### 查询数据

加载完成后，查询 `table4` 的数据以验证加载是否成功：

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

如上述查询结果所示，在 `example4.csv` 中 `id` 为 `101` 的记录已更新到 `table4`， `example4.csv` 中 `id` 为 `102` 和 `103` 的记录已插入到 `table4`。

## 条件更新

从 StarRocks v2.5 起，主键表支持条件更新。您可以指定非主键列作为条件，以确定是否可以生效数据更新。因此，仅当指定列中的源数据记录的值大于或等于指定列中的目标数据记录时，源记录才会更新到目标记录。

条件更新功能旨在解决数据失序问题。如果源数据失序，您可以使用此功能确保新数据不会被旧数据覆盖。

> **注意**
>
> - 您不能为同批数据指定不同的列作为更新条件。
> - DELETE 操作不支持条件更新。
> - 在 v3.1.3 版本之前，不能同时使用部分更新和条件更新。从 v3.1.3 起，StarRocks 支持同时使用部分更新和条件更新。
> - 仅支持流加载和常规加载支持条件更新。

### 数据示例

1. 准备一个数据文件。

   a. 在您的本地文件系统中创建名为 `example5.csv` 的 CSV 文件。该文件由三列组成，依次表示用户 ID、版本和用户分数。

      ```Plain
      101,1,100
      102,3,100
      ```

   b. 将 `example5.csv` 的数据发布到您的 Kafka 集群的 `topic5` 中。

2. 准备一个 StarRocks 表。

   a. 在 StarRocks 数据库 `test_db` 中创建名为 `table5` 的主键表。该表由三列组成：`id`、`version` 和 `score`，其中 `id` 是主键。

      ```SQL
      CREATE TABLE `table5`
      (
          `id` int(11) NOT NULL COMMENT "用户 ID", 
          `version` int NOT NULL COMMENT "版本",
          `score` int(11) NOT NULL COMMENT "用户分数"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`) DISTRIBUTED BY HASH(`id`);
      ```

      > **注意**
      >
      > 自 v2.5.7 起，StarRocks 可以在创建表或添加分区时自动设置存储桶的数量 (BUCKETS)。您无须再手动设置存储桶的数量。有关详细信息，请参阅[确定存储桶数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

   b. 将两条记录插入到 `table5`。

      ```SQL
      INSERT INTO table5 VALUES
          (101, 2, 80),
          (102, 2, 90);
      ```

### 加载数据

运行一个加载任务，将`id`值分别为`101`和`102`的记录从`example5.csv`更新到`table5`，并指定只有当两个记录中的`version`值大于或等于它们当前的`version`值时更新生效。

- 运行一个流式加载任务：

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label10" \
      -H "column_separator:," \
      -H "merge_condition:version" \
      -T example5.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table5/_stream_load
  ```

- 运行一个常规加载任务：

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
2 rows in set (0.02 sec)
```

如上查询结果所示，`example5.csv`中`id`为`101`的记录未更新到`table5`，而`example5.csv`中`id`为`102`的记录已插入到`table5`。