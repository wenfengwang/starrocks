---
displayed_sidebar: English
---

# 数据分布

从 '@theme/Tabs' 导入 Tabs;
从 '@theme/TabItem' 导入 TabItem;

在创建表时配置适当的分区和存储桶可以帮助实现均匀的数据分布。均匀的数据分布意味着根据特定规则将数据划分为子集，并将其均匀分布在不同节点上。这样可以减少扫描的数据量，并充分利用集群的并行处理能力，从而提高查询性能。

> **注意**
>
> - 在建表时指定了数据分布，业务场景中的查询模式或数据特征会发生变化，从 v3.2 开始，StarRocks 支持 [在建表后修改某些数据分布相关的属性](#optimize-data-distribution-after-table-creation-since-32) ，以满足最新业务场景对查询性能的要求。
> - 从 v3.1 开始，在创建表或添加分区时，无需在 DISTRIBUTED BY 子句中指定 bucketing 键。StarRocks 支持随机分桶，即在所有桶中随机分配数据。有关更多信息，请参阅 [随机分桶](#random-bucketing-since-v31)。
> - 从 v2.5.7 开始，您可以在创建表或添加分区时选择不手动设置存储桶数量。StarRocks 可以自动设置桶数（BUCKETS）。但是，如果 StarRocks 自动设置桶数后性能没有达到您的预期，并且您熟悉桶数机制，您仍然可以 [手动设置桶数](#set-the-number-of-buckets)。

## 分发方法

### 一般的分发方法

现代分布式数据库系统通常使用以下基本分发方法：循环、范围、列表和哈希。

![数据分发方法](../assets/3.3.2-1.png)

- **循环**：在不同的节点之间循环分配数据。
- **范围**：根据分区列值的范围，将数据分布到不同的节点上。如图所示，范围 [1-3] 和 [4-6] 对应不同的节点。
- **列表**：根据分区列的离散值（如性别、省份等）将数据分布到不同的节点上。每个离散值都映射到一个节点，并且多个不同的值可能映射到同一节点。
- **哈希**：基于哈希函数将数据分布在不同的节点上。

为了实现更灵活的数据分区，除了使用上述数据分发方法之一外，还可以根据具体的业务需求组合这些方式。常见的组合包括 Hash+Hash、Range+Hash 和 Hash+List。

### StarRocks 中的分发方法

StarRocks 支持单独使用和复合使用两种数据分发方式。

> **注意**
>
> 除了通用的分配方式外，StarRocks 还支持 Random 分配，以简化 bucketing 配置。

此外，StarRocks 还通过实现两级分区 + 桶的方式分发数据。

- 第一级是分区：可以对表中的数据进行分区。支持的分区方法包括表达式分区、范围分区和列表分区。或者您可以选择不使用分区（整个表被视为一个分区）。
- 第二个级别是存储桶：分区中的数据需要进一步分发到较小的存储桶中。支持的分桶方法包括哈希分桶和随机分桶。

| **分配方式**   | **分区和分桶方法**                        | **描述**                                              |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 随机分布       | 随机分桶                                             | 整个表被视为一个分区。表中的数据随机分布到不同的存储桶中。这是默认的数据分发方法。 |
| 哈希分布         | 哈希桶                                               | 整个表被视为一个分区。表中的数据被分发到对应的桶中，该桶基于数据桶键的哈希值，使用哈希函数。 |
| 范围+随机分布 | <ol><li>表达式分区或范围分区 </li><li>随机分桶 </li></ol> | <ol><li>表中的数据将分发到相应的分区，该分区基于分区列值所在的范围。 </li><li>分区中的数据随机分布在不同的存储桶中。 </li></ol> |
| 范围+哈希分布   | <ol><li>表达式分区或范围分区</li><li>哈希分桶 </li></ol> | <ol><li>表中的数据将分发到相应的分区，该分区基于分区列值所在的范围。</li><li>分区中的数据被分发到对应的存储桶中，该存储桶基于数据的存储桶键的哈希值，使用哈希函数。 </li></ol> |
| 列表+随机分布  | <ol><li>表达式分区或列表分区</li><li>随机分桶 </li></ol> | <ol><li>表中的数据将分发到相应的分区，该分区基于分区列值所在的范围。</li><li>分区中的数据随机分布在不同的存储桶中。</li></ol> |
| 列表+哈希分布    | <ol><li>表达式分区或列表分区</li><li>哈希分桶 </li></ol> | <ol><li>表中的数据根据分区列值所属的值列表进行分区。</li><li>分区中的数据被分发到对应的存储桶中，该存储桶基于数据的存储桶键的哈希值，使用哈希函数。</li></ol> |

- **随机分布**

  如果在创建表时未配置分区和分桶方法，则默认使用随机分布。此分配方法目前只能用于创建 Duplicate Key 表。

  ```SQL
  CREATE TABLE site_access1 (
      event_day DATE,
      site_id INT DEFAULT '10', 
      pv BIGINT DEFAULT '0' ,
      city_code VARCHAR(100),
      user_name VARCHAR(32) DEFAULT ''
  )
  DUPLICATE KEY (event_day,site_id,pv);
  -- 因为未配置分区和分桶方法，默认使用随机分布。
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
  -- 使用哈希分桶作为分桶方法，并且必须指定分桶键。
  DISTRIBUTED BY HASH(event_day,site_id); 
  ```

- **范围+随机分布** （此分布方法目前只能用于创建重复键表。

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
  -- 因为未配置分桶方法，默认使用随机分桶。
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
  -- 使用哈希分桶作为分桶方法，并且必须指定分桶键。
  DISTRIBUTED BY HASH(event_day, site_id);
  ```

- **列表+随机分布** （此分布方法目前只能用于创建 Duplicate Key 表。

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
  -- 因为未配置分桶方法，默认使用随机分桶。
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
  -- 使用哈希分桶作为分桶方法，并且必须指定分桶键。
  DISTRIBUTED BY HASH(city,id); 
  ```

#### 分区

分区方法将一个表划分为多个分区。分区主要用于根据分区键将表拆分为不同的管理单元（分区）。您可以为每个分区设置存储策略，包括存储桶数量、冷热数据存储策略、存储介质类型、副本数等。StarRocks 允许您在一个集群中使用不同类型的存储介质。例如，您可以将最新数据存储在固态硬盘 （SSD） 上以提高查询性能，并将历史数据存储在 SATA 硬盘上以降低存储成本。

| **分区方式**                   | **场景**                                                    | **创建分区的方法**               |
| ------------------------------------- | ------------------------------------------------------------ | ------------------------------------------ |
| 表达式分区（推荐） | 以前称为自动分区。这种分区方法更加灵活和易于使用。它适用于大多数方案，包括基于连续日期范围或枚举值查询和管理数据。 | 在数据加载期间自动创建 |
| 范围分区                    | 典型方案是存储简单的有序数据，这些数据通常基于连续的日期/数字范围进行查询和管理。例如，在某些特殊情况下，历史数据需要按月分区，而最近数据需要按天分区。 | 手动、动态或批量创建 |

| 列表分区                     | 典型的场景是基于枚举值进行数据查询和管理，每个分区需要包含不同值的数据。例如，如果您经常根据国家和城市进行数据查询和管理，则可以使用此方法，并选择 `city` 作为分区列。因此，一个分区可以存储属于同一国家/地区的多个城市的数据。 | 手动创建                           |

##### 如何选择分区列和粒度

- 选择适当的分区列可以有效减少查询过程中扫描的数据量。在大多数业务系统中，通常采用基于时间的分区来解决过期数据删除导致的某些问题，并便于管理热数据和冷数据的存储。在这种情况下，您可以使用表达式分区或范围分区，并指定时间列作为分区列。此外，如果经常根据枚举值查询和管理数据，则可以使用表达式分区或列表分区，并指定包含这些值的列作为分区列。
- 在选择分区粒度时，需要考虑数据量、查询模式和数据管理粒度等因素。
  - 示例1：如果表中的月数据量较小，则与按天分区相比，按月分区可以减少元数据量，从而减少元数据管理和调度的资源消耗。
  - 示例2：如果表中每月的数据量较大，且查询多为请求特定日期的数据，则按天分区可以有效减少查询时扫描的数据量。
  - 示例3：如果数据需要按天过期，建议按天分区。

#### 分桶

分桶方法将一个分区划分为多个存储桶。存储桶中的数据称为平板电脑。

支持的分桶方法包括[随机分桶](#random-bucketing-since-v31)（从 v3.1 开始）和[哈希分桶](#hash-bucketing)。

- 随机分桶：在创建表或添加分区时，无需设置分桶键。分区内的数据会随机分布到不同的存储桶中。

- 哈希分桶：在创建表或添加分区时，需要指定分桶键。同一分区内的数据根据分桶键的值划分为存储桶，分桶键中具有相同值的行被分配到对应且唯一的存储桶中。

存储桶数量：StarRocks 默认自动设置存储桶数量（从 v2.5.7 开始）。您也可以手动设置存储桶的数量。有关详细信息，请参阅 [确定存储桶数量](#set-the-number-of-buckets)。

## 创建和管理分区

### 创建分区

#### 表达式分区（推荐）

> **注意**
>
> 从 v3.1 开始，StarRocks 的 [共享数据模式](../deployment/shared_data/s3.md) 支持时间函数表达式，不支持列表达式。

从 v3.0 开始，StarRocks 支持[表达式分区](./expression_partitioning.md)（以前称为自动分区），这种分区方法更加灵活易用。适用于大多数场景，例如基于连续日期范围或枚举值查询和管理数据。

您只需在创建表时配置分区表达式（时间函数表达式或列表表达式），StarRocks 会在数据加载时自动创建分区。您不再需要提前手动创建大量分区，也不再需要配置动态分区属性。

#### 范围分区

范围分区适用于存储简单、连续的数据，例如时间序列数据（日期或时间戳）或连续数值数据。而且，您经常根据连续的日期/数值范围查询和管理数据。此外，它可以应用于一些特殊情况，其中历史数据需要按月分区，最近数据需要按天分区。

StarRocks 会根据每个分区显式定义的范围的显式映射，将数据存储在对应的分区中。

##### 动态分区

[动态分区](./dynamic_partitioning.md) 相关属性在创建表时进行配置。StarRocks 会自动提前创建新分区，并移除过期分区以保证数据新鲜度，从而实现分区的生存时间 （TTL） 管理。

与表达式分区提供的自动创建分区能力不同，动态分区只能根据属性周期性地创建新分区。如果新数据不属于这些分区，则会为加载作业返回错误。但是，表达式分区提供的自动分区创建能力始终可以根据加载的数据创建相应的新分区。

##### 手动创建分区

使用正确的分区键可以有效减少查询期间扫描的数据量。目前，只能选择日期或整数类型的列作为分区列来组成分区键。在业务场景中，分区键通常是从数据管理角度选择的。常见的分区列包括表示日期或位置的列。

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

可以在创建表时和创建表后批量创建多个分区。您可以指定 batch in 中创建的所有分区的开始和结束时间 `START()` `END()`，以及 中的分区增量值`EVERY()`。但是，请注意，分区范围是右半开的，其中包括开始时间，但不包括结束时间。分区的命名规则与动态分区的命名规则相同。

- **在创建表时对日期类型列（DATE 和 DATETIME）上的表进行分区**

  当分区列为日期类型时，在创建表时，可以使用 `START()` and `END()` 指定批量创建的所有分区的开始日期和结束日期，并指定 `EVERY(INTERVAL xxx)` 两个分区之间的增量间隔。目前支持间隔粒度 `HOUR` （从 v3.0 开始）、`DAY`、 `WEEK` `MONTH` 、和 `YEAR`。

  在以下示例中，批量创建的所有分区的日期范围从 2021-01-01 开始，到 2021-01-04 结束，增量间隔为 1 天：

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

  它等效于 `PARTITION BY` 在 CREATE TABLE 语句中使用以下子句：

    ```SQL
  PARTITION BY RANGE (datekey) (
  PARTITION p20210101 VALUES [('2021-01-01'), ('2021-01-02')),
  PARTITION p20210102 VALUES [('2021-01-02'), ('2021-01-03')),
  PARTITION p20210103 VALUES [('2021-01-03'), ('2021-01-04'))
  )
    ```

- **在创建表时，使用不同的日期间隔对日期类型列（DATE 和 DATETIME）上的表进行分区**

  您可以通过为每批分区指定不同的增量间隔来创建具有不同增量间隔的日期`EVERY`分区批次（确保不同批次之间的分区范围不重叠）。每个批处理中的分区都是根据子句创建的 `START (xxx) END (xxx) EVERY (xxx)` 。例如：

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

  它等效于 `PARTITION BY` 在 CREATE TABLE 语句中使用以下子句：

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

- **在创建表时对整数类型列上的表进行分区**

  当分区列的数据类型为 INT 时，可以在 中指定分区范围 `START` `END`，并在 中定义增量值`EVERY`。例：

  > **注意**
  >
  > START（）  和 ** END（） 中的分区列**值需要用双引号括起来，而 EVERY（**）** 中的增量值不需要用双引号括起来**。**

  在以下示例中，所有分区的范围从  开始`1`并结束`5`于 ，分区增量为 `1`：

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

  它相当于在 CREATE TABLE 语句中使用以下 `PARTITION BY` 子句：

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

##### 创建表后批量添加多个分区

  创建表后，您可以使用 ALTER TABLE 语句添加分区。语法与在创建表时批量创建多个分区类似。您需要在 `ADD PARTITIONS` 子句中配置 `START`、 `END` 和 `EVERY`。

  ```SQL
  ALTER TABLE site_access 
  ADD PARTITIONS START ("2021-01-04") END ("2021-01-06") EVERY (INTERVAL 1 DAY);
  ```

#### 列表分区（自 v3.1 起）

[列表分区](./list_partitioning.md) 适用于加速查询和基于枚举值高效管理数据。它特别适用于需要在分区列中包含具有不同值的数据的场景。例如，如果您经常根据国家和城市查询和管理数据，则可以使用此分区方法，选择 `city` 列作为分区列。在这种情况下，一个分区可以包含属于一个国家/地区的各个城市的数据。

StarRocks 根据每个分区的预定义值列表的显式映射，将数据存储在相应的分区中。

### 管理分区

#### 添加分区

对于范围分区和列表分区，您可以手动添加新分区以存储新数据。但是，对于表达式分区，由于分区是在数据加载期间自动创建的，因此不需要手动添加。

以下语句将向 `site_access` 表添加一个新分区，以存储新月份的数据：

```SQL
ALTER TABLE site_access
ADD PARTITION p4 VALUES LESS THAN ("2020-04-30")
DISTRIBUTED BY HASH(site_id);
```

#### 删除分区

以下语句从 `site_access` 表中删除了 `p1` 分区。

> **注意**
>
> 此操作不会立即删除分区中的数据。数据会在废纸篓中保留一段时间（默认为一天）。如果错误地删除了分区，则可以使用 [RECOVER](../sql-reference/sql-statements/data-definition/RECOVER.md) 命令来恢复该分区及其数据。

```SQL
ALTER TABLE site_access
DROP PARTITION p1;
```

#### 恢复分区

以下语句将 `p1` 分区及其数据恢复到 `site_access` 表中。

```SQL
RECOVER PARTITION p1 FROM site_access;
```

#### 查看分区

以下语句返回了 `site_access` 表中所有分区的详细信息。

```SQL
SHOW PARTITIONS FROM site_access;
```

## 配置分桶

### 随机分桶（自 v3.1 起）

StarRocks 将分区中的数据随机分布到所有存储桶中。适用于数据量较小、对查询性能要求较低的场景。如果您不设置存储桶的方式，StarRocks 默认使用随机存储桶，并自动设置存储桶的数量。

但是，如果您查询的数据量很大，并且经常使用某些列作为筛选条件，则随机分桶提供的查询性能可能不是最优的。在这种情况下，建议使用 [哈希分桶](#hash-bucketing)。当这些列作为查询的筛选条件时，只需要扫描和计算查询命中的少量存储桶中的数据，这样可以显著提高查询性能。

#### 限制

- 您只能使用随机分桶来创建重复键表。
- 您不能将随机存储的表指定为属于 [主机托管组](../using_starrocks/Colocate_join.md)。
- [Spark Load](../loading/SparkLoad.md) 不能用于将数据加载到随机存储桶的表中。

在下面的 CREATE TABLE 示例中，`DISTRIBUTED BY xxx` 语句没有使用，因此 StarRocks 默认使用随机存储桶，并自动设置存储桶的数量。

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

不过，如果您熟悉 StarRocks 的存储桶机制，您也可以在创建随机存储桶表时手动设置存储桶的数量。

```SQL
CREATE TABLE site_access2(
    event_day DATE,
    site_id INT DEFAULT '10', 
    pv BIGINT DEFAULT '0' ,
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT ''
)
DUPLICATE KEY(event_day,site_id,pv)
DISTRIBUTED BY RANDOM BUCKETS 8; -- 手动设置存储桶的数量为 8
```

### 哈希分桶

StarRocks 可以使用哈希分桶，根据存储桶键和[存储桶的数量](#set-the-number-of-buckets)将分区中的数据细分为存储桶。在哈希分桶中，哈希函数将数据的存储桶键值作为输入，并计算哈希值。根据哈希值与存储桶的映射关系，数据存储在相应的存储桶中。

#### 优势

- 提高查询性能：具有相同存储桶键值的行存储在同一个存储桶中，从而减少查询期间扫描的数据量。

- 数据分布均匀：通过选择基数较高（唯一值较多）的列作为存储桶键，可以更均匀地分布在存储桶中。

#### 如何选择存储桶列

建议您选择满足以下两个条件的列作为存储桶列。

- 高基数列，例如 ID
- 经常在查询筛选器中使用的列

但是，如果没有列同时满足这两个要求，则需要根据查询的复杂度确定存储桶列。

- 如果查询较复杂，建议选择高基数列作为存储桶列，以确保数据尽可能均匀地分布在所有存储桶中，提高集群资源利用率。
- 如果查询比较简单，建议选择查询中经常用作文件库条件的列作为存储桶列，以提高查询效率。

如果使用一个存储桶列无法将分区数据均匀分布在所有存储桶中，则可以选择多个存储桶列。请注意，建议使用不超过 3 列。

#### 预防措施

- **创建表时，必须指定存储桶列**。
- 存储桶列的数据类型必须为 INTEGER、DECIMAL、DATE/DATETIME 或 CHAR/VARCHAR/STRING。
- 从 3.2 开始，可以在创建表后使用 ALTER TABLE 修改存储桶列。

#### 例子

在以下示例中，`site_access` 表是使用 `site_id` 作为存储桶列创建的。此外，在查询表中的数据时，`site_access` 通常会按站点筛选数据。使用`site_id`作为存储桶键可以在查询期间修剪大量不相关的存储桶。

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

假设表的每个分区 `site_access` 有 10 个存储桶。在下面的查询中，10 个存储桶中有 9 个被修剪，因此 StarRocks 只需要扫描表中 1/10 的数据 `site_access` ：

```SQL
select sum(pv)
from site_access
where site_id = 54321;
```

但是，如果 `site_id` 分布不均，大量查询只请求少数站点的数据，则仅使用一个存储桶列会导致严重的数据偏斜，从而导致系统性能瓶颈。在这种情况下，您可以使用存储桶列的组合。例如，以下语句使用 `site_id` 和 `city_code` 作为存储列。

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

实际上，您可以根据业务特点使用一到两个存储桶列。使用一个存储桶列 `site_id` 对于短查询非常有益，因为它可以减少节点之间的数据交换，从而提高集群的整体性能。另一方面，采用两个存储桶列， `site_id` `city_code` 对于长查询是有利的，因为它可以利用分布式集群的整体并发性来显著提高性能。

> **注意**
>
> - 短查询涉及扫描少量数据，可以在单个节点上完成。
> - 长查询涉及扫描大量数据，通过跨分布式集群中的多个节点并行扫描，可以显著提高其性能。

### 设置存储桶的数量

存储桶反映了数据文件在 StarRocks 中的实际组织方式。

#### 在创建表时

- 自动设置存储桶的数量（推荐）

  从 v2.5.7 开始，StarRocks 支持根据机器资源和数据量自动设置分区的存储桶数量。
  
  :::tip

  当分区的原始数据量超过100GB时，建议您通过方式2手动配置存储桶的数量。

  :::

  <Tabs groupId="automaticexamples1">

  <TabItem value="example1" label="使用哈希分桶配置的表" default>示例：

  ```sql
  CREATE TABLE site_access (
      site_id INT DEFAULT '10',
      city_code SMALLINT,
      user_name VARCHAR(32) DEFAULT '',
      event_day DATE,
      pv BIGINT SUM DEFAULT '0')
  AGGREGATE KEY(site_id, city_code, user_name,event_day)
  PARTITION BY date_trunc('day', event_day)
  DISTRIBUTED BY HASH(site_id,city_code); -- 无需设置桶数
  ```

  </TabItem>
  <TabItem value="example2" label="使用随机分桶配置的表">
  对于配置了随机分桶的表，StarRocks 从 v3.2.0 开始进一步优化了逻辑，除了自动设置分区中的桶数外，还可以**根据集群容量和加载数据量，在数据加载过程中动态增加**分区中的存储桶数量。

  :::警告

  - 要启用桶数的按需和动态增加，您需要设置表属性 `PROPERTIES("bucket_size"="xxx")` 来指定单个桶的大小。如果分区中的数据量较小，可以将 `bucket_size` 设置为 1 GB。否则，可以将 `bucket_size` 设置为 4 GB。
  - 如果启用了桶数的按需和动态增加，并且需要回滚到3.1版本，必须先删除支持动态增加桶数的表。然后，需要在[回滚之前使用](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) ALTER SYSTEM CREATE IMAGE 手动执行元数据检查点。

  :::

  示例：

  ```sql
  CREATE TABLE details1 (
      event_day DATE,
      site_id INT DEFAULT '10', 
      pv BIGINT DEFAULT '0',
      city_code VARCHAR(100),
      user_name VARCHAR(32) DEFAULT '')
  DUPLICATE KEY (event_day,site_id,pv)
  PARTITION BY date_trunc('day', event_day)
  -- 分区中的桶数由 StarRocks 自动确定，并且由于桶的大小设置为 1 GB，桶数会根据需求动态增加。
  PROPERTIES("bucket_size"="1073741824")
  ;
  
  CREATE TABLE details2 (
      event_day DATE,
      site_id INT DEFAULT '10',
      pv BIGINT DEFAULT '0' ,
      city_code VARCHAR(100),
      user_name VARCHAR(32) DEFAULT '')
  DUPLICATE KEY (event_day,site_id,pv)
  PARTITION BY date_trunc('day', event_day)
  -- 表分区中的桶数由 StarRocks 自动确定，且由于未设置桶的大小，桶数是固定的，不会根据需求动态增加。
  ;
  ```

  </TabItem>
  </Tabs>

- 手动设置桶数

  从 v2.4.0 开始，StarRocks 支持在查询过程中使用多个线程并行扫描 Tablet，从而减少扫描性能对 Tablet 数量的依赖。我们建议每个 Tablet 包含大约 10 GB 的原始数据。如果您打算手动设置桶数，可以估计表的每个分区中的数据量，然后确定 Tablet 的数量。

  要在 Tablet 上启用并行扫描，请确保全局设置整个系统的 `enable_tablet_internal_parallel` 参数为 `TRUE`（`SET GLOBAL enable_tablet_internal_parallel = true;`）。

  <Tabs groupId="manualexamples1">
  <TabItem value="example1" label="使用哈希分桶配置的表" default>

    ```SQL
    CREATE TABLE site_access (
        site_id INT DEFAULT '10',
        city_code SMALLINT,
        user_name VARCHAR(32) DEFAULT '',
        event_day DATE,
        pv BIGINT SUM DEFAULT '0')
    AGGREGATE KEY(site_id, city_code, user_name,event_day)
    PARTITION BY date_trunc('day', event_day)
    DISTRIBUTED BY HASH(site_id,city_code) BUCKETS 30;
    -- 假设要加载到分区中的原始数据量为 300 GB。
    -- 因为我们建议每个 Tablet 包含 10 GB 的原始数据，所以桶数可以设置为 30。
    DISTRIBUTED BY HASH(site_id,city_code) BUCKETS 30;
    ```
  
  </TabItem>
  <TabItem value="example2" label="使用随机分桶配置的表">

  ```sql
  CREATE TABLE details (
      site_id INT DEFAULT '10', 
      city_code VARCHAR(100),
      user_name VARCHAR(32) DEFAULT '',
      event_day DATE,
      pv BIGINT DEFAULT '0'
  )
  DUPLICATE KEY (site_id,city_code)
  PARTITION BY date_trunc('day', event_day)
  DISTRIBUTED BY RANDOM BUCKETS 30
  ; 
  ```

    </TabItem>
    </Tabs>

#### 创建表后

- 自动设置桶数（推荐）

  从 v2.5.7 开始，StarRocks 支持根据机器资源和数据量自动设置分区的桶数。

  :::提示
  
  当分区的原始数据量超过 100GB 时，建议您通过方式2手动配置 Bucket 数量。
  :::

  <Tabs groupId="automaticexamples2">
  <TabItem value="example1" label="使用哈希分桶配置的表" default>

  ```sql
  -- 为所有分区自动设置桶数。
  ALTER TABLE site_access DISTRIBUTED BY HASH(site_id,city_code);
  
  -- 为特定分区自动设置桶数。
  ALTER TABLE site_access PARTITIONS (p20230101, p20230102)
  DISTRIBUTED BY HASH(site_id,city_code);
  
  -- 为新分区自动设置桶数。
  ALTER TABLE site_access ADD PARTITION p20230106 VALUES [('2023-01-06'), ('2023-01-07'))
  DISTRIBUTED BY HASH(site_id,city_code);
  ```

  </TabItem>
  <TabItem value="example2" label="使用随机分桶配置的表">

  对于配置了随机分桶的表，StarRocks 从 v3.2.0 开始进一步优化了逻辑，除了自动设置分区中的桶数外，还可以**根据集群容量和加载数据量，在数据加载过程中动态增加**分区中的存储桶数量。这不仅使分区创建更容易，而且还提高了批量加载性能。

  :::警告

  - 要启用桶数的按需和动态增加，您需要设置表属性 `PROPERTIES("bucket_size"="xxx")` 来指定单个桶的大小。如果分区中的数据量较小，可以将 `bucket_size` 设置为 1 GB。否则，可以将 `bucket_size` 设置为 4 GB。
  - 如果启用了桶数的按需和动态增加，并且需要回滚到3.1版本，必须先删除支持动态增加桶数的表。然后，需要在[回滚之前使用](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) ALTER SYSTEM CREATE IMAGE 手动执行元数据检查点。

  :::

  ```sql
  -- 所有分区的桶数由 StarRocks 自动设置，并且由于禁用了桶数的按需和动态增加，这个数量是固定的。
  ALTER TABLE details DISTRIBUTED BY RANDOM;
  -- 所有分区的桶数由 StarRocks 自动设置，并且启用了桶数的按需和动态增加。
  ALTER TABLE details SET("bucket_size"="1073741824");
  
  -- 为特定分区自动设置桶数。
  ALTER TABLE details PARTITIONS (p20230103, p20230104)
  DISTRIBUTED BY RANDOM;
  
  -- 为新分区自动设置桶数。
  ALTER TABLE details ADD PARTITION  p20230106 VALUES [('2023-01-06'), ('2023-01-07'))
  DISTRIBUTED BY RANDOM;
  ```

  </TabItem>
  </Tabs>

- 手动设置桶数

  您也可以手动指定桶数。要计算分区的桶数，可以参考上述在创建表时手动设置桶数时使用的方法[](#at-table-creation)。

  <Tabs groupId="manualexamples2">
  <TabItem value="example1" label="使用哈希分桶配置的表" default>

  ```sql
  -- 为所有分区手动设置桶数
  ALTER TABLE site_access
  DISTRIBUTED BY HASH(site_id,city_code) BUCKETS 30;
  -- 为特定分区手动设置桶数。
  ALTER TABLE site_access
  partitions p20230104
  DISTRIBUTED BY HASH(site_id,city_code)  BUCKETS 30;
  -- 为新分区手动设置桶数。
  ALTER TABLE site_access
  ADD PARTITION p20230106 VALUES [('2023-01-06'), ('2023-01-07'))
  DISTRIBUTED BY HASH(site_id,city_code) BUCKETS 30;
  ```

  </TabItem>
  <TabItem value="example2" label="使用随机分桶配置的表">

  ```sql
  -- 为所有分区手动设置桶数
  ALTER TABLE details
  DISTRIBUTED BY RANDOM BUCKETS 30;
  -- 为特定分区手动设置桶数。
  ALTER TABLE details
  partitions p20230104
  DISTRIBUTED BY RANDOM BUCKETS 30;
  -- 为新分区手动设置桶数。
  ALTER TABLE details
  ADD PARTITION p20230106 VALUES [('2023-01-06'), ('2023-01-07'))
  DISTRIBUTED BY RANDOM BUCKETS 30;
  ```

  手动设置动态分区的默认桶数。

  ```sql
  ALTER TABLE details_dynamic
  SET ("dynamic_partition.buckets"="xxx");
  ```

  </TabItem>
  </Tabs>

#### 查看桶数

创建表后，您可以执行[SHOW PARTITIONS](../sql-reference/sql-statements/data-manipulation/SHOW_PARTITIONS.md)来查看 StarRocks 为每个分区设置的桶数。对于配置了哈希分桶的表，每个分区的桶数是固定的。

:::信息

- 对于配置了随机分桶的表，可以按需和动态增加桶的数量，每个分区中的桶数会动态增加。因此，返回的结果显示每个分区的当前存储桶数。
- 对于此表类型，分区内的实际层次结构如下：分区>子分区>存储桶。为了增加存储桶的数量，StarRocks 实际上增加了一个新的子分区，其中包含一定数量的存储桶。因此，SHOW PARTITIONS 语句可能会返回具有相同分区名称的多个数据行，这些数据行显示同一分区内子分区的信息。

:::

## 表创建后优化数据分发（自 3.2 版本起）

> **注意**
>
> StarRocks [共享数据模式](../deployment/shared_data/s3.md)目前不支持此功能。

随着业务场景的查询模式和数据量的演进，建表时指定的配置，如桶的方法、桶的数量和排序键，可能不再适合新的业务场景，甚至可能导致查询性能下降。此时，您可以使用`ALTER TABLE`来修改桶的方法、桶的数量和排序键来优化数据分发。例如：

- **当分区内的数据量显著增加时，增加存储桶数量**

  当分区内的数据量显著大于以前时，需要修改存储桶的数量，以便将平板电脑的大小通常维持在 1 GB 到 10 GB 的范围内。
- **修改分桶键以避免数据倾斜**

  当当前分桶键可能导致数据倾斜时（例如，仅将 `k1` 列配置为分桶键），就需要指定更合适的列或向分桶键添加额外的列。例如：
  
  ```SQL
  ALTER TABLE t DISTRIBUTED BY HASH(k1, k2) BUCKETS 20;
  -- 当 StarRocks 的版本为 3.1 或更高，并且表是重复键表时，可以考虑直接使用系统的默认分桶设置，即随机分桶和由 StarRocks 自动设置的分桶数。
  ALTER TABLE t DISTRIBUTED BY RANDOM;
  ```

- **根据查询模式变化调整排序键**
  
  如果业务查询模式发生显著变化，并且使用其他列作为条件列，调整排序键可能会有所帮助。例如：
  
  ```SQL
  ALTER TABLE t ORDER BY k2, k1;
  ```

有关更多信息，请参阅 [ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)。