---
displayed_sidebar: English
---

# 临时分区

本主题描述了如何使用临时分区功能。

您可以在已定义分区规则的分区表上创建临时分区，并为这些临时分区定义新的数据分发策略。临时分区可以在以原子方式覆盖分区中的数据或调整分区和分桶策略时，作为临时数据载体。对于临时分区，您可以重置数据分发策略，如分区范围、存储桶数量，以及属性，如副本数和存储介质，以满足特定需求。

您可以在以下场景中使用临时分区功能：

- 原子覆盖操作
  
  如果需要在重写分区数据时保证数据可查询，您可以先在原有的分区基础上创建一个临时分区，并将新数据加载到临时分区。然后，您可以使用替换操作以原子方式将原始分区替换为临时分区。有关非分区表的原子覆盖操作，请参见 [ALTER TABLE - SWAP](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md#swap)。

- 调整分区数据查询并发

  如果需要修改分区的桶数，您可以先创建一个与原正式分区范围相同的临时分区，并指定新的桶数。然后，可以使用 `INSERT INTO` 命令将原始正式分区的数据加载到临时分区中。最后，可以使用替换操作以原子方式将原始正式分区替换为临时分区。

- 修改分区规则
  
  如果要修改分区策略，例如合并分区或将一个大分区拆分为多个较小的分区，可以先创建具有预期合并或拆分范围的临时分区。然后，您可以使用 `INSERT INTO` 命令将原始正式分区的数据加载到临时分区中。最后，可以使用替换操作以原子方式将原始正式分区替换为临时分区。

## 创建临时分区

您可以使用 [ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) 命令一次创建一个或多个分区。

### 语法

#### 创建单个临时分区

```SQL
ALTER TABLE <table_name>
ADD TEMPORARY PARTITION <temporary_partition_name> VALUES [("value1"), {MAXVALUE|("value2")})]
[(partition_desc)]
[DISTRIBUTED BY HASH(<bucket_key>)];
ALTER TABLE <table_name> 
ADD TEMPORARY PARTITION <temporary_partition_name> VALUES LESS THAN {MAXVALUE|(<"value">)}
[(partition_desc)]
[DISTRIBUTED BY HASH(<bucket_key>)];
```

#### 一次创建多个分区

```SQL
ALTER TABLE <table_name>
ADD TEMPORARY PARTITIONS START ("value1") END ("value2") EVERY {(INTERVAL <num> <time_unit>)|<num>}
[(partition_desc)]
[DISTRIBUTED BY HASH(<bucket_key>)];
```

### 参数

`partition_desc`：指定临时分区的存储桶数量和属性，如副本数和存储介质。

### 例子

在表 `site_access` 中创建临时分区 `tp1`，并使用 `VALUES [(...), (...)]` 语法指定其范围为 `[2020-01-01, 2020-02-01)`。

```SQL
ALTER TABLE site_access
ADD TEMPORARY PARTITION tp1 VALUES [("2020-01-01"), ("2020-02-01"));
```

在表 `site_access` 中创建临时分区 `tp2`，并使用 `VALUES LESS THAN (...)` 语法指定其上限为 `2020-03-01`。StarRocks 使用上一个临时分区的上限作为该临时分区的下限，生成一个左闭右开范围为 `[2020-02-01, 2020-03-01)` 的临时分区。

```SQL
ALTER TABLE site_access
ADD TEMPORARY PARTITION tp2 VALUES LESS THAN ("2020-03-01");
```

在表 `site_access` 中创建临时分区 `tp3`，使用 `VALUES LESS THAN (...)` 语法指定其上限为 `2020-04-01`，并将副本数指定为 `1`。

```SQL
ALTER TABLE site_access
ADD TEMPORARY PARTITION tp3 VALUES LESS THAN ("2020-04-01")
 ("replication_num" = "1")
DISTRIBUTED BY HASH (site_id);
```

使用 `START (...) END (...) EVERY (...)` 语法在表 `site_access` 中一次创建多个分区，并指定这些分区的范围为 `[2020-04-01, 2021-01-01)`，每月分区粒度。

```SQL
ALTER TABLE site_access 
ADD TEMPORARY PARTITIONS START ("2020-04-01") END ("2021-01-01") EVERY (INTERVAL 1 MONTH);
```

### 使用说明

- 临时分区的分区列必须与创建临时分区所基于的原始正式分区的分区列相同，并且不能更改。
- 临时分区的名称不能与任何正式分区或其他临时分区的名称相同。
- 表中所有临时分区的范围不能重叠，但临时分区和正式分区的范围可以重叠。

## 显示临时分区

您可以使用 [SHOW TEMPORARY PARTITIONS](../sql-reference/sql-statements/data-manipulation/SHOW_PARTITIONS.md) 命令查看临时分区。

```SQL
SHOW TEMPORARY PARTITIONS FROM [db_name.]table_name [WHERE] [ORDER BY] [LIMIT]
```

## 将数据加载到临时分区中

您可以使用 `INSERT INTO` 命令、STREAM LOAD 或 BROKER LOAD 将数据加载到一个或多个临时分区中。

### 使用 `INSERT INTO` 命令加载数据 

例：

```SQL
INSERT INTO site_access TEMPORARY PARTITION (tp1) VALUES ("2020-01-01",1,"ca","lily",4);
INSERT INTO site_access TEMPORARY PARTITION (tp2) SELECT * FROM site_access_copy PARTITION p2;
INSERT INTO site_access TEMPORARY PARTITION (tp3, tp4,...) SELECT * FROM site_access_copy PARTITION (p3, p4,...);
```

有关语法和参数的详细说明，请参见 [INSERT INTO](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

### 使用 STREAM LOAD 加载数据

例：

```bash
curl --location-trusted -u root: -H "label:123" -H "Expect:100-continue" -H "temporary_partitions: tp1, tp2, ..." -T testData \
    http://host:port/api/example_db/site_access/_stream_load    
```

有关语法和参数的详细说明，请参见 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。

### 使用 BROKER LOAD 装入数据

例：

```SQL
LOAD LABEL example_db.label1
(
    DATA INFILE("hdfs://hdfs_host:hdfs_port/user/starrocks/data/input/file")
    INTO TABLE my_table
    TEMPORARY PARTITION (tp1, tp2, ...)
    ...
)
WITH BROKER
(
    StorageCredentialParams
);
```

请注意，`StorageCredentialParams` 表示一组身份验证参数，这些参数因您选择的身份验证方法而异。有关语法和参数的详细描述，请参见 [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

### 使用 ROUTINE LOAD 加载数据


例子：

```SQL
CREATE ROUTINE LOAD example_db.site_access ON example_tbl
COLUMNS(col, col2,...),
TEMPORARY PARTITIONS(tp1, tp2, ...)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest"
);
```

有关详细的语法和参数描述，请参阅 [CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

## 查询临时分区中的数据

您可以使用 [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 语句查询指定临时分区中的数据。

```SQL
SELECT * FROM
site_access TEMPORARY PARTITION (tp1);

SELECT * FROM
site_access TEMPORARY PARTITION (tp1, tp2, ...);

SELECT event_day,site_id,pv FROM
site_access TEMPORARY PARTITION (tp1, tp2, ...);
```

您可以使用 JOIN 子句从两个表中查询临时分区中的数据。

```SQL
SELECT * FROM
site_access TEMPORARY PARTITION (tp1, tp2, ...)
JOIN
site_access_copy TEMPORARY PARTITION (tp1, tp2, ...)
ON site_access.site_id=site_access1.site_id and site_access.event_day=site_access1.event_day;
```

## 用临时分区替换原始正式分区

您可以使用 [ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) 语句将原始正式分区替换为临时分区，从而创建新的正式分区。

> **注意**
>
> 在 ALTER TABLE 语句中操作的原始正式分区和临时分区将被删除，并且无法恢复。

### 语法

```SQL
ALTER TABLE table_name REPLACE PARTITION (partition_name) WITH TEMPORARY PARTITION (temporary_partition_name1, ...)
PROPERTIES ("key" = "value");
```

### 参数

- **strict_range**
  
  默认值： `true`。

  当此参数设置为 `true` 时，所有原始正式分区的范围并集必须与用于替换的临时分区的范围并集完全相同。当该参数设置为 `false` 时，只需确保新正式分区的范围在替换后不与其他正式分区重叠。

  - 示例 1：
  
    在以下示例中，原始正式分区 `p1`, `p2`, 和 `p3` 的范围并集与临时分区 `tp1` 和 `tp2` 的范围并集相同，您可以使用 `tp1` 和 `tp2` 替换 `p1`, `p2`, 和 `p3`。

      ```plaintext
      # 原始正式分区 p1, p2, 和 p3 的范围 => 这些范围的并集
      [10, 20), [20, 30), [40, 50) => [10, 30), [40, 50)
      
      # 临时分区 tp1 和 tp2 的范围 => 这些范围的并集
      [10, 30), [40, 45), [45, 50) => [10, 30), [40, 50)
      ```

  - 示例 2：

    在以下示例中，原始正式分区的范围并集与临时分区的范围并集不同。如果参数 strict_range 的值设置为 true，临时分区 `tp1` 和 `tp2` 不能替换原始正式分区 `p1`。如果该值设置为 false，并且临时分区的范围 [10, 30) 和 [40, 50) 不与其他正式分区重叠，则临时分区可以替换原始正式分区。

      ```plaintext
      # 原始正式分区 p1 的范围 => 这个范围的并集
      [10, 50) => [10, 50)
      
      # 临时分区 tp1 和 tp2 的范围 => 这些范围的并集
      [10, 30), [40, 50) => [10, 30), [40, 50)
      ```

- **use_temp_partition_name**
  
  默认值： `false`。

  如果原始正式分区数与用于替换的临时分区数相同，则当该参数设置为 `false` 时，替换后新正式分区的名称保持不变。当此参数设置为 `true` 时，临时分区的名称将用作替换后新的正式分区的名称。

  在以下示例中，当此参数设置为 `false` 时，替换后新正式分区的分区名称保持为 `p1`。但是，其相关数据和属性将替换为临时分区 `tp1` 的数据和属性。当此参数设置为 `true` 时，替换后新正式分区的分区名称将更改为 `tp1`。原始正式分区 `p1` 不再存在。

    ```sql
    ALTER TABLE tbl1 REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
    ```

  如果要替换的正式分区数与用于替换的临时分区数不同，并且此参数仍为默认值 `false`，则此参数的值 `false` 无效。

  在以下示例中，替换后新正式分区的名称更改为 `tp1`，并且原始正式分区 `p1` 和 `p2` 不再存在。

    ```SQL
    ALTER TABLE site_access REPLACE PARTITION (p1, p2) WITH TEMPORARY PARTITION (tp1);
    ```

### 例子

将原始正式分区 `p1` 替换为临时分区 `tp1`。

```SQL
ALTER TABLE site_access REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
```

将原始正式分区 `p2` 和 `p3` 替换为临时分区 `tp2` 和 `tp3`。

```SQL
ALTER TABLE site_access REPLACE PARTITION (p2, p3) WITH TEMPORARY PARTITION (tp2, tp3);
```

将原始正式分区 `p4` 和 `p5` 替换为临时分区 `tp4` 和 `tp5`，并指定参数 `strict_range` 为 `false` 和 `use_temp_partition_name` 为 `true`。

```SQL
ALTER TABLE site_access REPLACE PARTITION (p4, p5) WITH TEMPORARY PARTITION (tp4, tp5)
PROPERTIES (
    "strict_range" = "false",
    "use_temp_partition_name" = "true"
);
```

### 使用说明

- 当表具有临时分区时，不能使用 `ALTER` 命令对表执行 Schema Change 操作。
- 在对表执行 Schema Change 操作时，不能向表添加临时分区。

## 删除临时分区

使用以下命令删除临时分区 `tp1`。

```SQL
ALTER TABLE site_access DROP TEMPORARY PARTITION tp1;
```

请注意以下限制：

- 如果使用 `DROP` 命令直接删除数据库或表，则可以使用 `RECOVER` 命令在有限的时间段内恢复该数据库或表。但是，无法恢复临时分区。
- 使用 `ALTER` 命令删除正式分区后，您可以使用 `RECOVER` 命令在有限的时间段内恢复该分区。临时分区不与正式分区绑定，因此对临时分区的操作不会影响正式分区。
- 使用 `ALTER` 命令删除临时分区后，无法使用 `RECOVER` 命令恢复该分区。
- 使用 `TRUNCATE` 命令删除表中的数据时，表的临时分区将被删除，且无法恢复。
- 使用 `TRUNCATE` 命令删除正式分区中的数据时，临时分区不受影响。
- TRUNCATE 命令不能用于删除临时分区中的数据。