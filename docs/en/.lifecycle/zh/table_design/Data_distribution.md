---
displayed_sidebar: English
---


```markdown
- 选择合适的分区列可以有效减少查询时扫描的数据量。在大多数业务系统中，通常采用基于时间的分区来解决过期数据删除带来的某些问题，并方便冷热数据的分层存储管理。此时，可以使用表达式分区或范围分区，并指定时间列作为分区列。另外，如果数据经常根据枚举值进行查询和管理，可以使用表达式分区或列表分区，并指定包含这些值的列作为分区列。
- 在选择分区粒度时，需要考虑数据量、查询模式和数据管理粒度等因素。
  - 示例1：如果表中每月的数据量较小，按月分区相比按天分区可以减少元数据量，从而减少元数据管理和调度的资源消耗。
  - 示例2：如果某个表每月数据量较大，且查询大多请求某天的数据，则按天分区可以有效减少查询时扫描的数据量。
  - 示例3：如果数据需要按天过期，建议按天分区。

#### 分桶

分桶方法将一个分区划分为多个桶。分桶中的数据称为 tablet。

支持的分桶方法是 [随机分桶](#random-bucketing-since-v31)（从 v3.1 开始）和 [哈希分桶](#hash-bucketing)。

- 随机分桶：创建表或添加分区时，不需要设置分桶键。分区内的数据随机分布到不同的桶中。

- 哈希分桶：创建表或添加分区时，需要指定分桶键。同一分区内的数据根据分桶键的值划分为桶，并且分桶键中具有相同值的行被分配到相应且唯一的桶中。

桶的数量：默认情况下，StarRocks 自动设置桶的数量（从 v2.5.7 开始）。你也可以手动设置桶的数量。更多信息，请参考[设置桶的数量](#set-the-number-of-buckets)。

## 创建和管理分区

### 创建分区

#### 表达式分区（推荐）

> **注意**
> 从 v3.1 开始，StarRocks 的[共享数据模式](../deployment/shared_data/s3.md)支持时间函数表达式，不支持列表达式。

从 v3.0 开始，StarRocks 支持[表达式分区](./expression_partitioning.md)（以前称为自动分区），它更加灵活和易于使用。这种分区方法适用于大多数场景，例如根据连续日期范围或枚举值查询和管理数据。

你只需要在创建表时配置一个分区表达式（时间函数表达式或列表达式），StarRocks 会在数据加载时自动创建分区。你不再需要提前手动创建大量分区，也不需要配置动态分区属性。

#### 范围分区

范围分区适合存储简单、连续的数据，如时间序列数据（日期或时间戳）或连续数值数据。你经常根据连续的日期/数值范围查询和管理数据。此外，它也适用于一些特殊情况，例如历史数据需要按月分区，而近期数据需要按天分区。

StarRocks 根据每个分区显式定义的范围将数据存储在相应的分区中。

##### 动态分区

[动态分区](./dynamic_partitioning.md)相关属性在创建表时配置。StarRocks 自动创建新分区并删除过期分区，确保数据的新鲜度，实现分区的生命周期管理。

与表达式分区提供的自动分区创建能力不同，动态分区只能基于属性定期创建新分区。如果新数据不属于这些分区，加载作业会返回错误。然而，表达式分区提供的自动分区创建能力总是可以根据加载的数据创建相应的新分区。

##### 手动创建分区

选择合适的分区键可以有效减少查询时扫描的数据量。目前，只能选择日期或整数类型的列作为分区列来构成分区键。在业务场景中，分区键通常是从数据管理的角度来选择的。常见的分区列包括表示日期或位置的列。

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

可以在创建表时和创建表后批量创建多个分区。你可以在 `START()` 和 `END()` 中指定批量创建的所有分区的开始和结束时间，并在 `EVERY()` 中指定分区增量值。但请注意，分区的范围是右半开的，包括开始时间但不包括结束时间。分区的命名规则与动态分区相同。

- **在创建表时根据日期类型列（DATE 和 DATETIME）对表进行分区**

  当分区列为日期类型时，在创建表时，你可以使用 `START()` 和 `END()` 指定批量创建的所有分区的开始日期和结束日期，并使用 `EVERY(INTERVAL xxx)` 指定两个分区之间的增量间隔。目前间隔粒度支持 `HOUR`（自 v3.0 起）、`DAY`、`WEEK`、`MONTH` 和 `YEAR`。

  在以下示例中，批量创建的所有分区的日期范围从 2021-01-01 开始，到 2021-01-04 结束，增量间隔为一天：

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
  PROPERTIES ("replication_num" = "3");
  ```

  它相当于在 CREATE TABLE 语句中使用以下 `PARTITION BY` 子句：

  ```SQL
  PARTITION BY RANGE (datekey) (
  PARTITION p20210101 VALUES [('2021-01-01'), ('2021-01-02')),
  PARTITION p20210102 VALUES [('2021-01-02'), ('2021-01-03')),
  PARTITION p20210103 VALUES [('2021-01-03'), ('2021-01-04'))
  )
  ```

