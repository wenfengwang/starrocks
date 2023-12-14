---
displayed_sidebar: "Chinese"
---

# Performance Optimization

This article introduces how to optimize the performance of StarRocks.

## Optimizing Performance through Table Creation

You can optimize the performance of StarRocks during table creation in the following ways.

### Choosing Data Models

StarRocks supports four types of data models: PRIMARY KEY, AGGREGATE KEY, UNIQUE KEY, and DUPLICATE KEY. Data in the four models are sorted based on the KEY.

* **AGGREGATE KEY Model**

    When AGGREGATE KEY is the same, the new and old records are aggregated. Currently supported aggregation functions include SUM, MIN, MAX, and REPLACE. The AGGREGATE KEY model can aggregate data in advance and is suitable for report and multidimensional analysis business.

    ```sql
    CREATE TABLE site_visit
    (
        siteid      INT,
        city        SMALLINT,
        username    VARCHAR(32),
        pv BIGINT   SUM DEFAULT '0'
    )
    AGGREGATE KEY(siteid, city, username)
    DISTRIBUTED BY HASH(siteid);
    ```

* **UNIQUE KEY Model**

    When UNIQUE KEY is the same, the new record overrides the old record. Currently, the implementation of UNIQUE KEY is the same as the REPLACE aggregation method of AGGREGATE KEY, and the two can be essentially considered the same. The UNIQUE KEY model is suitable for analytical business with updates.

    ```sql
    CREATE TABLE sales_order
    (
        orderid     BIGINT,
        status      TINYINT,
        username    VARCHAR(32),
        amount      BIGINT DEFAULT '0'
    )
    UNIQUE KEY(orderid)
    DISTRIBUTED BY HASH(orderid);
    ```

* **DUPLICATE KEY Model**

    DUPLICATE KEY is only used for sorting, and records with the same DUPLICATE KEY coexist. The DUPLICATE KEY model is suitable for analytical business that does not require advanced aggregation.

    ```sql
    CREATE TABLE session_data
    (
        visitorid   SMALLINT,
        sessionid   BIGINT,
        visittime   DATETIME,
        city        CHAR(20),
        province    CHAR(20),
        ip          varchar(32),
        brower      CHAR(20),
        url         VARCHAR(1024)
    )
    DUPLICATE KEY(visitorid, sessionid)
    DISTRIBUTED BY HASH(sessionid, visitorid);
    ```

* **PRIMARY KEY Model**

    The PRIMARY KEY model ensures that only one record exists under the same primary key. Compared to the update model, the primary key model does not require aggregation operations during queries and supports predicate and index pushdown, providing efficient queries while supporting real-time and frequent updates.

    ```sql
    CREATE TABLE orders (
        dt date NOT NULL,
        order_id bigint NOT NULL,
        user_id int NOT NULL,
        merchant_id int NOT NULL,
        good_id int NOT NULL,
        good_name string NOT NULL,
        price int NOT NULL,
        cnt int NOT NULL,
        revenue int NOT NULL,
        state tinyint NOT NULL
    )
    PRIMARY KEY (dt, order_id)
    DISTRIBUTED BY HASH(order_id);
    ```

### Using Colocate Tables

StarRocks supports storing related tables with the same distribution on common bucket columns, so that the JOIN operation of related tables can be performed locally, thereby accelerating queries. For more information, refer to [Colocate Join](../using_starrocks/Colocate_join.md).

```sql
CREATE TABLE colocate_table
(
    visitorid   SMALLINT,
    sessionid   BIGINT,
    visittime   DATETIME,
    city        CHAR(20),
    province    CHAR(20),
    ip          varchar(32),
    brower      CHAR(20),
    url         VARCHAR(1024)
)
DUPLICATE KEY(visitorid, sessionid)
DISTRIBUTED BY HASH(sessionid, visitorid)
PROPERTIES(
    "colocate_with" = "group1"
);
```

### Using Star Schema

StarRocks supports selecting a more flexible star schema to replace the traditional modeling of wide tables. With a star schema, you can use a view to model instead of a wide table, and directly use multiple table associations for queries. In the comparison of the standard test set of SSB, the performance of multi-table associations in StarRocks is not significantly lower than that of single-table queries.

Compared to a star schema, the shortcomings of a wide table include:

* Higher dimension update costs. In a wide table, updates to dimensional information will be reflected in the entire table, and the frequency of updates directly affects query efficiency.
* Higher maintenance costs. Building a wide table requires additional development work, storage space, and data backfill costs.
* Higher import costs. The number of fields in a wide table's schema is large, and there may be more Key columns in the aggregation model, which will increase the columns that need to be sorted during import, thereby increasing the import time.

It is recommended to prioritize the use of the star schema, which can ensure efficient metric analysis results while maintaining flexibility. However, if your business has higher requirements for high concurrency or low latency, you can still choose the wide table model for acceleration. StarRocks provides wide table query performance similar to ClickHouse.

### Using Partitioning and Bucketing

StarRocks supports two-level partition storage, with the first level being RANGE partition and the second level being HASH buckets.

