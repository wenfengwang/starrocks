---
displayed_sidebar: English
---

# 性能优化

## 表类型选择

StarRocks 支持四种表类型：重复键表、聚合表、唯一键表和主键表。所有这些表都是按键进行排序的。

- 聚合键（AGGREGATE KEY）：当具有相同聚合键的记录被加载到 StarRocks 中时，新旧记录将进行聚合。目前，聚合表支持以下聚合函数：SUM、MIN、MAX 和 REPLACE。聚合表支持预先聚合数据，以方便业务报告和多维度分析。
- 重复键（DUPLICATE KEY）：你只需要为重复键表指定排序键。具有相同重复键的记录可以同时存在。它适用于不需要预先聚合数据的分析。
- 唯一键（UNIQUE KEY）：当具有相同唯一键的记录被加载到 StarRocks 中时，新记录会覆盖旧记录。唯一键表类似于带有 REPLACE 函数的聚合表。两者都适合涉及频繁更新的分析。
- 主键（PRIMARY KEY）：主键表保证了记录的唯一性，并允许你进行实时更新。

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

## 共定位表

为了加快查询速度，具有相同数据分布的表可以使用相同的分桶列。这样，在进行联接操作时，数据可以在本地完成联接，无需跨集群传输。

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

更多关于共定位联接和副本管理的信息，请参见[共定位联接](../using_starrocks/Colocate_join.md)。

## 平面表和星型模式

StarRocks 支持比平面表更灵活的星型模式。在建模时，你可以创建视图来替代平面表，然后从多个表查询数据以加速查询。

平面表有以下缺点：

- 由于平面表通常包含大量维度，维度更新的成本很高。每次更新维度时，都必须更新整个表。随着更新频率的增加，这种情况变得更加严重。
- 由于平面表需要额外的开发工作量、存储空间和数据回填操作，维护成本高。
- 由于平面表字段众多，聚合表可能包含更多的关键字段，数据摄入成本高。在数据加载过程中，需要排序的字段更多，这延长了数据加载时间。

如果你对查询并发性或低延迟有较高要求，仍可以使用平面表。

## 分区和桶

StarRocks 支持两级分区：第一级是范围分区（RANGE partition），第二级是哈希桶（HASH bucket）。

- 范围分区：范围分区用于将数据划分成不同的区间（可以理解为将原始表划分为多个子表）。大多数用户选择按时间进行分区，这样做有以下优势：

  - 更容易区分热数据和冷数据
  - 可以利用 StarRocks 的分层存储（SSD + SATA）
  - 可以更快地按分区删除数据

- 哈希桶：根据哈希值将数据划分到不同的桶中。

  - 建议使用区分度高的列进行分桶，以避免数据倾斜。
  - 为了方便数据恢复，建议每个桶中压缩数据的大小保持在 100 MB 到 1 GB 之间。我们建议在创建表或添加分区时配置适当数量的桶。
  - 不推荐使用随机分桶。在创建表时，你必须明确指定哈希分桶列。

## 稀疏索引和布隆过滤器索引

StarRocks 以有序方式存储数据，并以 1024 行为粒度构建稀疏索引。

StarRocks 在模式（schema）中选择一个固定长度的前缀（目前为 36 字节）作为稀疏索引。

在创建表时，建议将常用的过滤字段放在模式声明的开头。区分度最高和查询频率最高的字段应该放在最前面。

VARCHAR 字段必须放在稀疏索引的末尾，因为索引会从 VARCHAR 字段开始截断。如果 VARCHAR 字段出现在前面，则索引可能会小于 36 字节。

以 site_visit 表为例，该表有四列：siteid、city、username、pv。排序键包括三列：siteid、city、username，分别占用 4、2、32 字节。因此，前缀索引（稀疏索引）可以是 siteid + city + username 的前 30 字节。

除了稀疏索引，StarRocks 还提供布隆过滤器索引，这对于过滤高区分度的列非常有效。如果你想将 VARCHAR 字段放在其他字段之前，可以创建布隆过滤器索引。

## 倒排索引

StarRocks 采用位图索引技术支持倒排索引，适用于重复键表的所有列以及聚合表和唯一键表的键列。位图索引适用于值范围较小的列，例如性别、城市和省份。随着值范围的扩大，位图索引也会相应扩展。

## 物化视图（滚动升级）

滚动升级本质上是原始表（基表）的物化索引。在创建滚动升级时，只能选择基表的部分列作为模式，并且模式中字段的顺序可以与基表的不同。以下是使用滚动升级的一些用例：

- 如果基表的数据聚合度不高，因为基表中有高区分度的字段。在这种情况下，你可以考虑选择一些列来创建滚动升级。以 site_visit 表为例：

  ```sql
  site_visit(siteid, city, username, pv)
  ```

  siteid 可能导致数据聚合度低。如果你经常需要按城市计算 PV，可以创建只包含城市和 PV 的滚动升级。

  ```sql
  ALTER TABLE site_visit ADD ROLLUP rollup_city(city, pv);
  ```

- 如果基表的前缀索引无法命中，因为基表的构建方式无法覆盖所有查询模式。在这种情况下，你可以考虑创建滚动升级来调整列的顺序。以 session_data 表为例：

  ```sql
  session_data(visitorid, sessionid, visittime, city, province, ip, brower, url)
  ```

  如果除了访问者 ID 之外，你还需要按浏览器和省份分析访问情况，那么可以创建一个独立的滚动升级：

  ```sql
  ALTER TABLE session_data
  ADD ROLLUP rollup_brower(brower,province,ip,url)
  DUPLICATE KEY(brower,province);
  ```

## 架构变更

StarRocks 中有三种更改架构的方法：排序架构变更、直接架构变更和链接架构变更。

- 排序架构变更：更改列的排序并重新排序数据。例如，删除排序架构中的列会导致数据重新排序。

  ALTER TABLE site_visit DROP COLUMN city;

- 直接架构变更：转换数据而不是重新排序，例如，更改列类型或向稀疏索引中添加列。

  ALTER TABLE site_visit MODIFY COLUMN username VARCHAR(64);

- 链接架构变更：无需转换数据即可完成更改，例如添加列。

  ALTER TABLE site_visit ADD COLUMN click BIGINT SUM DEFAULT '0';

  建议在创建表时选择合适的架构，以加快架构变更的速度。