- **在创建表时使用不同的日期间隔对日期类型列（DATE 和 DATETIME）进行分区**

  你可以通过为每批分区指定不同的增量间隔在 `EVERY` 中创建具有不同增量间隔的日期分区批次（确保不同批次之间的分区范围不重叠）。每批分区根据 `START(xxx) END(xxx) EVERY(xxx)` 子句创建。例如：

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

- **在创建表时根据整数类型列对表进行分区**

  当分区列的数据类型为 INT 时，你可以在 `START` 和 `END` 中指定分区范围，并在 `EVERY` 中定义增量值。例如：

    > **注意**
    > 在 **START()** 和 **END()** 中的分区列值需要用双引号括起来，而在 **EVERY()** 中的增量值不需要用双引号括起来。

  在以下示例中，所有分区的范围从 `1` 开始，到 `5` 结束，分区增量为 `1`：

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
  ```
```markdown
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

##### 批量创建多个分区

创建表后，您可以使用 `ALTER TABLE` 语句添加分区，语法与建表时批量创建多个分区类似。您需要在 `ADD PARTITIONS` 子句中配置 `START`、`END` 和 `EVERY`。

```SQL
ALTER TABLE site_access 
ADD PARTITIONS START ("2021-01-04") END ("2021-01-06") EVERY (INTERVAL 1 DAY);
```

#### 列表分区（自 v3.1 起）

[列表分区](./list_partitioning.md) 适合加速查询并高效管理基于枚举值的数据。对于需要在分区列中包含不同数值数据的场景，它尤其有用。例如，如果您经常根据国家和城市查询和管理数据，您可以使用这种分区方法，并选择 `city` 列作为分区列。在这种情况下，一个分区可以包含属于一个国家的各个城市的数据。

StarRocks 根据每个分区的预定义值列表的显式映射将数据存储在相应的分区中。

### 管理分区

#### 添加分区

对于范围分区和列表分区，您可以手动添加新分区来存储新数据。但是对于表达式分区，由于分区是在数据加载期间自动创建的，因此您不需要这样做。

以下语句向 `site_access` 表添加一个新分区以存储新月份的数据：

```SQL
ALTER TABLE site_access
ADD PARTITION p4 VALUES LESS THAN ("2020-04-30")
DISTRIBUTED BY HASH(site_id);
```

#### 删除分区

以下语句从 `site_access` 表中删除分区 `p1`。

> **注意**
> 此操作不会立即删除分区中的数据。数据会在回收站中保留一段时间（默认为一天）。如果分区被误删除，可以使用 [RECOVER](../sql-reference/sql-statements/data-definition/RECOVER.md) 命令恢复该分区及其数据。

```SQL
ALTER TABLE site_access
DROP PARTITION p1;
```

#### 恢复分区

以下语句将分区 `p1` 及其数据恢复到 `site_access` 表。

```SQL
RECOVER PARTITION p1 FROM site_access;
```

#### 查看分区

以下语句返回 `site_access` 表中所有分区的详细信息。

```SQL
SHOW PARTITIONS FROM site_access;
```

## 配置分桶

### 随机分桶（自 v3.1 起）

StarRocks 将分区中的数据随机分布到所有桶中。适用于数据量较小、对查询性能要求较低的场景。如果您不设置分桶方法，StarRocks 默认使用随机分桶并自动设置桶数量。

