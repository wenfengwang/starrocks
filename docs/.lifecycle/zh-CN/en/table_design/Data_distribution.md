---
displayed_sidebar: "Chinese"
---

# 数据分布

在创建表时配置适当的分区和存储桶，可以帮助实现均匀的数据分布。均匀的数据分布意味着根据某些规则将数据划分为子集，并将其均匀分布在不同的节点上。这样可以减少扫描的数据量，并充分利用集群的并行处理能力，从而提高查询性能。

> **注意**
>
> - 从 v3.1 开始，在创建表或添加分区时，无需在 `DISTRIBUTED BY` 子句中指定存储桶键。StarRocks 支持随机存储桶分布，可以将数据随机分布到所有存储桶。有关更多信息，请参见[随机存储桶](#random-bucketing-since-v31)。
> - 从 v2.5.7 开始，您可以选择在创建表或添加分区时不手动设置存储桶的数量。StarRocks 可以自动设置存储桶的数量（BUCKETS）。然而，如果StarRocks自动设置存储桶数量后性能不符合您的期望，并且您熟悉存储桶机制，您仍可以[手动设置存储桶数量](#determine-the-number-of-buckets)。

## 数据分布方法

### 一般的数据分布方法

现代分布式数据库系统通常使用以下基本的数据分布方法：循环分布（Round-Robin）、范围分布（Range）、列表分布（List）和哈希分布（Hash）。

![数据分布方法](../assets/3.3.2-1.png)

- **循环分布**：将数据循环分布到不同节点上。
- **范围分布**：根据分区列值的范围将数据分布到不同节点上。如图中所示，范围[1-3]和[4-6]分别对应不同的节点。
- **列表分布**：根据分区列的离散值将数据分布到不同节点上，例如性别和省份。每个离散值映射到一个节点，多个不同的值可能映射到同一个节点上。
- **哈希分布**：根据哈希函数将数据分布到不同节点上。

为了实现更灵活的数据分区，除了使用上述某一种数据分布方法外，还可以根据具体的业务需求将这些方法结合起来使用。常见的组合包括哈希+哈希、范围+哈希和哈希+列表。

### StarRocks 中的数据分布方法

StarRocks 支持数据分布方法的独立使用和组合使用。

> **注意**
>
> 除了一般的分布方法外，StarRocks 还支持随机分布，以简化存储桶的配置。

此外，StarRocks 通过实现两级分区+存储桶方法来分布数据。

- 第一级是分区：表中的数据可以进行分区。支持的分区方法有表达式分区、范围分区和列表分区。或者您可以选择不使用分区（将整个表视为一个分区）。
- 第二级是存储桶：分区中的数据需要进一步分布到更小的存储桶中。支持的存储桶方法有哈希和随机存储桶。

| **数据分布方法**         | **分区和存储桶方法**                     | **描述**                            |
| ------------------------- | ---------------------------------------- | ------------------------------------- |
| 随机分布                 | 随机存储桶                               | 将整个表视为一个分区。表中的数据随机分布到不同的存储桶。这是默认的数据分布方法。 |
| 哈希分布                 | 哈希存储桶                               | 将整个表视为一个分区。表中的数据根据哈希函数的哈希值分布到相应的存储桶。 |
| 范围+随机分布           | <ol><li>表达式分区或范围分区 </li><li>随机存储桶 </li></ol> | <ol><li>表中的数据根据分区列值所在范围分布到相应的分区。</li><li>分区中的数据随机分布到不同的存储桶。 </li></ol> |
| 范围+哈希分布           | <ol><li>表达式分区或范围分区</li><li>哈希存储桶 </li></ol> | <ol><li>表中的数据根据分区列值所在范围分布到相应的分区。</li><li>分区中的数据根据哈希函数的哈希值分布到相应的存储桶。 </li></ol> |
| 列表+随机分布           | <ol><li>表达式分区或列表分区</li><li>随机存储桶 </li></ol> | <ol><li>表中的数据根据分区列值所在范围分布到相应的分区。</li><li>分区中的数据随机分布到不同的存储桶。 </li></ol> |
| 列表+哈希分布           | <ol><li>表达式分区或列表分区</li><li>哈希存储桶 </li></ol> | <ol><li>表中的数据根据分区列值所在分区值列表进行分区。</li><li>分区中的数据根据哈希函数的哈希值分布到相应的存储桶。</li></ol> |

- **随机分布**

  如果在创建表时未配置分区和存储桶方法，则默认使用随机分布。当前随机分布方法只能用于创建重复键表。

  ```SQL
  CREATE TABLE site_access1 (
      event_day DATE,
      site_id INT DEFAULT '10', 
      pv BIGINT DEFAULT '0' ,
      city_code VARCHAR(100),
      user_name VARCHAR(32) DEFAULT ''
  )
  DUPLICATE KEY (event_day,site_id,pv);
  -- 因为未配置分区和存储桶方法，将默认使用随机分布。
  ```

- **哈希分布**

  ```SQL
  CREATE TABLE site_access2 (
      event_day DATE,
      site_id INT DEFAULT '10',
      city_code SMALLINT,
      user_name VARCHAR(32) DEFAULT '',
      pv BIGINT SUM DEFAULT '0'
  )
  AGGREGATE KEY (event_day, site_id, city_code, user_name)
  -- 使用哈希存储桶作为存储桶方法，并且必须指定存储桶键。
  DISTRIBUTED BY HASH(event_day,site_id); 
  ```

- **范围+随机分布**（当前此分布方法只能用于创建重复键表。

  ```SQL
  CREATE TABLE site_access3 (
      event_day DATE,
      site_id INT DEFAULT '10', 
      pv BIGINT DEFAULT '0' ,
      city_code VARCHAR(100),
      user_name VARCHAR(32) DEFAULT ''
  )
  DUPLICATE KEY(event_day,site_id,pv)
  -- 使用表达式分区作为分区方法，并配置时间函数表达式。
  -- 也可以使用范围分区。
  PARTITION BY date_trunc('day', event_day);
  -- 因为未配置存储桶方法，将默认使用随机存储桶。
  ```

- **范围+哈希分布**

  ```SQL
  CREATE TABLE site_access4 (
      event_day DATE,
      site_id INT DEFAULT '10',
      city_code VARCHAR(100),
      user_name VARCHAR(32) DEFAULT '',
      pv BIGINT SUM DEFAULT '0'
  )
  AGGREGATE KEY(event_day, site_id, city_code, user_name)
  -- 使用表达式分区作为分区方法，并配置时间函数表达式。
  -- 也可以使用范围分区。
  PARTITION BY date_trunc('day', event_day)
  -- 使用哈希存储桶作为存储桶方法，并且必须指定存储桶键。
  DISTRIBUTED BY HASH(event_day, site_id);
  ```

- **列表+随机分布**（当前此分布方法只能用于创建重复键表。

  ```SQL
  CREATE TABLE t_recharge_detail1 (
      id bigint,
      user_id bigint,
      recharge_money decimal(32,2), 
      city varchar(20) not null,
      dt date not null
  )
  DUPLICATE KEY(id)
  -- 使用表达式分区作为分区方法，并指定分区列。
  -- 也可以使用列表分区。
  PARTITION BY (city);
  -- 因为未配置存储桶方法，将默认使用随机存储桶。
  ```

- **列表+哈希分布**

  ```SQL
  CREATE TABLE t_recharge_detail2 (
      id bigint,
      user_id bigint,
      recharge_money decimal(32,2), 
      city varchar(20) not null,
      dt date not null
  )
  DUPLICATE KEY(id)
  -- 使用表达式分区作为分区方法，并指定分区列。
  -- 也可以使用列表分区。
  PARTITION BY (city)
  -- 使用哈希存储桶作为存储桶方法，并且必须指定存储桶键。
  DISTRIBUTED BY HASH(city,id); 
  ```

#### 分区

分区方法将表分为多个分区。主要用于根据分区键将表分割为不同的管理单元（分区）。您可以为每个分区设置存储策略，包括桶的数量、热数据和冷数据的存储策略、存储介质的类型以及副本的数量。StarRocks允许您在集群内使用不同类型的存储介质。例如，您可以将最新数据存储在固态硬盘(SSD)上以提高查询性能，并将历史数据存储在SATA硬盘上以降低存储成本。

| **分区方法**                   | **场景**                                                     | **创建分区的方法**               |
| -------------------------- | --------------------------------------------------------- | ---------------------------- |
| 表达式分区（推荐）                  | 以前称为自动分区。这种分区方法更灵活、易于使用。适用于大多数场景，包括基于连续日期范围或枚举值进行查询和管理数据。 | 在数据加载期间自动创建       |
| 范围分区                     | 典型场景是存储简单、有序的数据，通常根据连续的日期/数字范围进行查询和管理。例如，在某些特殊情况下，历史数据需要按月进行分区，而最近的数据需要按日进行分区。 | 手动、动态或批量创建             |
| 列表分区                     | 典型场景是根据枚举值进行查询和管理数据，且一个分区需要包含分区列的不同值的数据。例如，如果您经常根据国家和城市进行查询和管理数据，您可以使用此方法，并选择`city`作为分区列。因此，一个分区可以存储属于同一国家的多个城市的数据。 | 手动创建                       |

##### 如何选择分区列和粒度

- 选择适当的分区列可以有效地减少查询时扫描的数据量。在大多数业务系统中，基于时间的分区通常被广泛采用以解决由于过期数据的删除而引起的某些问题，并促进热数据和冷数据的层式存储管理。在这种情况下，您可以使用表达式分区或范围分区，并指定时间列作为分区列。此外，如果数据经常根据枚举值进行查询和管理，可以使用表达式分区或列表分区，并指定一个包含这些值的列作为分区列。
- 在选择分区粒度时，需要考虑数据量、查询模式和数据管理粒度等因素。
  - 例1: 如果表中每月的数据量较小，按月分区可减少元数据的数量，从而降低元数据管理和调度的资源消耗。
  - 例2: 如果表中每月的数据量较大，并且大多数查询大多数请求的数据是某些特定日期的数据，则按日分区可有效地减少查询时扫描的数据量。
  - 例3: 如果数据需要按日过期，则建议按日分区。

#### 分桶

分桶方法将分区分割为多个桶。一个桶中的数据称为一个tablet。

支持的分桶方法有[随机分桶](#random-bucketing-since-v31) (从v3.1开始) 和[哈希分桶](#hash-bucketing)。

- 随机分桶: 在创建表或添加分区时，无需设置分桶键。分区内的数据被随机分布到不同的桶中。

- 哈希分桶: 在创建表或添加分区时，需要指定一个分桶键。同一分区内的数据根据分桶键的值被划分到不同的桶中，并具有相同分桶键值的行会被分配到对应的唯一桶中。

桶的数量: 默认情况下，StarRocks自动设置桶的数量（从v2.5.7开始）。您也可以手动设置桶的数量。有关更多信息，请参阅[确定桶的数量](#determine-the-number-of-buckets)。

## 创建和管理分区

### 创建分区

#### 表达式分区（推荐）

> **注意**
>
> 自v3.1起，StarRocks的[共享数据模式](../deployment/shared_data/s3.md)支持时间函数表达式，不支持列表达式。

自v3.0起，StarRocks支持[表达式分区](./expression_partitioning.md)（以前称为自动分区），这种分区方法更灵活、易于使用。此分区方法适用于大多数场景，例如基于连续日期范围或枚举值进行查询和管理数据。

您只需要在表创建时配置一个分区表达式（时间函数表达式或列表达式），StarRocks将在数据加载期间自动创建分区。您不再需要提前手动创建大量分区，也不需要配置动态分区属性。

#### 范围分区

范围分区适用于存储简单、连续的数据，例如时间序列数据（日期或时间戳）或连续的数值数据。您经常根据连续的日期/数字范围查询和管理数据。它还可以应用在某些特殊情况下，例如历史数据需要按月进行分区，最近的数据需要按日进行分区。

StarRocks根据显式定义的每个分区的范围的映射来存储相应分区内的数据。

##### 动态分区

动态分区相关属性是在表创建时配置的。StarRocks会提前自动创建新的分区并移除过期分区，以确保数据的新鲜度，实现分区的生存周期（TTL）管理。

与表达式分区提供的自动分区创建能力不同，动态分区只能基于属性定期创建新的分区。如果新数据不属于这些分区，则加载作业将返回错误。然而，表达式分区提供的自动分区创建能力能够始终基于加载的数据创建相应的新分区。

##### 手动创建分区

使用适当的分区键可以有效地减少查询时扫描的数据量。目前，只有日期或整数类型的列可以选为分区键组成分区键。在业务场景中，分区键通常是基于数据管理的角度选择的。常见的分区列包括表示日期或位置的列。

```SQL
CREATE TABLE site_access(
    event_day DATE,
    site_id INT DEFAULT '10',
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT SUM DEFAULT '0'
)
AGGREGATE KEY(event_day, site_id, city_code, user_name)
PARTITION BY RANGE(event_day)(
    PARTITION p1 VALUES LESS THAN ("2020-01-31"),
    PARTITION p2 VALUES LESS THAN ("2020-02-29"),
    PARTITION p3 VALUES LESS THAN ("2020-03-31")
)
DISTRIBUTED BY HASH(site_id);
```

##### 批量创建多个分区

可以在表创建后批量创建多个分区。您可以使用`START()`和`END()`指定批量创建的所有分区的开始和结束时间，以及`EVERY()`指定分区之间的增量值。但是需要注意的是，分区范围是右开区间，包括开始时间但不包括结束时间。分区的命名规则与动态分区相同。

- **在表创建时对日期类型列（DATE和DATETIME）进行分区**

  当分区列为日期类型时，在表创建时，您可以使用`START()`和`END()`来指定所有批量创建的分区的开始日期和结束日期，使用`EVERY(INTERVAL xxx)`指定两个分区之间的增量间隔。目前增量粒度支持`HOUR`（自v3.0起）、`DAY`、`WEEK`、`MONTH`和`YEAR`。

  在下面的示例中，所有批量创建的分区的日期范围从2021-01-01开始，结束于2021-01-04，并且增量间隔为一天：

    ```SQL
  CREATE TABLE site_access (
      datekey DATE,
      site_id INT,
      city_code SMALLINT,
      user_name VARCHAR(32),
      pv BIGINT DEFAULT '0'
  )
  ENGINE=olap
  DUPLICATE KEY(datekey, site_id, city_code, user_name)
  PARTITION BY RANGE (datekey) (
      START ("2021-01-01") END ("2021-01-04") EVERY (INTERVAL 1 DAY)
  )
  DISTRIBUTED BY HASH(site_id)
  PROPERTIES ("replication_num" = "3" );
    ```

  这相当于在CREATE TABLE语句中使用以下`PARTITION BY`子句：

    ```SQL
  PARTITION BY RANGE (datekey) (
  PARTITION p20210101 VALUES [('2021-01-01'), ('2021-01-02')),
  PARTITION p20210102 VALUES [('2021-01-02'), ('2021-01-03')),
  PARTITION p20210103 VALUES [('2021-01-03'), ('2021-01-04'))
  )
    ```

- **在表创建时对日期类型列（DATE和DATETIME）使用不同的日期间隔进行分区**

您可以通过在每个分区批次的“每个”中指定不同的增量间隔来创建不同增量间隔的日期分区批次（确保不同批次之间的分区范围不重叠）。每个批次中的分区根据“START（xxx）END（xxx）EVERY（xxx）”子句创建。例如：

```SQL
CREATE TABLE site_access(
    datekey DATE,
    site_id INT,
    city_code SMALLINT,
    user_name VARCHAR(32),
    pv BIGINT DEFAULT '0'
)
ENGINE=olap
DUPLICATE KEY(datekey, site_id, city_code, user_name)
PARTITION BY RANGE (datekey) 
(
    START ("2019-01-01") END ("2021-01-01") EVERY (INTERVAL 1 YEAR),
    START ("2021-01-01") END ("2021-05-01") EVERY (INTERVAL 1 MONTH),
    START ("2021-05-01") END ("2021-05-04") EVERY (INTERVAL 1 DAY)
)
DISTRIBUTED BY HASH(site_id)
PROPERTIES(
    "replication_num" = "3"
);
```

相当于在CREATE TABLE语句的`PARTITION BY`子句中使用以下内容：

```SQL
PARTITION BY RANGE (datekey) (
PARTITION p2019 VALUES [('2019-01-01'), ('2020-01-01')),
PARTITION p2020 VALUES [('2020-01-01'), ('2021-01-01')),
PARTITION p202101 VALUES [('2021-01-01'), ('2021-02-01')),
PARTITION p202102 VALUES [('2021-02-01'), ('2021-03-01')),
PARTITION p202103 VALUES [('2021-03-01'), ('2021-04-01')),
PARTITION p202104 VALUES [('2021-04-01'), ('2021-05-01')),
PARTITION p20210501 VALUES [('2021-05-01'), ('2021-05-02')),
PARTITION p20210502 VALUES [('2021-05-02'), ('2021-05-03')),
PARTITION p20210503 VALUES [('2021-05-03'), ('2021-05-04'))
)
```

- **在创建表时在整型列上进行分区**

  当分区列的数据类型为INT时，您可以在`START`和`END`中指定分区范围，并在`EVERY`中定义增量值。例如：

  > **注意**
  >
  > 在**START()**和**END()**中的分区列值需要用双引号括起来，而**EVERY()**中的增量值不需要用双引号括起来。

  在以下示例中，所有分区的范围从`1`开始到`5`结束，分区增量为`1`：

```SQL
CREATE TABLE site_access (
    datekey INT,
    site_id INT,
    city_code SMALLINT,
    user_name VARCHAR(32),
    pv BIGINT DEFAULT '0'
)
ENGINE=olap
DUPLICATE KEY(datekey, site_id, city_code, user_name)
PARTITION BY RANGE (datekey) (START ("1") END ("5") EVERY (1)
)
DISTRIBUTED BY HASH(site_id)
PROPERTIES ("replication_num" = "3");
```

相当于在CREATE TABLE语句的`PARTITION BY`子句中使用以下内容：

```SQL
PARTITION BY RANGE (datekey) (
PARTITION p2019 VALUES [('2019-01-01'), ('2020-01-01')),
PARTITION p2020 VALUES [('2020-01-01'), ('2021-01-01')),
PARTITION p202101 VALUES [('2021-01-01'), ('2021-02-01')),
PARTITION p202102 VALUES [('2021-02-01'), ('2021-03-01')),
PARTITION p202103 VALUES [('2021-03-01'), ('2021-04-01')),
PARTITION p202104 VALUES [('2021-04-01'), ('2021-05-01')),
PARTITION p20210501 VALUES [('2021-05-01'), ('2021-05-02')),
PARTITION p20210502 VALUES [('2021-05-02'), ('2021-05-03')),
PARTITION p20210503 VALUES [('2021-05-03'), ('2021-05-04'))
)
```

#### 在创建表后批量创建多个分区

在创建表后，您可以使用ALTER TABLE语句批量添加分区。语法类似于在创建表时批量创建多个分区，需要在`ADD PARTITIONS`子句中配置`START`、`END`和`EVERY`。

```SQL
ALTER TABLE site_access 
ADD PARTITIONS START ("2021-01-04") END ("2021-01-06") EVERY (INTERVAL 1 DAY);
```

#### 列分区（从v3.1开始）

[列表分区](./list_partitioning.md) 适用于加速基于枚举值的查询并有效管理数据。在需要将分区包含具有分区列中不同值的数据的情况下特别有用。例如，如果您经常根据国家和城市查询和管理数据，可以使用此分区方法，并选择“city”列作为分区列。在这种情况下，一个分区可以包含属于一个国家的各个城市的数据。

StarRocks根据每个分区的预定义值列表的显式映射存储数据。

### 管理分区

#### 添加分区

对于范围分区和列表分区，您可以手动添加新的分区以存储新的数据。但是对于表达式分区，因为分区在数据加载期间自动创建，所以不需要手动添加。

以下语句添加了一个新分区到表`site_access`以存储新月的数据：

```SQL
ALTER TABLE site_access
ADD PARTITION p4 VALUES LESS THAN ("2020-04-30")
DISTRIBUTED BY HASH(site_id);
```

#### 删除分区

以下语句从表`site_access`中删除了分区`p1`。

> **注意**
>
> 此操作不会立即删除分区中的数据。数据会在垃圾箱中保留一段时间（默认为一天）。如果意外删除了分区，可以使用[RECOVER](../sql-reference/sql-statements/data-definition/RECOVER.md)命令恢复分区及其数据。

```SQL
ALTER TABLE site_access
DROP PARTITION p1;
```

#### 恢复分区

以下语句将分区`p1`及其数据恢复到表`site_access`。

```SQL
RECOVER PARTITION p1 FROM site_access;
```

#### 查看分区

以下语句返回表`site_access`中所有分区的详细信息。

```SQL
SHOW PARTITIONS FROM site_access;
```

## 配置分桶

### 随机分桶（从v3.1开始）

StarRocks将某个分区中的数据随机分布到所有分桶中。适用于数据量较小且对查询性能要求相对较低的方案。如果没有设置分桶方法，StarRocks默认使用随机分桶，并自动设置桶的数量。

但是，请注意，如果查询大量数据并频繁使用某些列作为过滤条件，随机分桶提供的查询性能可能不佳。在这种情况下，建议使用 [hash分桶](#hash分桶)。当这些列用作查询的过滤条件时，只需扫描和计算查询命中的少量桶中的数据，可以显著提高查询性能。

#### 限制

- 只能使用随机分桶创建一个Duplicate Key表。
- 不能指定随机分桶表属于[共位组](../using_starrocks/Colocate_join.md)。
- 无法使用[Spark Load](../loading/SparkLoad.md)将数据加载到随机分桶表中。

在以下的CREATE TABLE示例中，未使用`DISTRIBUTED BY xxx`语句，因此StarRocks默认使用随机分桶，并自动设置桶的数量。

```SQL
CREATE TABLE site_access1(
    event_day DATE,
    site_id INT DEFAULT '10', 
    pv BIGINT DEFAULT '0' ,
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT ''
)
DUPLICATE KEY(event_day,site_id,pv);
```

但是，如果您熟悉StarRocks的分桶机制，也可以在创建随机分桶表时手动设置桶的数量。

```SQL
CREATE TABLE site_access2(
    event_day DATE,
    site_id INT DEFAULT '10', 
    pv BIGINT DEFAULT '0' ,
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT ''
)
DUPLICATE KEY(event_day,site_id,pv)
DISTRIBUTED BY RANDOM BUCKETS 8; -- 手动设置桶的数量为8
```

### Hash分桶

StarRocks可以使用哈希分桶将分区中的数据根据分桶键和[桶的数量](#determine-the-number-of-buckets)进行细分。在哈希分桶中，哈希函数以数据的分桶键值作为输入，并计算出一个哈希值。根据哈希值和桶之间的映射关系，数据被存储在相应的桶中。

#### 优势

- 提升查询性能：具有相同分桶键值的行被存储在同一个桶中，减少查询过程中扫描的数据量。

- 数据分布均匀：通过选择具有更高基数（即唯一值较多）的列作为分桶键，数据可以更均匀地分布在各个桶中。

#### 如何选择分桶列

我们建议您选择满足以下两个条件的列作为分桶列。

- 具有很高基数的列，如ID。
- 在查询中经常用作过滤条件的列。

但如果没有列同时满足这两个要求，您需要根据查询的复杂性确定分桶列。

- 如果查询复杂，建议选择具有较高基数的列作为分桶列，以确保数据尽可能均匀地分布在所有桶中，并提高集群资源利用率。
- 如果查询相对简单，建议选择在查询中经常用作过滤条件的列作为分桶列，以提高查询效率。

如果使用一个分桶列无法使分区数据均匀地分布在所有桶中，您可以选择多个分桶列。注意建议不要超过3列。

#### 注意事项

- **创建表时必须指定分桶列**。
- 分桶列的数据类型必须是INTEGER、DECIMAL、DATE/DATETIME或CHAR/VARCHAR/STRING。
- 一旦指定分桶列后，就无法修改分桶列。

#### 示例

在以下示例中，`site_access`表是通过使用`site_id`作为分桶列创建的。另外，当查询`site_access`表中的数据时，经常需要按站点进行数据过滤。使用`site_id`作为分桶键能够在查询过程中修剪掉大量与查询无关的桶。

```SQL
CREATE TABLE site_access(
    event_day DATE,
    site_id INT DEFAULT '10',
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT SUM DEFAULT '0'
)
AGGREGATE KEY(event_day, site_id, city_code, user_name)
PARTITION BY RANGE(event_day)
(
    PARTITION p1 VALUES LESS THAN ("2020-01-31"),
    PARTITION p2 VALUES LESS THAN ("2020-02-29"),
    PARTITION p3 VALUES LESS THAN ("2020-03-31")
)
DISTRIBUTED BY HASH(site_id);
```

假设`site_access`表的每个分区有10个桶。在以下查询中，10个桶中的9个被修剪掉，因此StarRocks只需要扫描`site_access`表中1/10的数据：

```SQL
select sum(pv)
from site_access
where site_id = 54321;
```

然而，如果`site_id`的分布不均匀，并且大量查询只请求少数站点的数据，使用单个分桶列可能导致严重的数据倾斜，从而导致系统性能瓶颈。在这种情况下，可以使用多个分桶列的组合。例如，以下语句使用`site_id`和`city_code`作为分桶列。

```SQL
CREATE TABLE site_access
(
    site_id INT DEFAULT '10',
    city_code SMALLINT,
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT SUM DEFAULT '0'
)
AGGREGATE KEY(site_id, city_code, user_name)
DISTRIBUTED BY HASH(site_id,city_code);
```

实际应用中，您可以根据业务特性使用一个或两个分桶列。使用单个分桶列`site_id`对于短查询是非常有利的，因为它能够减少节点之间的数据交换，从而提高集群的整体性能。另一方面，采用两个分桶列`site_id`和`city_code`对于长查询是有优势的，因为它可以利用分布式集群的整体并发性来显著提高性能。

> **注意**
>
> - 短查询涉及扫描少量数据，可以在单个节点上完成。
> - 长查询涉及扫描大量数据，通过在分布式集群中的多个节点上进行并行扫描，可以显著提高性能。

### 确定桶的数量

桶反映了StarRocks中数据文件的实际组织方式。

- 在创建表时如何设置桶的数量

  - 方法1：自动设置桶的数量（推荐）

    - 通过哈希分桶配置的表

      自v2.5.7以来，StarRocks可以根据机器资源和分区中的数据量自动设置桶的数量。

      > **注意**
      >
      > 如果分区的原始数据大小超过100GB，建议您使用方法2手动配置桶的数量。

      示例：

      ```sql
      CREATE TABLE site_access (
          site_id INT DEFAULT '10',
          city_code SMALLINT,
          user_name VARCHAR(32) DEFAULT '',
          pv BIGINT SUM DEFAULT '0')
      AGGREGATE KEY(site_id, city_code, user_name)
      DISTRIBUTED BY HASH(site_id,city_code); -- 无需设置桶的数量
      ```

    - 通过随机分桶配置的表

      自v2.5.7以来，StarRocks可以根据机器资源和分区中的数据量在表创建时自动设置桶的数量。从v3.2.0开始，StarRocks进一步优化了自动设置桶数量的逻辑。除了在表创建时设置桶的数量外，StarRocks还会根据集群的容量和加载的数据量**动态增加**分区中的桶数量。这不仅使分区创建变得更加便捷，还能提高批量加载的性能。

      > **注意**
      >
      > 默认桶的大小为`1024 * 1024 * 1024 B`（4 GB）。您可以在表创建时使用`PROPERTIES ("bucket_size"="xxx")`指定桶的大小。

      示例：

      ```SQL
      CREATE TABLE site_access1 (
          event_day DATE,
          site_id INT DEFAULT '10', 
          pv BIGINT DEFAULT '0' ,
          city_code VARCHAR(100),
          user_name VARCHAR(32) DEFAULT ''
      )
      DUPLICATE KEY (event_day,site_id,pv)
      ; -- 该表分区中的桶的数量将由StarRocks自动设置。默认桶的大小为4GB。这张表使用随机分桶配置，因此无需设置桶的键。
      
      CREATE TABLE site_access1 (
          event_day DATE,
          site_id INT DEFAULT '10', 
          pv BIGINT DEFAULT '0' ,
          city_code VARCHAR(100),
          user_name VARCHAR(32) DEFAULT '')
      DUPLICATE KEY (event_day,site_id,pv) 
      PROPERTIES("bucket_size"="1073741824") -- 桶的大小设置为1GB。
      ; -- 该表分区中的桶的数量将由StarRocks自动设置。这张表使用随机分桶配置，因此无需设置桶的键。
      ```

    创建表后，您可以执行[SHOW PARTITIONS](../sql-reference/sql-statements/data-manipulation/SHOW_PARTITIONS.md)命令查看StarRocks为每个分区设置的桶的数量。对于配置哈希分桶的表，每个分区的桶数量是**固定**的。

    对于配置随机分桶的表，每个分区的桶数量可以**动态更改**。因此，返回的结果会显示每个分区的**当前**桶的数量。

    > **注意**
    >
    > 对于此表类型，在一个分区内显示的实际层次结构如下：分区 > 子分区 > 桶。为了增加桶的数量，StarRocks实际上会添加一个包含一定数量桶的新的子分区。因此，`SHOW PARTITIONS`命令可能会返回具有相同分区名称的多个数据行，这些数据行显示了同一分区内子分区的信息。

  - 方法2：手动设置桶的数量

    自v2.4.0以来，StarRocks支持使用多个线程并行扫描查询中的单个Tablet，从而降低了扫描性能对Tablet数量的依赖。我们建议每个Tablet包含约10GB的原始数据。如果您打算手动设置桶的数量，您可以估计表中每个分区中的数据量，然后决定Tablet的数量。

    要在Tablet上启用并行扫描，确保全局系统中的`enable_tablet_internal_parallel`参数设置为`TRUE`（`SET GLOBAL enable_tablet_internal_parallel = true;`）。

    ```SQL
    CREATE TABLE site_access (
        site_id INT DEFAULT '10',
        city_code SMALLINT,
        user_name VARCHAR(32) DEFAULT '',
        pv BIGINT SUM DEFAULT '0')
    聚合键(site_id, city_code, user_name)
    -- 假设要加载到分区中的原始数据量为300 GB。
    -- 因为我们建议每个Tablet包含10 GB的原始数据，所以可以将桶的数量设置为30。
    HASH(site_id,city_code) 分发 30 个 BUCKETS;


    - 在添加新分区时如何设置桶的数量

      - 方法1：自动设置桶的数量（推荐）
        - 使用哈希桶分配的表

          自v2.5.7版本开始，StarRocks支持在分区创建期间根据机器资源和数据量自动设置桶的数量。
          > **注意**
          >
          > 如果分区的原始数据大小超过100 GB，我们建议您使用方法2手动配置桶的数量。

        - 使用随机桶分配的表

          自v2.5.7版本开始，StarRocks可以在分区创建期间根据机器资源和数据量自动设置桶的数量。从v3.2.0版本开始，StarRocks进一步优化了自动设置桶数量的逻辑。除了在分区创建期间设置桶的数量外，StarRocks还会根据群集容量和加载数据的量在数据加载期间**动态增加**分区中的桶的数量。这不仅使表的创建更加简单，还提高了批量加载性能。
          > **注意**
          >
          > 默认情况下，桶的大小是`1024 * 1024 * 1024 B`（4 GB）。在添加分区时，您可以在`PROPERTIES ("bucket_size"="xxx")`中指定桶的大小。

        添加分区后，您可以执行[SHOW PARTITIONS](../sql-reference/sql-statements/data-manipulation/SHOW_PARTITIONS.md)来查看StarRocks为新分区设置的桶的数量。对于使用哈希桶分配的表，新分区的桶数量是**固定的**。
        对于使用随机桶分配的表，新分区的桶的数量可以**动态改变**。因此，返回的结果显示新分区的**当前**桶的数量。
        > **注意**
        >
        > 对于此类型的表，分区内部实际的层次结构为：分区 > 子分区 > 桶。为了增加桶的数量，StarRocks实际上添加了一个新的子分区，其中包括一定数量的桶。因此，`SHOW PARTITIONS`语句可能返回多个具有相同分区名称的数据行，显示了同一分区中子分区的信息。

      - 方法2：手动设置桶的数量

        您还可以在添加新分区时手动指定桶的数量。要计算新分区的桶的数量，可以参考上面在创建表时手动设置桶数量的方法。

        ```SQL
        -- 手动创建分区
        ALTER TABLE <table_name> 
        ADD PARTITION <partition_name>
            [DISTRIBUTED BY HASH (k1[,k2 ...]) [BUCKETS num]];
        
        -- 动态分区
        ALTER TABLE <table_name> 
        SET ("dynamic_partition.buckets"="xxx");
        ```