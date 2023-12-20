---
displayed_sidebar: English
---

# SSB 平面表基准测试

星型模式基准（SSB）旨在测试 OLAP 数据库产品的基本性能指标。SSB 使用了在学术界和工业界广泛应用的星型模式测试集。更多信息请参见论文 [Star Schema Benchmark](https://www.cs.umb.edu/~poneil/StarSchemaB.PDF)。
ClickHouse 将星型模式扁平化为宽平面表，并将 SSB 重写为单表基准测试。更多信息，请参阅 [ClickHouse 星型模式基准](https://clickhouse.tech/docs/en/getting-started/example-datasets/star-schema/)。本测试比较了 StarRocks、Apache Druid 和 ClickHouse 在 SSB 单表数据集上的性能。

## 测试结论

- 在 SSB 标准数据集上执行的 13 个查询中，StarRocks 的整体查询性能是 ClickHouse 的 **2.1 倍**，是 Apache Druid 的 **8.7 倍**。
- 启用 StarRocks 的 Bitmap Index 后，性能比禁用此功能时提高了 1.3 倍。StarRocks 的整体性能是 **ClickHouse 的 2.8 倍** 和 **Apache Druid 的 11.4 倍**。

![overall comparison](../assets/7.1-1.png)

## 测试准备

### 硬件

|机器|3 台云主机|
|---|---|
|CPU|16 核 Intel (R) Xeon (R) Platinum 8269CY CPU @2.50GHz <br />缓存大小：36608 KB|
|内存|64 GB|
|网络带宽|5 Gbit/s|
|磁盘|ESSD|

### 软件

StarRocks、Apache Druid 和 ClickHouse 部署在配置相同的主机上。

- StarRocks：一个 FE 和三个 BE。FE 可以单独部署，也可以与 BE 混合部署。
- ClickHouse：三个节点，使用分布式表
- Apache Druid：三个节点。一个部署了 Master Servers 和 Data Servers，一个部署了 Query Servers 和 Data Servers，第三个仅部署了 Data Servers。

内核版本：Linux 3.10.0-1160.59.1.el7.x86_64

操作系统版本：CentOS Linux release 7.9.2009

软件版本：StarRocks 社区版 3.0，ClickHouse 23.3，Apache Druid 25.0.0

## 测试数据及结果

### 测试数据

|表|记录数|描述|
|---|---|---|
|lineorder|6 亿|Lineorder 事实表|
|customer|300 万|Customer 维度表|
|part|140 万|Part 维度表|
|supplier|20 万|Supplier 维度表|
|dates|2,556|Date 维度表|
|lineorder_flat|6 亿|Lineorder 平面表|

### 测试结果

下表显示了 13 个查询的性能测试结果。查询延迟的单位是毫秒（ms）。表头中的 `ClickHouse vs StarRocks` 表示使用 ClickHouse 的查询响应时间除以 StarRocks 的查询响应时间。数值越大，表示 StarRocks 的性能越好。

|StarRocks-3.0|StarRocks-3.0-index|ClickHouse-23.3|ClickHouse vs StarRocks|Druid-25.0.0|Druid vs StarRocks|
|---|---|---|---|---|---|
|Q1.1|33|30|48|1.45|430|13.03|
|Q1.2|10|10|15|1.50|270|27.00|
|Q1.3|23|30|14|0.61|820|35.65|
|Q2.1|186|116|301|1.62|760|4.09|
|Q2.2|156|50|273|1.75|920|5.90|
|Q2.3|73|36|255|3.49|910|12.47|
|Q3.1|173|233|398|2.30|1080|6.24|
|Q3.2|120|80|319|2.66|850|7.08|
|Q3.3|123|30|227|1.85|890|7.24|
|Q3.4|13|16|18|1.38|750|57.69|
|Q4.1|203|196|469|2.31|1230|6.06|
|Q4.2|73|76|160|2.19|1020|13.97|
|Q4.3|50|36|148|2.96|820|16.40|
|总计|1236|939|2645|2.14|10750|8.70|

## 测试流程

有关如何创建 ClickHouse 表和向表中加载数据的更多信息，请参见 [ClickHouse 官方文档](https://clickhouse.tech/docs/en/getting-started/example-datasets/star-schema/)。以下部分描述了 StarRocks 的数据生成和数据加载过程。

### 生成数据

下载 ssb-poc 工具包并编译。

```Bash
wget https://starrocks-public.oss-cn-zhangjiakou.aliyuncs.com/ssb-poc-1.0.zip
unzip ssb-poc-1.0.zip
cd ssb-poc-1.0/
make && make install
cd output/
```

编译完成后，所有相关工具都会被安装到 `output` 目录下，接下来的操作都在该目录下进行。

首先，为 SSB 标准数据集生成比例因子为 100 的数据。

```Bash
sh bin/gen-ssb.sh 100 data_dir
```

### 创建表结构

1. 修改配置文件 `conf/starrocks.conf`，指定集群地址。特别注意 `mysql_host` 和 `mysql_port`。

2. 运行以下命令来创建表：

   ```SQL
   sh bin/create_db_table.sh ddl_100
   ```

### 查询数据

```Bash
sh bin/benchmark.sh ssb-flat
```

### 启用 Bitmap 索引

启用 Bitmap 索引后，StarRocks 的性能会得到提升。如果您想测试启用 Bitmap 索引的 StarRocks 性能，特别是在 Q2.2、Q2.3 和 Q3.3 上，您可以为所有 STRING 类型的列创建 Bitmap 索引。

1. 创建另一个 `lineorder_flat` 表并为其创建 Bitmap 索引。

   ```SQL
   sh bin/create_db_table.sh ddl_100_bitmap_index
   ```

2. 在所有 BE 节点的 `be.conf` 文件中添加以下配置，并重启 BE 以使配置生效。

   ```SQL
   bitmap_max_filter_ratio=1000
   ```

3. 运行数据加载脚本。

   ```SQL
   sh bin/flat_insert.sh data_dir
   ```

数据加载完成后，等待数据版本压缩完成，然后执行 [4.4](#query-data) 再次查询数据。

您可以通过运行 `select CANDIDATES_NUM from information_schema.be_compactions` 来查看数据版本压缩的进度。对于三个 BE 节点，以下结果显示压缩已完成：

```SQL
mysql> select CANDIDATES_NUM from information_schema.be_compactions;
+----------------+
| CANDIDATES_NUM |
+----------------+
|              0 |
|              0 |
|              0 |
+----------------+
3 行返回 (0.01 秒)
```

## 测试 SQL 和建表语句

### 测试 SQL

```SQL
--Q1.1 
SELECT sum(lo_extendedprice * lo_discount) AS `revenue` 
FROM lineorder_flat 
WHERE lo_orderdate >= '1993-01-01' AND lo_orderdate <= '1993-12-31'
AND lo_discount BETWEEN 1 AND 3 AND lo_quantity < 25; 
 
--Q1.2 
SELECT sum(lo_extendedprice * lo_discount) AS revenue FROM lineorder_flat  
WHERE lo_orderdate >= '1994-01-01' AND lo_orderdate <= '1994-01-31'
AND lo_discount BETWEEN 4 AND 6 AND lo_quantity BETWEEN 26 AND 35; 
 
--Q1.3 
SELECT sum(lo_extendedprice * lo_discount) AS revenue 
FROM lineorder_flat 
WHERE weekofyear(lo_orderdate) = 6
AND lo_orderdate >= '1994-01-01' AND lo_orderdate <= '1994-12-31' 
AND lo_discount BETWEEN 5 AND 7 AND lo_quantity BETWEEN 26 AND 35; 
 
--Q2.1 
SELECT sum(lo_revenue), year(lo_orderdate) AS year, p_brand 
FROM lineorder_flat 
WHERE p_category = 'MFGR#12' AND s_region = 'AMERICA' 
GROUP BY year, p_brand 
ORDER BY year, p_brand; 
 
--Q2.2
SELECT 
sum(lo_revenue), year(lo_orderdate) AS year, p_brand 
FROM lineorder_flat 
WHERE p_brand >= 'MFGR#2221' AND p_brand <= 'MFGR#2228' AND s_region = 'ASIA' 
GROUP BY year, p_brand 
ORDER BY year, p_brand; 
  
--Q2.3
SELECT sum(lo_revenue), year(lo_orderdate) AS year, p_brand 
FROM lineorder_flat 
WHERE p_brand = 'MFGR#2239' AND s_region = 'EUROPE' 
GROUP BY year, p_brand 
ORDER BY year, p_brand; 
 
--Q3.1
SELECT
```
```SQL
--Q3.1
SELECT
    c_nation,
    s_nation,
    year(lo_orderdate) AS year,
    sum(lo_revenue) AS revenue
FROM lineorder_flat
WHERE c_region = 'ASIA' AND s_region = 'ASIA' AND lo_orderdate BETWEEN '1992-01-01' AND '1997-12-31'
GROUP BY c_nation, s_nation, year
ORDER BY year ASC, revenue DESC;

--Q3.2
SELECT c_city, s_city, year(lo_orderdate) AS year, sum(lo_revenue) AS revenue
FROM lineorder_flat
WHERE c_nation = 'UNITED STATES' AND s_nation = 'UNITED STATES'
AND lo_orderdate BETWEEN '1992-01-01' AND '1997-12-31'
GROUP BY c_city, s_city, year
ORDER BY year ASC, revenue DESC;

--Q3.3
SELECT c_city, s_city, year(lo_orderdate) AS year, sum(lo_revenue) AS revenue
FROM lineorder_flat
WHERE c_city IN ('UNITED KI1', 'UNITED KI5') AND s_city IN ('UNITED KI1', 'UNITED KI5')
AND lo_orderdate BETWEEN '1992-01-01' AND '1997-12-31'
GROUP BY c_city, s_city, year
ORDER BY year ASC, revenue DESC;

--Q3.4
SELECT c_city, s_city, year(lo_orderdate) AS year, sum(lo_revenue) AS revenue
FROM lineorder_flat
WHERE c_city IN ('UNITED KI1', 'UNITED KI5') AND s_city IN ('UNITED KI1', 'UNITED KI5')
AND lo_orderdate BETWEEN '1997-12-01' AND '1997-12-31'
GROUP BY c_city, s_city, year
ORDER BY year ASC, revenue DESC;

--Q4.1
SELECT year(lo_orderdate) AS year, c_nation, sum(lo_revenue - lo_supplycost) AS profit
FROM lineorder_flat
WHERE c_region = 'AMERICA' AND s_region = 'AMERICA' AND p_mfgr IN ('MFGR#1', 'MFGR#2')
GROUP BY year, c_nation
ORDER BY year ASC, c_nation ASC;

--Q4.2
SELECT year(lo_orderdate) AS year,
    s_nation, p_category, sum(lo_revenue - lo_supplycost) AS profit
FROM lineorder_flat
WHERE c_region = 'AMERICA' AND s_region = 'AMERICA'
AND lo_orderdate BETWEEN '1997-01-01' AND '1998-12-31'
AND p_mfgr IN ('MFGR#1', 'MFGR#2')
GROUP BY year, s_nation, p_category
ORDER BY year ASC, s_nation ASC, p_category ASC;

--Q4.3
SELECT year(lo_orderdate) AS year, s_city, p_brand,
    sum(lo_revenue - lo_supplycost) AS profit
FROM lineorder_flat
WHERE s_nation = 'UNITED STATES'
AND lo_orderdate BETWEEN '1997-01-01' AND '1998-12-31'
AND p_category = 'MFGR#14'
GROUP BY year, s_city, p_brand
ORDER BY year ASC, s_city ASC, p_brand ASC;
```

### 建表语句

#### 默认 `lineorder_flat` 表

以下语句适用于当前集群大小和数据规模（三个BE节点，比例因子=100）。如果您的集群有更多的BE节点或更大的数据规模，您可以调整桶数，重新创建表，并重新加载数据以获得更好的测试结果。

```SQL
CREATE TABLE `lineorder_flat` (
  `LO_ORDERDATE` date NOT NULL COMMENT "",
  `LO_ORDERKEY` int(11) NOT NULL COMMENT "",
  `LO_LINENUMBER` tinyint(4) NOT NULL COMMENT "",
  `LO_CUSTKEY` int(11) NOT NULL COMMENT "",
  `LO_PARTKEY` int(11) NOT NULL COMMENT "",
  `LO_SUPPKEY` int(11) NOT NULL COMMENT "",
  `LO_ORDERPRIORITY` varchar(100) NOT NULL COMMENT "",
  `LO_SHIPPRIORITY` tinyint(4) NOT NULL COMMENT "",
  `LO_QUANTITY` tinyint(4) NOT NULL COMMENT "",
  `LO_EXTENDEDPRICE` int(11) NOT NULL COMMENT "",
  `LO_ORDTOTALPRICE` int(11) NOT NULL COMMENT "",
  `LO_DISCOUNT` tinyint(4) NOT NULL COMMENT "",
  `LO_REVENUE` int(11) NOT NULL COMMENT "",
  `LO_SUPPLYCOST` int(11) NOT NULL COMMENT "",
  `LO_TAX` tinyint(4) NOT NULL COMMENT "",
  `LO_COMMITDATE` date NOT NULL COMMENT "",
  `LO_SHIPMODE` varchar(100) NOT NULL COMMENT "",
  `C_NAME` varchar(100) NOT NULL COMMENT "",
  `C_ADDRESS` varchar(100) NOT NULL COMMENT "",
  `C_CITY` varchar(100) NOT NULL COMMENT "",
  `C_NATION` varchar(100) NOT NULL COMMENT "",
  `C_REGION` varchar(100) NOT NULL COMMENT "",
  `C_PHONE` varchar(100) NOT NULL COMMENT "",
  `C_MKTSEGMENT` varchar(100) NOT NULL COMMENT "",
  `S_NAME` varchar(100) NOT NULL COMMENT "",
  `S_ADDRESS` varchar(100) NOT NULL COMMENT "",
  `S_CITY` varchar(100) NOT NULL COMMENT "",
  `S_NATION` varchar(100) NOT NULL COMMENT "",
  `S_REGION` varchar(100) NOT NULL COMMENT "",
  `S_PHONE` varchar(100) NOT NULL COMMENT "",
  `P_NAME` varchar(100) NOT NULL COMMENT "",
  `P_MFGR` varchar(100) NOT NULL COMMENT "",
  `P_CATEGORY` varchar(100) NOT NULL COMMENT "",
  `P_BRAND` varchar(100) NOT NULL COMMENT "",
  `P_COLOR` varchar(100) NOT NULL COMMENT "",
  `P_TYPE` varchar(100) NOT NULL COMMENT "",
  `P_SIZE` tinyint(4) NOT NULL COMMENT "",
  `P_CONTAINER` varchar(100) NOT NULL COMMENT ""
) ENGINE=OLAP
DUPLICATE KEY(`LO_ORDERDATE`, `LO_ORDERKEY`)
COMMENT "OLAP"
PARTITION BY RANGE(`LO_ORDERDATE`)
(
    PARTITION p1992 VALUES LESS THAN ('1993-01-01'),
    PARTITION p1993 VALUES LESS THAN ('1994-01-01'),
    PARTITION p1994 VALUES LESS THAN ('1995-01-01'),
    PARTITION p1995 VALUES LESS THAN ('1996-01-01'),
    PARTITION p1996 VALUES LESS THAN ('1997-01-01'),
    PARTITION p1997 VALUES LESS THAN ('1998-01-01')
)
DISTRIBUTED BY HASH(`LO_ORDERKEY`) BUCKETS 48
PROPERTIES ("replication_num" = "1");
```

#### 带有位图索引的 `lineorder_flat` 表

```SQL
CREATE TABLE `lineorder_flat` (
  `LO_ORDERDATE` date NOT NULL COMMENT "",
  `LO_ORDERKEY` int(11) NOT NULL COMMENT "",
  `LO_LINENUMBER` tinyint(4) NOT NULL COMMENT "",
  `LO_CUSTKEY` int(11) NOT NULL COMMENT "",
  `LO_PARTKEY` int(11) NOT NULL COMMENT "",
  `LO_SUPPKEY` int(11) NOT NULL COMMENT "",
  `LO_ORDERPRIORITY` varchar(100) NOT NULL COMMENT "",
  `LO_SHIPPRIORITY` tinyint(4) NOT NULL COMMENT "",
  `LO_QUANTITY` tinyint(4) NOT NULL COMMENT "",
  `LO_EXTENDEDPRICE` int(11) NOT NULL COMMENT "",
  `LO_ORDTOTALPRICE` int(11) NOT NULL COMMENT "",
  `LO_DISCOUNT` tinyint(4) NOT NULL COMMENT "",
  `LO_REVENUE` int(11) NOT NULL COMMENT "",
  `LO_SUPPLYCOST` int(11) NOT NULL COMMENT "",
  `LO_TAX` tinyint(4) NOT NULL COMMENT "",
  `LO_COMMITDATE` date NOT NULL COMMENT "",
  `LO_SHIPMODE` varchar(100) NOT NULL COMMENT "",
  `C_NAME` varchar(100) NOT NULL COMMENT "",
  `C_ADDRESS` varchar(100) NOT NULL COMMENT "",
  `C_CITY` varchar(100) NOT NULL COMMENT "",
  `C_NATION` varchar(100) NOT NULL COMMENT "",
  `C_REGION` varchar(100) NOT NULL COMMENT "",
  `C_PHONE` varchar(100) NOT NULL COMMENT "",
  `C_MKTSEGMENT` varchar(100) NOT NULL COMMENT "",
  `S_NAME` varchar(100) NOT NULL COMMENT "",
  `S_ADDRESS` varchar(100) NOT NULL COMMENT "",
  `S_CITY` varchar(100) NOT NULL COMMENT "",
  `S_NATION` varchar(100) NOT NULL COMMENT "",
  `S_REGION` varchar(100) NOT NULL COMMENT "",
  `S_PHONE` varchar(100) NOT NULL COMMENT "",
  `P_NAME` varchar(100) NOT NULL COMMENT "",
  `P_MFGR` varchar(100) NOT NULL COMMENT "",
  `P_CATEGORY` varchar(100) NOT NULL COMMENT "",
  `P_BRAND` varchar(100) NOT NULL COMMENT "",
  `P_COLOR` varchar(100) NOT NULL COMMENT "",
  `P_TYPE` varchar(100) NOT NULL COMMENT "",
  `P_SIZE` tinyint(4) NOT NULL COMMENT "",
  `P_CONTAINER` varchar(100) NOT NULL COMMENT "",
  INDEX `bitmap_lo_orderpriority` (`LO_ORDERPRIORITY`) USING BITMAP,
  INDEX `bitmap_lo_shipmode` (`LO_SHIPMODE`) USING BITMAP,
  INDEX `bitmap_c_name` (`C_NAME`) USING BITMAP,
  INDEX `bitmap_c_address` (`C_ADDRESS`) USING BITMAP,
  INDEX `bitmap_c_city` (`C_CITY`) USING BITMAP,
  INDEX `bitmap_c_nation` (`C_NATION`) USING BITMAP,
  INDEX `bitmap_c_region` (`C_REGION`) USING BITMAP,
  INDEX `bitmap_c_phone` (`C_PHONE`) USING BITMAP,
  INDEX `bitmap_c_mktsegment` (`C_MKTSEGMENT`) USING BITMAP,
  INDEX `bitmap_s_name` (`S_NAME`) USING BITMAP,
  INDEX `bitmap_s_address` (`S_ADDRESS`) USING BITMAP,
  INDEX `bitmap_s_city` (`S_CITY`) USING BITMAP,
  INDEX `bitmap_s_nation` (`S_NATION`) USING BITMAP,
  INDEX `bitmap_s_region` (`S_REGION`) USING BITMAP,
  INDEX `bitmap_s_phone` (`S_PHONE`) USING BITMAP,
  INDEX `bitmap_p_name` (`P_NAME`) USING BITMAP,
  INDEX `bitmap_p_mfgr` (`P_MFGR`) USING BITMAP,
  INDEX `bitmap_p_category` (`P_CATEGORY`) USING BITMAP,
  INDEX `bitmap_p_brand` (`P_BRAND`) USING BITMAP,
  INDEX `bitmap_p_color` (`P_COLOR`) USING BITMAP,
  INDEX `bitmap_p_type` (`P_TYPE`) USING BITMAP,
  INDEX `bitmap_p_container` (`P_CONTAINER`) USING BITMAP
) ENGINE=OLAP
DUPLICATE KEY(`LO_ORDERDATE`, `LO_ORDERKEY`)
COMMENT "OLAP"
PARTITION BY RANGE(`LO_ORDERDATE`)
(
    PARTITION p1992 VALUES LESS THAN ('1993-01-01'),
    PARTITION p1993 VALUES LESS THAN ('1994-01-01'),
    PARTITION p1994 VALUES LESS THAN ('1995-01-01'),
    PARTITION p1995 VALUES LESS THAN ('1996-01-01'),
    PARTITION p1996 VALUES LESS THAN ('1997-01-01'),
    PARTITION p1997 VALUES LESS THAN ('1998-01-01')
)
DISTRIBUTED BY HASH(`LO_ORDERKEY`) BUCKETS 48
PROPERTIES ("replication_num" = "1");
```
```SQL
) ENGINE=OLAP
DUPLICATE KEY(`LO_ORDERDATE`, `LO_ORDERKEY`)
COMMENT "OLAP"
PARTITION BY date_trunc('year', `LO_ORDERDATE`)
DISTRIBUTED BY HASH(`LO_ORDERKEY`) BUCKETS 48
PROPERTIES ("replication_num" = "1");
```