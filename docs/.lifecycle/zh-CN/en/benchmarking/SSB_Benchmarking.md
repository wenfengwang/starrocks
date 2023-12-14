---
displayed_sidebar: "Chinese"
---

# SSB平面表基准测试

星型模式基准（SSB）旨在测试OLAP数据库产品的基本性能指标。SSB使用了一个广泛应用于学术界和工业界的星型模式测试集。更多信息请参阅论文[星型模式基准](https://www.cs.umb.edu/~poneil/StarSchemaB.PDF)。ClickHouse将星型模式展平为一个宽平面表，并将SSB重写为单表基准。更多信息请参阅[ClickHouse的星型模式基准](https://clickhouse.tech/docs/en/getting-started/example-datasets/star-schema/)。此测试比较了StarRocks、Apache Druid和ClickHouse对SSB单表数据集的性能。

## 测试结论

- 在对SSB标准数据集执行的13个查询中，StarRocks的整体查询性能是**ClickHouse的2.1倍，Apache Druid的8.7倍**。

- 启用StarRocks的位图索引后，性能比禁用时提高了1.3倍。StarRocks的整体性能是**ClickHouse的2.8倍，Apache Druid的11.4倍**。


![整体比较](../assets/7.1-1.png)

## 测试准备

### 硬件

| 机器            | 3个云主机                                                    |
| -------------- | ------------------------------------------------------------ |
| CPU               | 16核英特尔（R）至强（R）白金8269CYCPU @2.50GHz <br />缓存大小: 36608 KB |
| 内存            | 64 GB                                                        |
| 网络带宽 | 5 Gbit/s                                                     |
| 硬盘              | ESSD                                                         |

### 软件

StarRocks、Apache Druid和ClickHouse均部署在相同配置的主机上。

- StarRocks：一个前端和三个后端。前端可以单独部署或与后端混合部署。
- ClickHouse：三个节点，具有分布式表
- Apache Druid：三个节点。一个部署了主服务器和数据服务器，一个部署了查询服务器和数据服务器，第三个只部署了数据服务器。

内核版本: Linux 3.10.0-1160.59.1.el7.x86_64

操作系统版本: CentOS Linux release 7.9.2009

软件版本: StarRocks社区版 3.0, ClickHouse 23.3, Apache Druid 25.0.0

## 测试数据和结果

### 测试数据

| 表          | 记录数       | 描述              |
| -------------- | ------------ | ------------------------ |
| lineorder      | 6亿  | Lineorder事实表     |
| customer       | 300万    | Customer维表 |
| part           | 140万  | Part维表     |
| supplier       | 20万 | Supplier维表 |
| dates          | 2,556        | Date维表     |
| lineorder_flat | 6亿  | lineorder平面表    |

### 测试结果

下表显示了十三个查询的性能测试结果。查询延迟的单位是毫秒。表头中的`ClickHouse vs StarRocks`表示使用ClickHouse的查询响应时间除以StarRocks的查询响应时间。较大的值表示StarRocks的性能更好。

|      | StarRocks-3.0 | StarRocks-3.0-index | ClickHouse-23.3 | ClickHouse vs StarRocks | Druid-25.0.0 | Druid vs StarRocks |
| ---- | ------------- | ------------------- | ------------------- | ----------------------- | ---------------- | ------------------ |
| Q1.1 | 33            | 30                  | 48                  | 1.45                    | 430              | 13.03              |
| Q1.2 | 10            | 10                  | 15                  | 1.50                    | 270              | 27.00              |
| Q1.3 | 23            | 30                  | 14                  | 0.61                    | 820              | 35.65              |
| Q2.1 | 186           | 116                 | 301                 | 1.62                    | 760              | 4.09               |
| Q2.2 | 156           | 50                  | 273                 | 1.75                    | 920              | 5.90               |
| Q2.3 | 73            | 36                  | 255                 | 3.49                    | 910              | 12.47              |
| Q3.1 | 173           | 233                 | 398                 | 2.30                    | 1080             | 6.24               |
| Q3.2 | 120           | 80                  | 319                 | 2.66                    | 850              | 7.08               |
| Q3.3 | 123           | 30                  | 227                 | 1.85                    | 890              | 7.24               |
| Q3.4 | 13            | 16                  | 18                  | 1.38                    | 750              | 57.69              |
| Q4.1 | 203           | 196                 | 469                 | 2.31                    | 1230             | 6.06               |
| Q4.2 | 73            | 76                  | 160                 | 2.19                    | 1020             | 13.97              |
| Q4.3 | 50            | 36                  | 148                 | 2.96                    | 820              | 16.40              |
| 总计  | 1236          | 939                 | 2645                | 2.14                    | 10750            | 8.70               |

## 测试流程

有关如何创建ClickHouse表和将数据加载到表中的更多信息，请参见[ClickHouse官方文档](https://clickhouse.tech/docs/en/getting-started/example-datasets/star-schema/)。下面的部分描述了StarRocks的数据生成和数据加载。

### 生成数据

下载ssb-poc工具包并编译。

```Bash
wget https://starrocks-public.oss-cn-zhangjiakou.aliyuncs.com/ssb-poc-1.0.zip
unzip ssb-poc-1.0.zip
cd ssb-poc-1.0/
make && make install
cd output/
```

编译完成后，所有相关工具都安装在`output`目录下，并且以下操作都是在该目录下执行。

首先，为SSB标准数据集`scale factor=100`生成数据。

```Bash
sh bin/gen-ssb.sh 100 data_dir
```

### 创建表模式

1. 修改配置文件`conf/starrocks.conf`并指定集群地址。特别注意`mysql_host`和`mysql_port`。

2. 运行以下命令以创建表：

    ```SQL
    sh bin/create_db_table.sh ddl_100
    ```

### 查询数据

```Bash
sh bin/benchmark.sh ssb-flat
```

### 启用位图索引

启用位图索引后，StarRocks的性能会提高。如果要测试StarRocks在启用位图索引后的性能，特别是在Q2.2、Q2.3和Q3.3上，可以为所有STRING列创建位图索引。

1. 创建另一个`lineorder_flat`表并创建位图索引。

    ```SQL
    sh bin/create_db_table.sh ddl_100_bitmap_index
    ```

2. 将以下配置添加到所有BE的`be.conf`文件中，并重新启动BE，使配置生效。

    ```SQL
    bitmap_max_filter_ratio=1000
    ```

3. 运行数据加载脚本。

    ```SQL
    sh bin/flat_insert.sh data_dir
    ```

数据加载完成后，等待数据版本压缩完成，然后再次执行[4.4](#query-data)以查询启用位图索引后的数据。

可以通过运行`select CANDIDATES_NUM from information_schema.be_compactions`来查看数据版本压缩的进度。对于三个BE节点，以下结果显示了压缩已完成：

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

## 测试SQL和表创建语句

### 测试SQL

```SQL
--Q1.1 
SELECT sum(lo_extendedprice * lo_discount) AS `revenue` 
FROM lineorder_flat 
WHERE lo_orderdate >= '1993-01-01' and lo_orderdate <= '1993-12-31'
AND lo_discount BETWEEN 1 AND 3 AND lo_quantity < 25; 
 
--Q1.2 
SELECT sum(lo_extendedprice * lo_discount) AS revenue FROM lineorder_flat  
WHERE lo_orderdate >= '1994-01-01' and lo_orderdate <= '1994-01-31'
```SQL
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

### 创建表语句

#### 默认 `lineorder_flat` 表

以下语句符合当前集群大小和数据量（三个BE，规模因子 = 100）。如果您的集群具有更多BE节点或更大的数据量，可以调整桶的数量，重新创建表并重新加载数据，以获得更好的测试结果。

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

#### 具有位图索引的 `lineorder_flat` 表

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
```sql
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