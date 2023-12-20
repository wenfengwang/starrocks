---
displayed_sidebar: English
---

# 性能优化

## 表类型选择

StarRocks 支持四种表类型：Duplicate Key 表、Aggregate Key 表、Unique Key 表和 Primary Key 表。所有这些都按 KEY 排序。

- `AGGREGATE KEY`：当具有相同 AGGREGATE KEY 的记录被加载到 StarRocks 时，新旧记录将被合并。目前，Aggregate Key 表支持以下聚合函数：SUM、MIN、MAX 和 REPLACE。Aggregate Key 表支持预先聚合数据，便于业务报表和多维度分析。
- `DUPLICATE KEY`：你只需为 DUPLICATE KEY 表指定排序键。具有相同 DUPLICATE KEY 的记录可以同时存在。它适用于不需要预先聚合数据的分析。
- `UNIQUE KEY`：当具有相同 UNIQUE KEY 的记录被加载到 StarRocks 时，新记录会覆盖旧记录。UNIQUE KEY 表类似于具有 REPLACE 函数的 Aggregate Key 表。两者都适合涉及频繁更新的分析。
- `PRIMARY KEY`：Primary Key 表保证记录的唯一性，并允许你执行实时更新。

```sql
CREATE TABLE site_visit
(
    siteid      INT,
    city        SMALLINT,
    username    VARCHAR(32),
    pv          BIGINT   SUM DEFAULT '0'
)
AGGREGATE KEY(siteid, city, username)
DISTRIBUTED BY HASH(siteid);


CREATE TABLE session_data
(
    visitorid   SMALLINT,
    sessionid   BIGINT,
    visittime   DATETIME,
    city        CHAR(20),
    province    CHAR(20),
    ip          VARCHAR(32),
    browser     CHAR(20),
    url         VARCHAR(1024)
)
DUPLICATE KEY(visitorid, sessionid)
DISTRIBUTED BY HASH(sessionid, visitorid);

CREATE TABLE sales_order
(
    orderid     BIGINT,
    status      TINYINT,
    username    VARCHAR(32),
    amount      BIGINT DEFAULT '0'
)
UNIQUE KEY(orderid)
DISTRIBUTED BY HASH(orderid);

CREATE TABLE sales_order
(
    orderid     BIGINT,
    status      TINYINT,
    username    VARCHAR(32),
    amount      BIGINT DEFAULT '0'
)
PRIMARY KEY(orderid)
DISTRIBUTED BY HASH(orderid);
```

## Colocate 表

为了加快查询速度，具有相同分布的表可以使用公共的 bucket 列。这样，数据可以在本地进行 `join` 操作，而无需在集群间传输。

```sql
CREATE TABLE colocate_table
(
    visitorid   SMALLINT,
    sessionid   BIGINT,
    visittime   DATETIME,
    city        CHAR(20),
    province    CHAR(20),
    ip          VARCHAR(32),
    browser     CHAR(20),
    url         VARCHAR(1024)
)
DUPLICATE KEY(visitorid, sessionid)
DISTRIBUTED BY HASH(sessionid, visitorid)
PROPERTIES(
    "colocate_with" = "group1"
);
```

有关 Colocate Join 和副本管理的更多信息，请参阅[Colocate Join](../using_starrocks/Colocate_join.md)

## 平面表和星型模式

StarRocks 支持星型模式，相比平面表在建模上更加灵活。你可以在建模时创建视图来替代平面表，然后从多个表中查询数据以加速查询。

平面表有以下缺点：

- 维度更新成本高，因为平面表通常包含大量维度。每次更新维度时，都必须更新整个表。随着更新频率的增加，这种情况会变得更加严重。
- 维护成本高，因为平面表需要额外的开发工作量、存储空间和数据回填操作。
- 数据摄入成本高，因为平面表有许多字段，聚合表可能包含更多的键字段。在数据加载过程中，需要排序的字段更多，这会延长数据加载时间。

如果你对查询并发性或低延迟有较高要求，仍可以使用平面表。

## 分区和桶

StarRocks 支持两级分区：第一级是 RANGE 分区，第二级是 HASH 桶。

