```yaml
---
displayed_sidebar: "Chinese"

---

# 数据分布

在创建表时，您可以通过设置合理的分区和分桶，实现数据均匀分布和查询性能提升。数据均匀分布是指数据按照一定规则划分为子集，并且均衡地分布在不同节点上。这样查询时就可以有效减少数据扫描量，最大限度地利用集群的并发性能，从而提升查询性能。

> **说明**
>

> - 自 3.1 版本起，您在建表和新增分区时可以不设置分桶键（即 DISTRIBUTED BY 子句）。StarRocks 默认使用随机分桶，将数据随机地分布在分区的所有分桶中。更多信息，请参见[随机分桶](#随机分桶自-v31)。

> - 自 2.5.7 版本起，您在建表和新增分区时可以不设置分桶数量 (BUCKETS)。StarRocks 默认自动设置分桶数量，如果自动设置分桶数量后性能未能达到预期，并且您比较熟悉分桶机制，则您也可以[手动设置分桶数量](#确定分桶数量)。

## 数据分布概览

### 常见的数据分布方式

在现代分布式数据库中，常见的数据分布方式有如下几种：Round-Robin、Range、List 和 Hash。如下图所示：

![数据分布方式](../assets/3.3.2-1.png)

- Round-Robin：以轮询的方式把数据逐个放置在相邻节点上。

- Range：按区间进行数据分布。如上图所示，区间 [1-3]、[4-6] 分别对应不同的范围 (Range)。

- List：直接基于离散的各个取值做数据分布，性别、省份等数据就满足这种离散的特性。每个离散值会映射到一个节点上，多个不同的取值可能也会映射到相同节点上。

- Hash：通过哈希函数把数据映射到不同节点上。

为了更灵活地划分数据，除了单独采用上述数据分布方式之一以外，您还可以根据具体的业务场景需求组合使用这些数据分布方式。常见的组合方式有 Hash+Hash、Range+Hash、Hash+List。

### StarRocks 的数据分布方式


StarRocks 支持单独和组合使用数据分布方式。

> **说明**
>

> 除了常见的分布方式外， StarRocks 还支持了 Random 分布，可以简化分桶设置。

并且 StarRocks 通过设置分区 + 分桶的方式来实现数据分布。

- 第一层为分区：在一张表中，可以进行分区，支持的分区方式有表达式分区、Range 分区和 List 分区，或者不分区（即全表只有一个分区）。
- 第二层为分桶：在一个分区中，必须进行分桶。支持的分桶方式有哈希分桶和随机分桶。

| 数据分布方式      | 分区和分桶方式                                               | 说明                                                         |

| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |

| Random 分布       | 随机分桶                                                     | 一张表为一个分区，表中数据随机分布至不同分桶。该方式为默认数据分布方式。 |

| Hash 分布         | 哈希分桶                                                     | 一张表为一个分区，对表中数据的分桶键值使用哈希函数进行计算后，得出其哈希值，分布到对应分桶。 |
| Range+Random 分布 | <ol><li>表达式分区或者 Range 分区</li><li>随机分桶</li></ol> | <ol><li>表的数据根据分区列值所属范围，分布至对应分区。</li><li>同一分区的数据随机分布至不同分桶。</li></ol> |
| Range+Hash 分布   | <ol><li>表达式分区或者 Range 分区</li><li>哈希分桶</li></ol> | <ol><li>表的数据根据分区列值所属范围，分布至对应分区。</li><li>对同一分区的数据的分桶键值使用哈希函数进行计算，得出其哈希值，分布到对应分桶。</li></ol> |
| List+Random 分布  | <ol><li>表达式分区或者 List 分区</li><li>随机分桶</li></ol>  | <ol><li>表的数据根据分区列值所属枚举值列表，分布至对应分区。</li><li>同一分区的数据随机分布至不同分桶。</li></ol> |
| List+ Hash 分布   | <ol><li>表达式分区或者 List 分区</li><li>哈希分桶</li></ol>  | <ol><li>表的数据根据分区列值所属枚举值列表，分布至对应分区。</li><li>对同一分区的数据的分桶键值使用哈希函数进行计算，得出其哈希值，分布到对应分桶。</li></ol> |

- Random 分布

  建表时不设置分区和分桶方式，则默认使用 Random 分布

    ```SQL

    CREATE TABLE site_access1 (

        event_day DATE,
        site_id INT DEFAULT '10', 
        pv BIGINT DEFAULT '0' ,
        city_code VARCHAR(100),
        user_name VARCHAR(32) DEFAULT ''
    )
    DUPLICATE KEY (event_day,site_id,pv);
    -- 没有设置任何分区和分桶方式，默认为 Random 分布（目前仅支持明细模型的表）
    ```

- Hash 分布


    ```SQL

    CREATE TABLE site_access2 (
        event_day DATE,
        site_id INT DEFAULT '10',
        city_code SMALLINT,
        user_name VARCHAR(32) DEFAULT '',
        pv BIGINT SUM DEFAULT '0'
    )
    AGGREGATE KEY (event_day, site_id, city_code, user_name)
    -- 设置分桶方式为哈希分桶，并且必须指定分桶键
    DISTRIBUTED BY HASH(event_day,site_id); 
    ```

- Range + Random 分布

    ```SQL

    CREATE TABLE site_access3 (
        event_day DATE,
        site_id INT DEFAULT '10', 
        pv BIGINT DEFAULT '0' ,
        city_code VARCHAR(100),
        user_name VARCHAR(32) DEFAULT ''
    )
    DUPLICATE KEY(event_day,site_id,pv)
    -- 设为分区方式为表达式分区，并且使用时间函数的分区表达式（当然您也可以设置分区方式为 Range 分区）
    PARTITION BY date_trunc('day', event_day);
    -- 没有设置分桶方式，默认为随机分桶（目前仅支持明细模型的表）
    ```

- Range + Hash 分布

    ```SQL

    CREATE TABLE site_access4 (
    创建表 t_recharge_detail1 (
        id bigint,
        user_id bigint,
        recharge_money decimal(32,2), 
        city varchar(20) 不为空,
        dt date 不为空
    )
    DUPLICATE KEY(id)
    -- 设为分区方式为表达式分区，并且使用列分区表达式（当然您也可以设置分区方式为 List 分区）
    PARTITION BY (city);
    -- 没有设置分桶方式，默认为随机分桶（目前仅支持明细模型的表）
    ```

- List + Hash 分布

    ```SQL
    创建表 t_recharge_detail2 (
        id bigint,
        user_id bigint,
        recharge_money decimal(32,2), 
        city varchar(20) 不为空,
        dt date 不为空
    )
    DUPLICATE KEY(id)
    -- 设为分区方式为表达式分区，并且使用列分区表达式（当然您也可以设置分区方式为 List 分区）
    PARTITION BY (city)
    -- 设置分桶方式为哈希分桶，并且必须指定分桶键
    DISTRIBUTED BY HASH(city,id); 
    ```

#### 分区

分区用于将数据划分成不同的区间。分区的主要作用是将一张表按照分区键拆分成不同的管理单元，针对每一个管理单元选择相应的存储策略，比如分桶数、冷热策略、存储介质、副本数等。StarRocks 支持在一个集群内使用多种存储介质，您可以将新数据所在分区放在 SSD 盘上，利用 SSD 优秀的随机读写性能来提高查询性能，将旧数据存放在 SATA 盘上，以节省数据存储的成本。

| **分区方式**       | **适用场景**                                                     | **分区创建方式**                                  |
| ------------------ | ------------------------------------------------------------ | --------------------------------------------- |
| 表达式分区（推荐） | 原称自动创建分区，适用大多数场景，并且灵活易用。适用于按照连续日期范围或者枚举值来查询和管理数据。 | 导入时自动创建                                |
| Range 分区         | 典型的场景是数据简单有序，并且通常按照连续日期/数值范围来查询和管理数据。再如一些特殊场景，比如历史数据需要按月划分分区，而最近数据需要按天划分分区。 | 动态、批量或者手动创建 |
| List  分区         | 典型的场景是按照枚举值来查询和管理数据，并且一个分区中需要包含各分区列的多值。比如经常按照国家和城市来查询和管理数据，则可以使用该方式，选择分区列为 `city`，一个分区包含属于一个国家的多个城市的数据。 | 手动创建                              |

**选择分区列和分区粒度**

- 选择合理的分区列可以有效的裁剪查询数据时扫描的数据量。业务系统中⼀般会选择根据时间进行分区，以优化大量删除过期数据带来的性能问题，同时也方便冷热数据分级存储，此时可以使用时间列作为分区列进行表达式分区或者 Range 分区。此外，如果经常按照枚举值查询数据和管理数据，则可以选择枚举值的列作为分区列进行表达式分区或者 List 分区。
- 选择分区单位时需要综合考虑数据量、查询特点、数据管理粒度等因素。
  - 示例 1：表单月数据量很小，可以按月分区，相比于按天分区，可以减少元数据数量，从而减少元数据管理和调度的资源消耗。
  - 示例 2：表单月数据量很大，而大部分查询条件精确到天，如果按天分区，可以做有效的分区裁剪，减少查询扫描的数据量。
  - 示例 3：数据要求按天过期，可以按天分区。

#### 分桶

一个分区按分桶方式被分成了多个桶 bucket，每个桶的数据称之为一个 tablet。

分桶方式：StarRocks 支持[随机分桶](#随机分桶自-v31)（自 v3.1）和[哈希分桶](#哈希分桶)。

- 随机分桶，建表和新增分区时无需设置分桶键。在同一分区内，数据随机分布到不同的分桶中。
- 哈希分桶，建表和新增分区时需要指定分桶键。在同一分区内，数据按照分桶键划分分桶后，所有分桶键的值相同的行会唯一分配到对应的一个分桶。

分桶数量：默认由 StarRocks 自动设置分桶数量（自 v2.5.7）。同时也支持您手动设置分桶数量。更多信息，请参见[确定分桶数量](#确定分桶数量)。

## 创建和管理分区

### 创建分区

#### 表达式分区（推荐）

> **注意**
>
> 3.1 版本起，StarRocks [存算分离模式](../deployment/shared_data/s3.md)支持时间函数的分区表达式。

[表达式分区](./expression_partitioning.md)，原称自动创建分区，更加灵活易用，适用于大多数场景，比如按照连续日期范围或者枚举值来查询和管理数据。

您仅需要在建表时使用分区表达式（时间函数表达式或列表达式），即可实现导入数据时自动创建分区，不需要预先创建出分区或者配置动态分区属性。

#### Range 分区

Range 分区适用于简单且具有连续性的数据，如时间序列数据（日期或时间戳）或连续的数值数据。并且经常按照连续日期/数值范围，来查询和管理数据。以及一些特殊场景，比如一张表的分区粒度不一致，历史数据需要按月划分分区，而最近数据需要按天划分分区。

StarRocks 会根据您显式定义的范围与分区的映射关系将数据分配到相应的分区中。

**动态分区**

建表时[配置动态分区属性](./dynamic_partitioning.md)，StarRocks 会⾃动提前创建新的分区，删除过期分区，从而确保数据的时效性，实现对分区的⽣命周期管理（Time to Life，简称 “TTL”）。

区别于表达式分区中自动创建分区功能，动态创建分区只是根据您配置的动态分区属性，定期提前创建一些分区。如果导入的新数据不属于这些提前创建的分区，则导入任务会报错。而表达式分区中自动创建分区功能会根据导入数据创建对应的新分区。

**手动创建分区**

选择合理的分区键可以有效的裁剪扫描的数据量。**目前仅支持分区键的数据类型为日期和整数类型**。在实际业务场景中，一般从数据管理的角度选择分区键，常见的分区键为时间或者区域。

```SQL
CREATE TABLE site_access(
    event_day DATE,
    site_id INT DEFAULT '10',
    city_code VARCHAR(100),
```SQL
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

**批量创建分区**

建表时和建表后，支持批量创建分区，通过 START、END 指定批量分区的开始和结束，EVERY 子句指定分区增量值。其中，批量分区包含 START 的值，但是不包含 END 的值。分区的命名规则同动态分区一样。

- **建表时批量创建日期分区**

    当分区键为日期类型时，建表时通过 START、END 指定批量分区的开始日期和结束日期，EVERY 子句指定分区增量值。并且 EVERY 子句中用 INTERVAL 关键字表示日期间隔，目前支持日期间隔的单位为 HOUR（自 3.0 版本起）、DAY、WEEK、MONTH、YEAR。

    如下示例中，批量分区的开始日期为 `2021-01-01` 和结束日期为 `2021-01-04`，增量值为一天：

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
    PROPERTIES (
        "replication_num" = "3" 

    );

    ```

    则相当于在建表语句中使用如下 PARTITION BY 子句：

    ```SQL
    PARTITION BY RANGE (datekey) (
        PARTITION p20210101 VALUES [('2021-01-01'), ('2021-01-02')),

        PARTITION p20210102 VALUES [('2021-01-02'), ('2021-01-03')),

        PARTITION p20210103 VALUES [('2021-01-03'), ('2021-01-04'))

    )
    ```

- **建表时批量创建不同日期间隔的日期分区**

    建表时批量创建日期分区时，支持针对不同的日期分区区间（日期分区区间不能相重合），使用不同的 EVERY 子句指定日期间隔。一个日期分区区间，按照对应 EVERY 子句定义的日期间隔，批量创建分区，例如：

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
        START ("2019-01-01") END ("2021-01-01") EVERY (INTERVAL 1 YEAR),
        START ("2021-01-01") END ("2021-05-01") EVERY (INTERVAL 1 MONTH),

        START ("2021-05-01") END ("2021-05-04") EVERY (INTERVAL 1 DAY)

    )
    DISTRIBUTED BY HASH(site_id)
    PROPERTIES (
        "replication_num" = "3"
    );
    ```

    则相当于在建表语句中使用如下 PARTITION BY 子句：

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

- **建表时批量创建数字分区**

    当分区键为整数类型时，建表时通过 START、END 指定批量分区的开始值和结束值，EVERY 子句指定分区增量值。

    > **说明**
    >
    > START、END 所指定的分区列的值需要使用英文引号包裹，而 EVERY 子句中的分区增量值不用英文引号包裹。

    如下示例中，批量分区的开始值为 `1` 和结束值为 `5`，分区增量值为 `1`：

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
    PARTITION BY RANGE (datekey) (
        START ("1") END ("5") EVERY (1)
    )

    DISTRIBUTED BY HASH(site_id)

    PROPERTIES (

        "replication_num" = "3"
    );
    ```


    则相当于在建表语句中使用如下 PARTITION BY 子句：

    ```SQL

    PARTITION BY RANGE (datekey) (

        PARTITION p1 VALUES [("1"), ("2")),

        PARTITION p2 VALUES [("2"), ("3")),

        PARTITION p3 VALUES [("3"), ("4")),
        PARTITION p4 VALUES [("4"), ("5"))

    )

    ```

- **建表后批量创建分区**

    建表后，支持通过ALTER TABLE 语句批量创建分区。相关语法与建表时批量创建分区类似，通过指定 ADD PARTITIONS 关键字，以及 START、END 以及 EVERY 子句来批量创建分区。示例如下：

    ```SQL

    ALTER TABLE site_access 

> Explanation: Data in the partition will not be immediately deleted, but will be retained in the Trash for a period of time (default is one day). If a partition is mistakenly deleted, you can use the [RECOVER command](../sql-reference/sql-statements/data-definition/RECOVER.md) to restore the partition and its data.

```SQL
ALTER TABLE site_access
DROP PARTITION p1;
```

#### Restore Partition

Execute the following statement to restore the partition `p1` and its data in the `site_access` table:

```SQL
RECOVER PARTITION p1 FROM site_access;
```

#### View Partitions

Execute the following statement to view the partition information in the `site_access` table:

```SQL
SHOW PARTITIONS FROM site_access;
```

## Set Buckets

### Random Buckets (Since v3.1)

For the data in each partition, StarRocks distributes the data randomly into all buckets, which is suitable for scenarios with small data volume and low query performance requirements. If you do not set the bucketing method, StarRocks will use random buckets by default and automatically set the number of buckets.

However, it is worth noting that if a massive amount of data is queried and a series of columns are frequently used as conditional columns, the query performance provided by random buckets may not be ideal. In this scenario, it is recommended to use [Hash Buckets](#hash-buckets). When the columns are frequently used as conditional columns during queries, scanning and calculating a small number of buckets that the query hits can significantly improve query performance.

**Usage Restrictions**

- Only supports detailed model tables.
- Does not support specifying [Colocation Group](../using_starrocks/Colocate_join.md).
- Does not support [Spark Load](../loading/SparkLoad.md).

In the following table creation example, if the `DISTRIBUTED BY xxx` statement is not used, it means that StarRocks uses random buckets by default and automatically sets the number of buckets.

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

Of course, if you are familiar with the bucketing mechanism of StarRocks, when creating a table with random buckets, you can also manually set the number of buckets.

```SQL
CREATE TABLE site_access2(
    event_day DATE,
    site_id INT DEFAULT '10', 
    pv BIGINT DEFAULT '0' ,
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT ''
)
DUPLICATE KEY(event_day,site_id,pv)
DISTRIBUTED BY RANDOM BUCKETS 8; -- Manually set the number of buckets to 8
```

### Hash Buckets

For the data in each partition, StarRocks performs hash bucketing based on the bucketing key and [number of buckets](#determining-the-number-of-buckets). In hash bucketing, a specific column value is used as input to calculate a hash value through a hash function, and then the data is allocated to the corresponding bucket based on this hash value.

**Advantages**

- Improve query performance. Rows with the same bucketing key value are allocated to a bucket, reducing the amount of data scanned during queries.
- Evenly distribute data. By selecting columns with a higher cardinality (a large number of unique values) as the bucketing key, the data can be more evenly distributed to each bucket.

**How to Choose Bucketing Keys**

Assuming there is a column that satisfies both high cardinality and frequent use as a query condition, it is recommended to select it as the bucketing key for hash bucketing. If there is no column that satisfies both conditions, a judgment needs to be made based on the query.

- If the query is complex, it is recommended to select a column with high cardinality as the bucketing key to ensure that the data is distributed as evenly as possible among the buckets, thus improving the utilization of cluster resources.
- If the query is simple, it is recommended to select the column frequently used as a query condition as the bucketing key to improve query efficiency.

Additionally, if the data skew is severe, you can use multiple columns as the bucketing keys, but it is recommended not to exceed 3 columns.

**Notes**

- **When creating a table, if hash bucketing is used, the bucketing key must be specified**.
- The columns composing the bucketing key only support integer, DECIMAL, DATE/DATETIME, CHAR/VARCHAR/STRING data types.
- After the bucketing key is specified, it cannot be modified.

In the following example, the `site_access` table uses `site_id` as the bucketing key because `site_id` is a high cardinality column. In addition, for the query requests related to the `site_access` table, most of them use the site as a query filter condition, using `site_id` as the bucketing key can also trim a large number of irrelevant buckets during querying.

```SQL
CREATE TABLE site_access(
    event_day DATE,
    site_id INT DEFAULT '10',
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT SUM DEFAULT '0'
)
AGGREGATE KEY(event_day, site_id, city_code, user_name)
PARTITION BY RANGE(event_day) (
    PARTITION p1 VALUES LESS THAN ("2020-01-31"),
    PARTITION p2 VALUES LESS THAN ("2020-02-29"),
    PARTITION p3 VALUES LESS THAN ("2020-03-31")
)
DISTRIBUTED BY HASH(site_id);
```

In the following query, assuming each partition has 10 buckets, 9 buckets will be pruned, so the system only needs to scan 1/10 of the data in the `site_access` table:

```SQL
select sum(pv)
from site_access
where site_id = 54321;
```

However, if the distribution of `site_id` is extremely uneven, and a large amount of browsing data is about a small number of websites (power law distribution, 80-20 rule), using the above bucketing method will cause severe data skew, leading to local performance bottlenecks. In this case, you need to adjust the bucketing fields appropriately to disperse the data and avoid performance issues. For example, you can use the combination of `site_id` and `city_code` as the bucketing key to evenly distribute the data. The relevant table creation statement is as follows:

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

In actual use, you can choose one of the two bucketing methods based on the characteristics of your business. Using the bucketing method of `site_id` is very beneficial for short queries, reducing data exchange between nodes and improving the overall performance of the cluster. The combined bucketing method of `site_id` and `city_code` is beneficial for long queries, utilizing the overall concurrent performance of the distributed cluster.

> Note:
>
> - Short queries refer to queries that scan a small amount of data and can be completed by a single machine.
>
> - Long queries refer to queries that scan a large amount of data, and significant performance improvement can be achieved by parallel scanning on multiple machines.

- 哈希分桶表

      Since version 2.5.7, when creating a table, you don't need to manually set the number of buckets in the partition. StarRocks will automatically set the number of buckets in the partition based on machine resources and data volume.
      > **Note**
      >
      > If the original data scale of a single partition is expected to exceed 100 GB, it is recommended to manually set the number of buckets in the partition.

      Example of creating a table:

      ```sql
      CREATE TABLE site_access (
          site_id INT DEFAULT '10',
          city_code SMALLINT,
          user_name VARCHAR(32) DEFAULT '',
          pv BIGINT SUM DEFAULT '0')
      AGGREGATE KEY(site_id, city_code, user_name)
      DISTRIBUTED BY HASH(site_id,city_code); -- No need to manually set the number of buckets in the partition
      ```

    - Random bucket table

      Since version 2.5.7, when creating a table, you don't need to manually set the number of buckets in the partition. StarRocks will automatically set the number of buckets in the partition based on machine resources and data volume. Furthermore, since version 3.2, StarRocks has further optimized the logic for automatically setting the number of buckets, in addition to supporting automatic setting of the number of buckets in the partition when **creating partitions**, it also supports **dynamically increasing** the number of buckets in the partition **during the data import process** based on cluster capabilities and import data volume, etc., enhancing the usability of table creation while improving the import performance of large datasets.

      > **Note**
      >
      > The default size of a single bucket is `1024 * 1024 * 1024 B` (4 GB). When creating a table, you can specify the size of a single bucket in `PROPERTIES("bucket_size"="xxx")`.


      Example of creating a table:

      ```sql
      CREATE TABLE site_access1 (
          event_day DATE,
          site_id INT DEFAULT '10', 
          pv BIGINT DEFAULT '0' ,
          city_code VARCHAR(100),
          user_name VARCHAR(32) DEFAULT '')
      DUPLICATE KEY (event_day,site_id,pv)
      ; -- The number of buckets in the partition of this table is automatically set by StarRocks, and the default size of a single bucket is 4 GB. This table uses random buckets, and you do not need to set the bucket keys.

      CREATE TABLE site_access1 (
          event_day DATE,
          site_id INT DEFAULT '10', 
          pv BIGINT DEFAULT '0' ,
          city_code VARCHAR(100),
          user_name VARCHAR(32) DEFAULT '')
      DUPLICATE KEY (event_day,site_id,pv)
      PROPERTIES("bucket_size"="1073741824") -- The size of a single bucket is specified as 1 GB.
      ; -- The number of buckets in the partition of this table is automatically set by StarRocks. This table uses random buckets, and you do not need to set the bucket keys.

      ```

    After creating the table, you can execute [SHOW PARTITIONS](../sql-reference/sql-statements/data-manipulation/SHOW_PARTITIONS.md) to view the number of buckets set by StarRocks for the partitions. For a hash bucket table, the number of buckets in the partition after table creation is **fixed**.

    For a random bucket table, during the import process after table creation, the number of buckets in the partitions will be **dynamically increased**, and the returned results will show the **current** number of buckets in the partitions.
    > **Note**
    >
    > For random bucket tables, the actual division hierarchy within the partition is: partition > sub-partition > bucket. To increase the number of buckets, StarRocks actually adds a sub-partition, which includes a certain number of buckets, so the returned results of SHOW PARTITIONS will display multiple rows with the same partition name, indicating the situation of sub-partitions within the same partition.

  - Method 2: Manually set the number of buckets in the partition

    Since version 2.4, StarRocks has provided adaptive Tablet parallel scanning capability, which means that any Tablet involved in a query in one query may be parallel scanned by multiple threads, reducing the limitation of query capability on the number of Tablets. As a result, the setting of the number of buckets in the partition can be simplified. After simplification, the method to determine the number of buckets in the partition can be: first estimate the data volume of each partition, and then calculate the number of buckets in the partition based on every 10 GB original data for one Tablet.

    If you need to enable Tablet parallel scanning, you need to ensure that the system variable `enable_tablet_internal_parallel` takes effect globally `SET GLOBAL enable_tablet_internal_parallel = true;`.

    ```SQL
    CREATE TABLE site_access (
        site_id INT DEFAULT '10',
        city_code SMALLINT,
        user_name VARCHAR(32) DEFAULT '',
        pv BIGINT SUM DEFAULT '0')
    AGGREGATE KEY(site_id, city_code, user_name)
    DISTRIBUTED BY HASH(site_id,city_code) BUCKETS 30; -- Assuming the original data volume imported into a partition is 300 GB, the number of buckets in the partition can be set to 30 based on every 10 GB original data for one Tablet.
    ```

- How to set the number of buckets in the partition when adding a new partition

  - Method 1: Automatically set the number of buckets in the partition (recommended)

    - Hash bucket table

      Since version 2.5.7, when adding a new partition, you don't need to manually set the number of buckets in the partition. StarRocks will automatically set the number of buckets in the partition based on machine resources and data volume.
      > **Note**
      >
      > If the original data scale of a single partition is expected to exceed 100 GB, it is recommended to manually set the number of buckets in the partition.

    - Random bucket table

       Since version 2.5.7, when adding a new partition, you don't need to manually set the number of buckets in the partition. StarRocks will automatically set the number of buckets in the partition based on machine resources and data volume. Furthermore, since version 3.2, StarRocks has further optimized the logic for automatically setting the number of buckets, in addition to supporting automatic setting of the number of buckets in the partition **when creating a new partition**, it also supports **dynamically increasing** the number of buckets in the partition **during the data import process** based on cluster capabilities and import data volume, etc., enhancing the usability of adding new partitions while improving the import performance of large datasets.
       > **Note**
       >
       > The default size of a single bucket is `1024 * 1024 * 1024 B` (4 GB). When adding a new partition, you can specify the size of a single bucket in `PROPERTIES("bucket_size"="xxx")`.

    After adding a new partition, you can execute [SHOW PARTITIONS](../sql-reference/sql-statements/data-manipulation/SHOW_PARTITIONS.md) to view the number of buckets set by StarRocks for the new partition. For a hash bucket table, the number of buckets in the new partition is **fixed**.

    For a random bucket table, during the data import into the new partition, the number of buckets in the new partition will be **dynamically increased**, and the returned results will show the **current** number of buckets in the new partition.
    > **Note**
    >
    > For random bucket tables, the actual division hierarchy within the partition is: partition > sub-partition > bucket. To increase the number of buckets, StarRocks actually adds a sub-partition, which includes a certain number of buckets, so the returned results of SHOW PARTITIONS will display multiple rows with the same partition name, indicating the situation of sub-partitions within the same partition.

  - Method 2: Manually set the number of buckets in the partition

*注意*

当您新增分区时，也可以手动指定分桶数量。您可以参考上述手动设置分区中分桶数量的建表方式进行新增分区的分桶数量计算。

```SQL
-- 手动创建分区
ALTER TABLE <table_name> 
ADD PARTITION <partition_name>
    [DISTRIBUTED BY HASH (k1[,k2 ...]) [BUCKETS num]];
    
-- 手动设置动态分区的默认分桶数量
ALTER TABLE <table_name> 

SET ("dynamic_partition.buckets"="xxx");
```


## 最佳实践

对于 StarRocks 而言，分区和分桶的选择是非常关键的。建议在选择分区键和分桶键时，根据业务情况进行调整，这可以有效提高集群整体性能。

- **数据倾斜**
  
  如果业务场景中单独采用倾斜度大的列做分桶，很大程度会导致访问数据倾斜，那么建议采用多列组合的方式进行数据分桶。

- **高并发**
  
  分区和分桶应该尽量覆盖查询语句所带的条件，这样可以有效减少扫描数据，提高并发。

- **高吞吐**
  
  尽量把数据打散，让集群以更高的并发扫描数据，完成相应计算。

- **元数据管理**

  Tablet 过多会增加 FE/BE 的元数据管理和调度的资源消耗。