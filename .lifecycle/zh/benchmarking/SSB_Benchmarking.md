---
displayed_sidebar: English
---

# SSB 平面基准测试

星型模式基准（SSB）设计用于测试 OLAP 数据库产品的基础性能指标。SSB 使用了在学术界和工业界广泛应用的星型模式测试集。更多信息请参见[《星型模式基准》](https://www.cs.umb.edu/~poneil/StarSchemaB.PDF)论文。
ClickHouse将星型模式转换为宽平面表，并将SSB改写为单表基准测试。更多信息，请参阅[ClickHouse 的星型模式基准](https://clickhouse.tech/docs/en/getting-started/example-datasets/star-schema/)。此测试比较了StarRocks、Apache Druid和ClickHouse在SSB单表数据集上的性能。

## 测试结论

- 在对 SSB 标准数据集进行的 13 个查询中，StarRocks 的整体查询性能是 ClickHouse 的 **2.1 倍**，是 Apache Druid 的 **8.7 倍**。
- 在 StarRocks 启用位图索引后，性能比未启用时提高了 1.3 倍。StarRocks 的整体性能是 **ClickHouse 的 2.8 倍，是 Apache Druid 的 11.4 倍**。

![overall comparison](../assets/7.1-1.png)

## 测试准备

### 硬件

|机器|云主机3台|
|---|---|
|CPU|16 核 Intel (R) Xeon (R) Platinum 8269CY CPU @2.50GHz 缓存大小：36608 KB|
|内存|64 GB|
|网络带宽|5 Gbit/s|
|磁盘|SSD|

### 软件

StarRocks、Apache Druid 和 ClickHouse 都部署在配置相同的主机上。

- StarRocks：一个 FE 和三个 BE。FE 可以单独部署，也可以与 BEs 混合部署。
- ClickHouse：三个节点，使用分布式表。
- Apache Druid：三个节点。一个部署了主服务器和数据服务器，一个部署了查询服务器和数据服务器，第三个仅部署了数据服务器。

内核版本：Linux 3.10.0-1160.59.1.el7.x86_64

操作系统版本：CentOS Linux 发行版 7.9.2009

软件版本：StarRocks 社区版 3.0、ClickHouse 23.3、Apache Druid 25.0.0

## 测试数据和结果

### 测试数据

|表|记录|描述|
|---|---|---|
|lineorder|6亿|lineorder事实表|
|客户|300万|客户维度表|
|零件|140万|零件尺寸表|
|供应商|20万|供应商维度表|
|日期|2,556|日期维度表|
|lineorder_flat|6亿|lineorder平表|

### 测试结果

下表展示了 13 个查询的性能测试结果。查询延迟的单位是毫秒（ms）。表头中的 "ClickHouse 对 StarRocks" 表示使用 ClickHouse 的查询响应时间除以 StarRocks 的查询响应时间。数值越大，表示 StarRocks 的性能越好。

|StarRocks-3.0|StarRocks-3.0-index|ClickHouse-23.3|ClickHouse 与 StarRocks|Druid-25.0.0|Druid 与 StarRocks|
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
|Q4.1|203|196|469|2.31​​|1230|6.06|
|Q4.2|73|76|160|2.19|1020|13.97|
|Q4.3|50|36|148|2.96|820|16.40|
|总和|1236|939|2645|2.14|10750|8.70|

## 测试过程

有关如何创建 [ClickHouse official doc](https://clickhouse.tech/docs/en/getting-started/example-datasets/star-schema) 及向其中加载数据的更多信息，请参阅 ClickHouse 官方文档。以下部分描述了 StarRocks 的数据生成和数据加载过程。

### 生成数据

下载 ssb-poc 工具包并进行编译。

```Bash
wget https://starrocks-public.oss-cn-zhangjiakou.aliyuncs.com/ssb-poc-1.0.zip
unzip ssb-poc-1.0.zip
cd ssb-poc-1.0/
make && make install
cd output/
```

编译完成后，所有相关工具都会安装到输出目录中，后续的操作都在此目录下进行。

首先，为 SSB 标准数据集生成比例因子为 100 的数据。

```Bash
sh bin/gen-ssb.sh 100 data_dir
```

### 创建表结构

1. 修改配置文件 conf/starrocks.conf，指定集群地址。特别注意 mysql_host 和 mysql_port 的设置。

2. 运行以下命令来创建表：

   ```SQL
   sh bin/create_db_table.sh ddl_100
   ```

### 查询数据

```Bash
sh bin/benchmark.sh ssb-flat
```

### 启用位图索引

启用位图索引后，StarRocks 的查询性能会得到提升。如果您希望测试启用位图索引的 StarRocks 性能，尤其是在 Q2.2、Q2.3 和 Q3.3 这些查询上，您可以为所有 STRING 类型的列创建位图索引。

1. 创建一个新的 lineorder_flat 表并为其创建位图索引。

   ```SQL
   sh bin/create_db_table.sh ddl_100_bitmap_index
   ```

2. 在所有 BE 节点的 be.conf 文件中添加以下配置，并重启 BE 以使配置生效。

   ```SQL
   bitmap_max_filter_ratio=1000
   ```

3. 运行数据加载脚本。

   ```SQL
   sh bin/flat_insert.sh data_dir
   ```

数据加载完成后，等待数据版本合并完成，然后再次执行[4.4](#query-data)以在启用位图索引后查询数据。

您可以通过执行 select CANDIDATES_NUM from information_schema.be_compactions 来查看数据版本合并的进度。对于三个 BE 节点，以下结果表明合并已经完成：

```SQL
mysql> select CANDIDATES_NUM from information_schema.be_compactions;
+----------------+
| CANDIDATES_NUM |
+----------------+
|              0 |
|              0 |
|              0 |
+----------------+
3 rows in set (0.01 sec)
```

## 测试 SQL 语句和表创建语句

### 测试 SQL 语句

```SQL
--Q1.1 
SELECT sum(lo_extendedprice * lo_discount) AS `revenue` 
FROM lineorder_flat 
WHERE lo_orderdate >= '1993-01-01' and lo_orderdate <= '1993-12-31'
AND lo_discount BETWEEN 1 AND 3 AND lo_quantity < 25; 
 
--Q1.2 
SELECT sum(lo_extendedprice * lo_discount) AS revenue FROM lineorder_flat  
WHERE lo_orderdate >= '1994-01-01' and lo_orderdate <= '1994-01-31'
AND lo_discount BETWEEN 4 AND 6 AND lo_quantity BETWEEN 26 AND 35; 
 
--Q1.3 
SELECT sum(lo_extendedprice * lo_discount) AS revenue 
FROM lineorder_flat 
WHERE weekofyear(lo_orderdate) = 6
AND lo_orderdate >= '1994-01-01' and lo_orderdate <= '1994-12-31' 
AND lo_discount BETWEEN 5 AND 7 AND lo_quantity BETWEEN 26 AND 35; 
 
--Q2.1 
SELECT sum(lo_revenue), year(lo_orderdate) AS year,  p_brand 
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
    c_nation,
    s_nation,
    year(lo_orderdate) AS year,
    sum(lo_revenue) AS revenue FROM lineorder_flat 
WHERE c_region = 'ASIA' AND s_region = 'ASIA' AND lo_orderdate >= '1992-01-01'
AND lo_orderdate <= '1997-12-31' 
GROUP BY c_nation, s_nation, year 
ORDER BY  year ASC, revenue DESC; 
 
--Q3.2 
SELECT c_city, s_city, year(lo_orderdate) AS year, sum(lo_revenue) AS revenue
FROM lineorder_flat 
WHERE c_nation = 'UNITED STATES' AND s_nation = 'UNITED STATES'
AND lo_orderdate  >= '1992-01-01' AND lo_orderdate <= '1997-12-31' 
GROUP BY c_city, s_city, year 
ORDER BY year ASC, revenue DESC; 
 
--Q3.3 
SELECT c_city, s_city, year(lo_orderdate) AS year, sum(lo_revenue) AS revenue 
FROM lineorder_flat 
WHERE c_city in ( 'UNITED KI1' ,'UNITED KI5') AND s_city in ('UNITED KI1', 'UNITED KI5')
AND lo_orderdate  >= '1992-01-01' AND lo_orderdate <= '1997-12-31' 
GROUP BY c_city, s_city, year 
ORDER BY year ASC, revenue DESC; 
 
--Q3.4 
SELECT c_city, s_city, year(lo_orderdate) AS year, sum(lo_revenue) AS revenue 
FROM lineorder_flat 
WHERE c_city in ('UNITED KI1', 'UNITED KI5') AND s_city in ('UNITED KI1', 'UNITED KI5')
AND lo_orderdate  >= '1997-12-01' AND lo_orderdate <= '1997-12-31' 
GROUP BY c_city, s_city, year 
ORDER BY year ASC, revenue DESC; 
 
--Q4.1 
SELECT year(lo_orderdate) AS year, c_nation, sum(lo_revenue - lo_supplycost) AS profit
FROM lineorder_flat 
WHERE c_region = 'AMERICA' AND s_region = 'AMERICA' AND p_mfgr in ('MFGR#1', 'MFGR#2') 
GROUP BY year, c_nation 
ORDER BY year ASC, c_nation ASC; 
 
--Q4.2 
SELECT year(lo_orderdate) AS year, 
    s_nation, p_category, sum(lo_revenue - lo_supplycost) AS profit 
FROM lineorder_flat 
WHERE c_region = 'AMERICA' AND s_region = 'AMERICA'
AND lo_orderdate >= '1997-01-01' and lo_orderdate <= '1998-12-31'
AND p_mfgr in ( 'MFGR#1' , 'MFGR#2') 
GROUP BY year, s_nation, p_category 
ORDER BY year ASC, s_nation ASC, p_category ASC; 
 
--Q4.3 
SELECT year(lo_orderdate) AS year, s_city, p_brand, 
    sum(lo_revenue - lo_supplycost) AS profit 
FROM lineorder_flat 
WHERE s_nation = 'UNITED STATES'
AND lo_orderdate >= '1997-01-01' and lo_orderdate <= '1998-12-31'
AND p_category = 'MFGR#14' 
GROUP BY year, s_city, p_brand 
ORDER BY year ASC, s_city ASC, p_brand ASC; 
```

### 表创建语句

#### 默认的 lineorder_flat 表

以下语句适用于当前集群规模和数据规模（三个 BE，比例因子 = 100）。如果您的集群有更多的 BE 节点或更大的数据规模，您可以调整桶的数量，重新创建表，并重新加载数据以获得更好的测试结果。

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
PARTITION BY date_trunc('year', `LO_ORDERDATE`)
DISTRIBUTED BY HASH(`LO_ORDERKEY`) BUCKETS 48
PROPERTIES ("replication_num" = "1");
```

#### 带位图索引的 lineorder_flat 表

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
  index bitmap_lo_orderpriority (lo_orderpriority) USING BITMAP,
  index bitmap_lo_shipmode (lo_shipmode) USING BITMAP,
  index bitmap_c_name (c_name) USING BITMAP,
  index bitmap_c_address (c_address) USING BITMAP,
  index bitmap_c_city (c_city) USING BITMAP,
  index bitmap_c_nation (c_nation) USING BITMAP,
  index bitmap_c_region (c_region) USING BITMAP,
  index bitmap_c_phone (c_phone) USING BITMAP,
  index bitmap_c_mktsegment (c_mktsegment) USING BITMAP,
  index bitmap_s_region (s_region) USING BITMAP,
  index bitmap_s_nation (s_nation) USING BITMAP,
  index bitmap_s_city (s_city) USING BITMAP,
  index bitmap_s_name (s_name) USING BITMAP,
  index bitmap_s_address (s_address) USING BITMAP,
  index bitmap_s_phone (s_phone) USING BITMAP,
  index bitmap_p_name (p_name) USING BITMAP,
  index bitmap_p_mfgr (p_mfgr) USING BITMAP,
  index bitmap_p_category (p_category) USING BITMAP,
  index bitmap_p_brand (p_brand) USING BITMAP,
  index bitmap_p_color (p_color) USING BITMAP,
  index bitmap_p_type (p_type) USING BITMAP,
  index bitmap_p_container (p_container) USING BITMAP
) ENGINE=OLAP
DUPLICATE KEY(`LO_ORDERDATE`, `LO_ORDERKEY`)
COMMENT "OLAP"
PARTITION BY date_trunc('year', `LO_ORDERDATE`)
DISTRIBUTED BY HASH(`LO_ORDERKEY`) BUCKETS 48
PROPERTIES ("replication_num" = "1");
```