- `RANGE` 分区：RANGE 分区用于将数据划分为不同的区间（可以理解为将原表划分为多个子表）。大多数用户选择按时间设置分区，这有以下优点：

  - 更容易区分热数据和冷数据
  - 能够利用 StarRocks 的分层存储（SSD + SATA）
  - 通过分区更快地删除数据

- `HASH` 桶：根据哈希值将数据划分到不同的桶中。

  - 建议使用区分度高的列进行分桶，以避免数据倾斜。
  - 为了便于数据恢复，建议每个桶中的压缩数据大小保持在 100 MB 到 1 GB 之间。我们建议在创建表或添加分区时配置适当数量的桶。
  - 不建议随机分桶。创建表时必须显式指定 HASH 桶列。

## 稀疏索引和 BloomFilter 索引

StarRocks 以有序方式存储数据，并以 1024 行为粒度构建稀疏索引。

StarRocks 在 schema 中选择一个固定长度的前缀（当前为 36 字节）作为稀疏索引。

创建表时，建议将常用过滤字段放在 schema 声明的开头。区分度最高和查询频率最高的字段必须放在最前面。

VARCHAR 字段必须放在稀疏索引的末尾，因为索引会从 VARCHAR 字段开始截断。如果 VARCHAR 字段首先出现，则索引可能小于 36 字节。

以 `site_visit` 表为例。该表有四列：`siteid`、`city`、`username`、`pv`。排序键包含三列 `siteid`、`city`、`username`，分别占用 4、2、32 字节。因此，前缀索引（稀疏索引）可以是 `siteid + city + username` 的前 38 字节。

除了稀疏索引之外，StarRocks 还提供 BloomFilter 索引，对于区分度高的列过滤非常有效。如果你想将 VARCHAR 字段放在其他字段之前，你可以创建 BloomFilter 索引。

## 倒排索引

StarRocks 采用 Bitmap Indexing 技术支持倒排索引，适用于 Duplicate Key 表的所有列和 Aggregate Key 表及 Unique Key 表的键列。Bitmap Index 适用于值域较小的列，如性别、城市和省份。随着值域的扩大，Bitmap Index 也会相应扩大。

## 物化视图（Rollup）

Rollup 本质上是原始表（基表）的物化索引。创建 Rollup 时，只能选择基表的部分列作为 schema，并且 schema 中的字段顺序可以与基表不同。以下是使用 Rollup 的一些用例：

- 基表中的数据聚合度不高，因为基表中有区分度较高的字段。在这种情况下，你可以考虑选择一些列来创建 Rollup。以 `site_visit` 表为例：

  ```sql
  site_visit(siteid, city, username, pv)
  ```

  `siteid` 可能导致数据聚合度不佳。如果你需要频繁按城市计算 PV，可以创建只包含 `city` 和 `pv` 的 Rollup。

  ```sql
  ALTER TABLE site_visit ADD ROLLUP rollup_city(city, pv);
  ```

- 基表中的前缀索引无法命中，因为基表的构建方式无法覆盖所有查询模式。在这种情况下，你可以考虑创建 Rollup 来调整列顺序。以 `session_data` 表为例：

  ```sql
  session_data(visitorid, sessionid, visittime, city, province, ip, browser, url)
  ```

  如果除了 `visitorid` 之外，你还需要按 `browser` 和 `province` 分析访问情况，可以创建一个单独的 Rollup：

  ```sql
  ALTER TABLE session_data
  ADD ROLLUP rollup_browser(browser, province, ip, url)
  DUPLICATE KEY(browser, province);
  ```

## Schema 变更

StarRocks 中有三种更改 schema 的方法：排序 schema 变更、直接 schema 变更和链接 schema 变更。

- 排序 schema 变更：更改列的排序并重新排序数据。例如，删除排序 schema 中的列会导致数据重新排序。

  `ALTER TABLE site_visit DROP COLUMN city;`

- 直接 schema 变更：转换数据而不是重新排序，例如更改列类型或向稀疏索引添加列。

  `ALTER TABLE site_visit MODIFY COLUMN username VARCHAR(64);`

- 链接 schema 变更：无需转换数据即可完成更改，例如添加列。

  `ALTER TABLE site_visit ADD COLUMN click BIGINT SUM DEFAULT '0';`

  建议在创建表时选择合适的 schema，以加速 schema 变更。