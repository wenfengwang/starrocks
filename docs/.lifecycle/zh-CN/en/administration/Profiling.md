---
displayed_sidebar: "英文"
---

# 性能优化

## 表类型选择

StarRocks支持四种表类型：重复键表、聚集表、唯一键表和主键表。它们都是按键排序的。

- `聚集键`: 当具有相同聚集键的记录被加载到StarRocks时，旧记录和新记录会被聚集。目前，聚集表支持以下聚集函数：SUM、MIN、MAX和REPLACE。聚集表支持提前聚集数据，便于业务报表和多维分析。
- `重复键`: 只需为重复键表指定排序键。具有相同重复键的记录同时存在。适用于不涉及提前聚集数据的分析。
- `唯一键`: 当具有相同唯一键的记录被加载到StarRocks时，新记录将取代旧记录。唯一键表类似于具有REPLACE函数的聚集表。两者都适用于涉及常量更新的分析。
- `主键`: 主键表保证记录的唯一性，并允许您进行实时更新。

~~~sql
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
~~~

## 部署表

为了加快查询速度，具有相同分布的表可以使用共同的数据桶列。在这种情况下，在`join`操作期间，数据可以在本地加快加入而无需跨集群传输。

~~~sql
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
~~~

有关部署连接和副本管理的更多信息，请参阅[部署连接](../using_starrocks/Colocate_join.md)

## 平面表和星形模式

StarRocks支持星形模式，比平面表在建模方面更加灵活。您可以创建视图来代替平面表进行建模，然后从多个表中查询数据以加速查询。

平面表具有以下缺点：

- 昂贵的维度更新，因为平面表通常包含大量维度。每次更新维度时，必须更新整个表。随着更新频率的增加，情况将变得更加严重。
- 高维护成本，因为平面表需要额外的开发工作量、存储空间和数据回填操作。
- 高数据摄入成本，因为平面表具有许多字段，聚集表可能包含更多关键字段。在数据加载期间，需要对更多字段进行排序，这会延长数据加载时间。

如果您对查询并发性或低延迟具有高要求，您仍可以使用平面表。

## 分区和数据桶

StarRocks支持两级分区：第一级是范围分区，第二级是哈希数据桶。

- 范围分区：范围分区用于将数据划分为不同的区间（可以理解为将原始表划分为多个子表）。大多数用户选择按时间设置分区，这样做具有以下优点：

  - 更容易区分热数据和冷数据
  - 能够利用StarRocks分层存储（SSD + SATA）
  - 更快地通过分区删除数据

- 哈希数据桶: 根据哈希值将数据划分为不同的桶。

  - 建议使用具有较高区分度的列进行数据分桶，以避免数据倾斜。
  - 为了便于数据恢复，建议保持每个桶中压缩数据的大小在100 MB至1 GB之间。我们建议您在创建表或添加分区时配置适当数量的桶。
  - 不建议使用随机分桶。您必须在创建表时明确指定哈希分桶列。

## 稀疏索引和布隆过滤器索引

StarRocks以有序方式存储数据，并在1024行的粒度上构建稀疏索引。

StarRocks选择架构中固定长度的前缀（当前为36个字节）作为稀疏索引。

在创建表时，建议将常见的过滤字段放在架构声明的开头。具有最高区分度和查询频率的字段必须放在首位。

VARCHAR字段必须放在稀疏索引的末尾，因为索引从VARCHAR字段开始被截断。如果VARCHAR字段首先出现，索引可能小于36个字节。

以上`site_visit`表为例。该表有四列：`siteid, city, username, pv`。排序键包含三列 `siteid，city，username`，分别占据4、2和32个字节。因此，前缀索引（稀疏索引）可以是`siteid + city + username`的前30个字节。

除了稀疏索引，StarRocks还提供布隆过滤器索引，对于具有较高区分度的过滤列来说非常有效。如果您希望将VARCHAR字段放在其他字段之前，可以创建布隆过滤器索引。

## 倒排索引

StarRocks采用位图索引技术支持倒排索引，可应用于重复键表的所有列和聚集表和唯一键表的关键列。位图索引适用于值范围较小的列，例如性别、城市和省份。随着范围的扩大，位图索引会相应扩展。

## 材料化视图（rollup）

rollup本质上是原始表（基本表）的材料化索引。在创建rollup时，只能选择基本表的某些列作为架构，并且架构中的字段顺序可以与基本表不同。以下是使用rollup的一些用例：

- 基本表中数据聚合不高，因为基本表具有较高的区分度字段。在这种情况下，可以考虑选择一些列创建rollup。以上`site_visit`表为例：

  ~~~sql
  site_visit(siteid, city, username, pv)
  ~~~

  `siteid`可能导致数据聚合不佳。如果您需要频繁按城市计算PV，可以创建只包含`city`和`pv`的rollup。

  ~~~sql
  ALTER TABLE site_visit ADD ROLLUP rollup_city(city, pv);
  ~~~

- 基本表中的前缀索引无法命中，因为基本表的构建方式无法涵盖所有查询模式。在这种情况下，可以考虑创建rollup以调整列顺序。以上`session_data`表为例：

  ~~~sql
  session_data(visitorid, sessionid, visittime, city, province, ip, brower, url)
  ~~~

  如果有需要分析`browser`和`province`访问的情况，除了`visitorid`，您可以创建一个单独的rollup：

  ~~~sql
  ALTER TABLE session_data
  ADD ROLLUP rollup_brower(brower,province,ip,url)
  DUPLICATE KEY(brower,province);
  ~~~

## 架构变更

在StarRocks中，有三种方式可以变更架构：排序架构变更、直接架构变更和链接架构变更。

- 排序架构变更：改变列的排序并重新排序数据。例如，删除排序架构中的列会导致数据重新排序。

  `ALTER TABLE site_visit DROP COLUMN city;`

- 直接架构变更：改变数据而不是重新排序数据，例如改变列类型或将列添加到稀疏索引中。

  `ALTER TABLE site_visit MODIFY COLUMN username varchar(64);`

- 链接架构变更：无需转换数据即可完成更改，例如添加列。

  `ALTER TABLE site_visit ADD COLUMN click bigint SUM default '0';`

  建议在创建表时选择适当的架构以加速架构变更。