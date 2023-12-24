---
displayed_sidebar: English
---

# 性能优化

## 表类型选择

StarRocks 支持四种表类型：重复键表、聚合表、唯一键表和主键表。所有这些表都按 KEY 排序。

- `AGGREGATE KEY`：当具有相同的 AGGREGATE KEY 的记录加载到 StarRocks 中时，新旧记录将被聚合。目前，聚合表支持以下聚合函数：SUM、MIN、MAX 和 REPLACE。聚合表支持提前汇总数据，有利于业务报表和多维分析。
- `DUPLICATE KEY`：对于 DUPLICATE KEY 表，您只需要指定排序键。具有相同 DUPLICATE KEY 的记录同时存在。它适用于不需要提前聚合数据的分析。
- `UNIQUE KEY`：当具有相同的 UNIQUE KEY 的记录加载到 StarRocks 中时，新记录将覆盖旧记录。唯一键表类似于具有 REPLACE 功能的聚合表。两者都适用于涉及不断更新的分析。
- `PRIMARY KEY`：主键表保证记录的唯一性，并允许您执行实时更新。

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

## 共置表

为了加快查询速度，具有相同分布的表可以使用共同的存储桶列。在这种情况下，数据可以在本地进行联接，而无需在 `join` 操作期间跨集群传输。

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

有关共置联接和副本管理的详细信息，请参阅[共置联接](../using_starrocks/Colocate_join.md)

## 平面表和星型架构

StarRocks 支持星型架构，比平面表在建模上更加灵活。您可以创建视图来替换平面表进行建模，然后从多个表中查询数据以加速查询。

平面表具有以下缺点：

- 尺寸更新成本高，因为平面表通常包含大量维度。每次更新维度时，都必须更新整个表。随着更新频率的增加，情况会变得更糟。
- 维护成本高，因为平面表需要额外的开发工作量、存储空间和数据回填操作。
- 数据引入成本高，因为平面表具有许多字段，而聚合表可能包含更多关键字段。在数据加载过程中，需要对更多字段进行排序，从而延长数据加载时间。

如果对查询并发性要求较高或延迟较低，仍然可以使用平面表。

## 分区和存储桶

StarRocks 支持两级分区：第一级是 RANGE 分区，第二级是 HASH 存储桶。

- RANGE分区：RANGE分区用于将数据划分为不同的区间（可以理解为将原始表划分为多个子表）。大多数用户选择按时间设置分区，具有以下优点：

  - 更容易区分热点数据和冷数据
  - 能够利用 StarRocks 分层存储（SSD + SATA）
  - 按分区删除数据的速度更快

- HASH存储桶：根据哈希值将数据划分为不同的存储桶。

  - 建议使用区分度高的列进行分桶，以避免数据倾斜。
  - 为了方便数据恢复，建议将每个存储桶中的压缩数据大小保持在 100 MB 到 1 GB 之间。建议您在创建表或添加分区时配置适当数量的存储桶。
  - 不建议随机分桶。创建表时，必须显式指定HASH存储桶列。

## 稀疏索引和布隆过滤器索引

StarRocks以有序方式存储数据，并以1024行的粒度构建稀疏索引。

StarRocks在schema中选择一个固定长度的前缀（当前为36字节）作为稀疏索引。

创建表时，建议将常用的筛选字段放在schema声明的开头。具有最高区分度和查询频率的字段必须放在前面。

VARCHAR字段必须放在稀疏索引的末尾，因为索引会从VARCHAR字段截断。如果VARCHAR字段出现在前面，索引可能小于36个字节。

以`site_visit`表为例。该表有四列：`siteid, city, username, pv`。排序键包含三列`siteid，city，username`，分别占用4、2和32个字节。因此，前缀索引（稀疏索引）可以是`siteid + city + username`的前30个字节。

除了稀疏索引，StarRocks还提供了布隆过滤器索引，可用于过滤具有高区分度的列。如果要将VARCHAR字段放在其他字段之前，可以创建布隆过滤器索引。

## 倒排索引

StarRocks采用位图索引技术，支持倒排索引，可应用于重复键表的所有列以及聚合表和唯一键表的key列。位图索引适用于值范围较小的列，如性别、城市和省份。随着范围的扩展，位图索引也会相应扩展。

## 具体化视图（汇总）

汇总实质上是原始表（基表）的具体化索引。创建汇总时，只能选择基表的部分列作为结构，并且结构中字段的顺序可以与基表不同。以下是使用汇总的一些用例：

- 基表中的数据聚合度不高，因为基表具有高区分度的字段。在这种情况下，您可以考虑选择一些列来创建汇总。以`site_visit`表为例：

  ~~~sql
  site_visit(siteid, city, username, pv)
  ~~~

  `siteid`可能会导致数据聚合不佳。如果需要频繁按城市计算PV，则可以使用以下方式创建汇总：`city`和`pv`。

  ~~~sql
  ALTER TABLE site_visit ADD ROLLUP rollup_city(city, pv);
  ~~~

- 无法命中基表中的前缀索引，因为基表的构建方式无法涵盖所有查询模式。在这种情况下，您可以考虑创建汇总来调整列顺序。以`session_data`表为例：

  ~~~sql
  session_data(visitorid, sessionid, visittime, city, province, ip, brower, url)
  ~~~

  如果需要分析除`visitorid`之外的访问情况，例如按`browser`和`province`进行分析，则可以创建单独的汇总：

  ~~~sql
  ALTER TABLE session_data
  ADD ROLLUP rollup_brower(brower,province,ip,url)
  DUPLICATE KEY(brower,province);
  ~~~

## 架构更改

在StarRocks中，更改schema有三种方式：排序schema更改、直接schema更改和链接schema更改。

- 排序架构更改：更改列的排序并重新排序数据。例如，删除排序架构中的列会导致数据重新排序。

  `ALTER TABLE site_visit DROP COLUMN city;`

- 直接架构更改：转换数据而不是重新排序数据，例如，更改列类型或将列添加到稀疏索引。

  `ALTER TABLE site_visit MODIFY COLUMN username varchar(64);`

- 链接架构更改：在不转换数据的情况下完成更改，例如添加列。

  `ALTER TABLE site_visit ADD COLUMN click bigint SUM default '0';`

  建议在创建表时选择合适的结构，以加速结构变更。