然而，需要注意的是，如果查询海量数据，并且频繁使用某些列作为过滤条件，随机分桶提供的查询性能可能不是最优的。在这种场景下，建议使用 [哈希分桶](#hash-bucketing)。当这些列作为过滤条件进行查询时，只需要扫描和计算查询命中的少量桶中的数据，可以显着提高查询性能。

#### 限制

- 您只能使用随机分桶来创建重复键表。
- 您不能指定随机分桶的表属于[Colocation Group](../using_starrocks/Colocate_join.md)。
- [Spark Load](../loading/SparkLoad.md) 不能用来加载数据到随机分桶的表中。

在下面的 `CREATE TABLE` 示例中，没有使用 `DISTRIBUTED BY xxx` 语句，因此 StarRocks 默认使用随机分桶，并自动设置桶数量。

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

不过，如果您熟悉 StarRocks 的分桶机制，您也可以在创建随机分桶表时手动设置桶数量。

```SQL
CREATE TABLE site_access2(
    event_day DATE,
    site_id INT DEFAULT '10', 
    pv BIGINT DEFAULT '0' ,
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT ''
)
DUPLICATE KEY(event_day,site_id,pv)
DISTRIBUTED BY RANDOM BUCKETS 8; -- 手动设置桶数量为 8
```

### 哈希分桶

StarRocks 可以使用哈希分桶，根据分桶键和[桶数量](#set-the-number-of-buckets)将分区中的数据细分为桶。在哈希分桶中，哈希函数将数据的分桶键值作为输入并计算哈希值。数据根据哈希值与桶的映射关系存储在相应的桶中。

#### 优点

- 提高查询性能：具有相同分桶键值的行存储在同一个桶中，减少查询时扫描的数据量。

- 数据分布均匀：通过选择基数较高（唯一值较多）的列作为分桶键，数据可以更均匀地分布在各个桶中。

#### 如何选择分桶列

我们建议您选择满足以下两个要求的列作为分桶列。

- 高基数列，例如 ID
- 查询中经常用作过滤条件的列

但如果没有列同时满足这两个要求，您需要根据查询的复杂程度来确定分桶列。

- 如果查询比较复杂，建议选择基数高的列作为分桶列，以保证数据尽可能均匀分布在所有桶中，提高集群资源利用率。
- 如果查询比较简单，建议选择查询中经常用作过滤条件的列作为分桶列，以提高查询效率。

如果使用一个分桶列无法将分区数据均匀分布到所有桶中，则可以选择多个分桶列。请注意，建议使用不超过 3 列。

#### 注意事项

- **创建表时，必须指定分桶列**。
- 分桶列的数据类型必须是 INTEGER、DECIMAL、DATE/DATETIME 或 CHAR/VARCHAR/STRING。
- 从 3.2 开始，可以在创建表后使用 `ALTER TABLE` 修改分桶列。

#### 示例

在以下示例中，`site_access` 表是使用 `site_id` 作为分桶列创建的。另外，在查询 `site_access` 表中的数据时，往往会按站点过滤数据。使用 `site_id` 作为分桶键可以在查询期间剪枝大量不相关的桶。

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

假设 `site_access` 表的每个分区有 10 个桶。在下面的查询中，10 个桶中有 9 个被剪枝，所以 StarRocks 只需要扫描 `site_access` 表中 1/10 的数据：

```SQL
select sum(pv)
from site_access
where site_id = 54321;
```

但如果 `site_id` 分布不均，大量查询只请求少数站点的数据，仅使用一个分桶列会导致严重的数据倾斜，造成系统性能瓶颈。在这种情况下，您可以使用分桶列的组合。例如，以下语句使用 `site_id` 和 `city_code` 作为分桶列。

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

实际上，您可以根据您的业务特点使用一到两个分桶列。使用一个分桶列 `site_id` 对于短查询非常有益，因为它减少了节点之间的数据交换，从而提高了集群的整体性能。另一方面，采用两个分桶列 `site_id` 和 `city_code` 对于长查询是有利的，因为它可以利用分布式集群的整体并发性来显着提高性能。

> **注意**
> - 短查询涉及扫描少量数据，可以在单个节点上完成。
> - 长查询涉及扫描大量数据，通过分布式集群中多个节点的并行扫描可以显着提高其性能。

### 设置桶数量

桶反映了数据文件在 StarRocks 中的实际组织方式。

#### 创建表时

- 自动设置桶数量（推荐）

  从 v2.5.7 开始，StarRocks 支持根据分区的机器资源和数据量自动设置桶的数量。

  :::tip

  如果分区的原始数据大小超过 100 GB，建议您使用方法 2 手动配置桶数量。

  :::

  <Tabs groupId="automaticexamples1">
  <TabItem value="example1" label="配置了哈希分桶的表" default>
  示例:

  ```sql
  CREATE TABLE site_access (
      site_id INT DEFAULT '10',
      city_code SMALLINT,
      user_name VARCHAR(32) DEFAULT '',
      event_day DATE,
```
```sql
CREATE TABLE site_access (
    site_id INT DEFAULT '10',
    city_code SMALLINT,
    user_name VARCHAR(32) DEFAULT '',
    event_day DATE,
    pv BIGINT SUM DEFAULT '0')
AGGREGATE KEY(site_id, city_code, user_name,event_day)
PARTITION BY date_trunc('day', event_day)
DISTRIBUTED BY HASH(site_id,city_code); -- 不需要设置桶的数量
```

对于配置了随机分桶的表，除了自动设置分区的桶数量外，StarRocks从v3.2.0开始进一步优化了逻辑。StarRocks还可以**在数据加载过程中动态增加**分区中的桶数，根据集群容量和加载数据量。这不仅使分区创建更容易，而且还提高了批量加载性能。

:::warning

- 为了实现桶数量的按需动态增加，需要设置表属性 `PROPERTIES("bucket_size"="xxx")` 来指定单个桶的大小。如果分区的数据量较小，可以将 `bucket_size` 设置为1GB。否则，您可以将 `bucket_size` 设置为4 GB。
- 一旦启用了按需动态增加桶数，需要回滚到3.1版本，则必须先删除启用了动态增加桶数的表。然后，您需要在回滚之前使用 [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) 手动执行元数据检查点。

:::

例子：

```sql
CREATE TABLE details1 (
    event_day DATE,
    site_id INT DEFAULT '10', 
    pv BIGINT DEFAULT '0',
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT '')
DUPLICATE KEY (event_day,site_id,pv)
PARTITION BY date_trunc('day', event_day)
-- 分区中的桶数由StarRocks自动确定，并且因为设置了桶的大小为1 GB，所以桶数会根据需求动态增加。
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
-- 表分区中的桶数由StarRocks自动确定，且数量固定，不会根据需求动态增加，因为没有设置桶的大小。
;
```

手动设置桶数

从v2.4.0开始，StarRocks支持在查询过程中使用多线程并行扫描tablet，从而减少扫描性能对tablet数量的依赖。我们建议每个tablet包含大约 10 GB 的原始数据。如果您打算手动设置桶的数量，您可以估算表的每个分区的数据量，然后决定桶的数量。

要在tablet上启用并行扫描，请确保整个系统的 `enable_tablet_internal_parallel` 参数全局设置为 `TRUE` (`SET GLOBAL enable_tablet_internal_parallel = true;`)。

```sql
CREATE TABLE site_access (
    site_id INT DEFAULT '10',
    city_code SMALLINT,
    user_name VARCHAR(32) DEFAULT '',
    event_day DATE,
    pv BIGINT SUM DEFAULT '0')
AGGREGATE KEY(site_id, city_code, user_name,event_day)
PARTITION BY date_trunc('day', event_day)
DISTRIBUTED BY HASH(site_id,city_code) BUCKETS 30;
-- 假设您想要加载到分区中的原始数据量为300 GB。
-- 因为我们建议每个tablet包含10 GB的原始数据，所以桶的数量可以设置为30。
DISTRIBUTED BY HASH(site_id,city_code) BUCKETS 30;
```

创建表后

自动设置桶数（推荐）

从v2.5.7开始，StarRocks支持根据分区的机器资源和数据量自动设置桶的数量。

:::tip

如果分区的原始数据大小超过100GB，建议您使用方法二手动配置桶数。

:::

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

对于配置了随机分桶的表，除了自动设置分区的桶数量外，StarRocks从v3.2.0开始进一步优化了逻辑。StarRocks还可以**在数据加载过程中动态增加**分区中的桶数，根据集群容量和加载数据量。这不仅使分区创建更容易，而且还提高了批量加载性能。

:::warning

- 为了实现桶数量的按需动态增加，需要设置表属性 `PROPERTIES("bucket_size"="xxx")` 来指定单个桶的大小。如果分区的数据量较小，可以将 `bucket_size` 设置为1GB。否则，您可以将 `bucket_size` 设置为4 GB。
- 一旦启用了按需动态增加桶数，需要回滚到3.1版本，则必须先删除启用了动态增加桶数的表。然后，您需要在回滚之前使用 [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) 手动执行元数据检查点。

:::

```sql
-- 所有分区的桶数由StarRocks自动设置，且数量固定，因为禁用了按需动态增加桶数的功能。
ALTER TABLE details DISTRIBUTED BY RANDOM;
-- 所有分区的桶数由StarRocks自动设置，并且启用了按需动态增加桶数的功能。
ALTER TABLE details SET("bucket_size"="1073741824");

-- 为特定分区自动设置桶数。
ALTER TABLE details PARTITIONS (p20230103, p20230104)
DISTRIBUTED BY RANDOM;

-- 为新分区自动设置桶数。
ALTER TABLE details ADD PARTITION  p20230106 VALUES [('2023-01-06'), ('2023-01-07'))
DISTRIBUTED BY RANDOM;
```

手动设置桶数

您也可以手动指定桶数量。计算分区的桶数可以参考上面建表时手动设置桶数的方法，[如上所述](#at-table-creation)。

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

手动设置动态分区的默认桶数。

```sql
ALTER TABLE details_dynamic
SET ("dynamic_partition.buckets"="xxx");
```

查看桶数

创建表后，可以执行 [SHOW PARTITIONS](../sql-reference/sql-statements/data-manipulation/SHOW_PARTITIONS.md) 查看StarRocks为每个分区设置的桶数。对于配置了哈希分桶的表，每个分区的桶数是固定的。

:::info

- 对于配置随机分桶、按需动态增加桶数的表，每个分区的桶数会动态增加。所以返回的结果显示了每个分区当前的桶数。
- 对于此表类型，分区内的实际层次结构如下：分区 > 子分区 > 桶。为了增加桶的数量，StarRocks实际上添加了一个新的子分区，其中包含一定数量的桶。因此，SHOW PARTITIONS 语句可能会返回多个具有相同分区名称的数据行，这些数据行显示同一分区内的子分区的信息。

:::

优化建表后的数据分布（3.2以后）

> **注意**
> StarRocks的 [共享数据模式](../deployment/shared_data/s3.md) 目前不支持此功能。

随着业务场景的查询模式和数据量的演变，建表时指定的配置，例如分桶方式、分桶数量、排序键等可能不再适合新的业务场景，甚至会影响查询性能减少。此时，您可以使用 `ALTER TABLE` 修改分桶方式、分桶数量以及排序键来优化数据分布。例如：

- **当分区内数据量显着增加时，增加桶的数量**

  当分区内的数据量明显大于之前时，需要修改桶的数量，以将tablet大小维持在1GB到10GB的范围内。

- **修改bucketing key以避免数据倾斜**

  当当前的bucketing key可能导致数据倾斜时（例如，只配置了`k1`列作为bucketing key），需要指定更合适的列或添加额外的列到bucketing key。例如：

  ```SQL
  ALTER TABLE t DISTRIBUTED BY HASH(k1, k2) BUCKETS 20;
  -- 当StarRocks的版本是3.1或更高，并且表是Duplicate Key表时，可以考虑直接使用系统的默认分桶设置，即随机分桶和StarRocks自动设置的桶数。
  ALTER TABLE t DISTRIBUTED BY RANDOM;
  ```

- **由于查询模式的变化，调整排序键**

  如果业务查询模式发生了显著变化，并且额外的列被用作条件列，调整排序键可能会有益。例如：

  ```SQL
  ALTER TABLE t ORDER BY k2, k1;
  ```

有关更多信息，请参阅 [ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)。
当当前的bucketing key可能导致数据倾斜时（例如，仅将`k1`列配置为bucketing key），需要指定更合适的列或向bucketing key添加额外的列。例如：

```SQL
ALTER TABLE t DISTRIBUTED BY HASH(k1, k2) BUCKETS 20;
-- 当StarRocks的版本是3.1或更高，并且表是Duplicate Key表时，可以考虑直接使用系统的默认bucketing设置，即随机bucketing以及由StarRocks自动设置的桶数量。
ALTER TABLE t DISTRIBUTED BY RANDOM;
```

- **由于查询模式的变化而调整排序键**

如果业务查询模式发生显著变化，并且使用额外的列作为条件列，则调整排序键可能会有所帮助。例如：

```SQL
ALTER TABLE t ORDER BY k2, k1;
```

有关详细信息，请参阅[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)。