RANGE partition is used to divide data into different intervals, which is equivalent to dividing the original table into multiple sub-tables logically. In a production environment, most users will partition based on time. Partitioning based on time has the following advantages:

* Can distinguish between hot and cold data.
* Can use StarRocks' tiered storage (SSD + SATA) feature.
* When deleting data by partition, it is faster.

HASH bucketing divides the data into different buckets based on the hash value.

* It is recommended to use columns with high distinctness for bucketing to avoid data skew.
* To facilitate data recovery, it is recommended to keep the size of a single bucket relatively small. The compressed size of the data in a single bucket should be kept at around 100MB to 1GB. It is recommended to reasonably consider the number of buckets when creating tables or adding partitions, with different partitions specifying different numbers of buckets.
* It is not recommended to use the Random bucketing method. When creating tables, please specify explicit Hash bucketing columns.

### Using Sparse Indexes and Bloom Filters

StarRocks supports storing data in an ordered manner and establishing sparse indexes based on the ordered data, with the index granularity being a Block (1024 rows).

Sparse indexes select a fixed length prefix in the Schema as the index content. Currently, StarRocks selects a 36-byte prefix as the index.

When creating tables, it is recommended to place commonly used filtering fields at the beginning of the Schema. Fields with higher distinctness and frequency of use should be placed at the beginning. VARCHAR type fields can only be the last field for a sparse index because the index will be truncated at the VARCHAR field. If VARCHAR data appears before other fields, the index length may be insufficient for 36 bytes.

For example, for the `site_visit` table mentioned above, its sorting columns include `siteid`, `city`, and `username`. Among them, `siteid` occupies 4 bytes, `city` occupies 2 bytes, and `username` occupies 32 bytes, so the content of the prefix index of this table is the first 30 bytes of `siteid` + `city` + `username`.

除稀疏索引之外，StarRocks 还提供了Bloomfilter索引。Bloomfilter索引在差异较大的列上具有明显的过滤效果。如果需要将VARCHAR字段放在前面，您可以创建Bloomfilter索引。

### 使用倒排索引

StarRocks支持倒排索引，使用位图技术构建索引（位图索引）。您可以将索引应用于DUPLICATE KEY数据模型的所有列，以及应用于AGGREGATE KEY和UNIQUE KEY模型的Key列。位图索引适合取值空间较小的列，例如性别、城市、省份等信息列。随着取值空间的增加，位图索引会同步膨胀。

### 使用物化视图

物化视图（Rollup）本质上可以理解为原始表（基表）的一个物化指标。在建立物化视图时，您可以选择基表中的部分列作为模式，且模式中的字段顺序可以与基表不同。下列情况可以考虑建立物化视图：

* 基表中数据的聚合度不高。通常是由于基表具有较大的区分度字段而导致。此时，您可以考虑选择部分列来建立物化视图。例如，对于上述的 `site_visit` 表，`siteid` 可能导致数据聚合度不高。如果经常需要根据城市统计 `pv`，可以建立仅含 `city`、`pv` 的物化视图。

    ```sql
    ALTER TABLE site_visit ADD ROLLUP rollup_city(city, pv);
    ```

* 基表中的前缀索引无法命中，通常是由于基表的建表方式无法覆盖所有的查询模式。此时，您可以考虑调整列的顺序来建立物化视图。例如，对于上述的 `session_data` 表，如果除了通过 `visitorid` 分析访问情况外，还有通过 `browser`、`province` 分析的情况，可以单独建立物化视图。

    ```sql
    ALTER TABLE session_data ADD ROLLUP rollup_browser(browser,province,ip,url) DUPLICATE KEY(browser,province);
    ```

## 优化导入性能

StarRocks目前提供了Broker Load和Stream Load两种导入方式，通过指定导入标签来标识一批次的导入。StarRocks对于单批次的导入会保证原子生效，即使单次导入多张表也同样保证其原子性。

* Stream Load：通过HTTP推送方式导入数据，用于微批导入。在该模式下，1MB数据的导入延迟可维持在秒级，适合高频导入。
* Broker Load：通过拉取方式批量导入数据，适合大批量数据的导入。

## 优化Schema Change性能

StarRocks目前支持三种Schema Change方式，即Sorted Schema Change，Direct Schema Change，Linked Schema Change。

* Sorted Schema Change：改变列的排序方式，需要对数据进行重新排序。例如删除排序列中的一个列，字段重新排序。

    ```sql
    ALTER TABLE site_visit DROP COLUMN city;
    ```

* Direct Schema Change：无需重新排序，但是需要对数据进行一次转换。例如修改列的类型，在稀疏索引中新增一列等。

    ```sql
    ALTER TABLE site_visit MODIFY COLUMN username varchar(64);
    ```

* Linked Schema Change：无需转换数据，直接完成。例如增加列操作。

    ```sql
    ALTER TABLE site_visit ADD COLUMN click bigint SUM default '0';
    ```

建议在建表时考虑好Schema，以加快后续Schema Change的速度。