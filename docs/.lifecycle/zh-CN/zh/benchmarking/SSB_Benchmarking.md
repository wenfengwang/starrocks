---
displayed_sidebar: "Chinese"
---

# SSB Flat Table Performance Test

## Test Conclusion

Star Schema Benchmark (referred to as SSB) is widely used in both academic and industrial circles as a star model test suite (source [paper](https://www.cs.umb.edu/~poneil/StarSchemaB.PDF)). Through this test suite, various OLAP product's basic performance metrics can be conveniently compared. ClickHouse has rewritten the SSB to convert the star model to a wide table (flat table), creating a single-table test benchmark (refer to [link](https://clickhouse.tech/docs/en/getting-started/example-datasets/star-schema/)). This report records the performance comparison results of StarRocks, ClickHouse, and Apache Druid on the SSB single-table dataset, and the test conclusion is as follows:

- On the 13 standard test queries, StarRocks' overall query performance is 2.1 times that of ClickHouse and 8.7 times that of Apache Druid.
- After StarRocks enables Bitmap Index, the overall query performance is 1.3 times that of the unenabled state. At this time, the overall query performance is 2.8 times that of ClickHouse and 11.4 times that of Apache Druid.

![img](../assets/7.1-1.png)

This article compares the query performance of StarRocks, ClickHouse, and Apache Druid in the SSB single-table scenario. Testing was conducted on a cloud server with 3x16 core 64GB memory, with a data scale of 600 million rows.

## Test Preparation


### Hardware Environment

| Machine | 3 Alibaba Cloud servers |
| -------- | ------------------------- |
| CPU | 16core Intel(R) Xeon(R) Platinum 8269CY CPU @ 2.50GHz <br/> Cache size: 36608 KB |
| Memory | 64GB |
| Network Bandwidth | 5 Gbits/s |
| Disk | ESSD Cloud Drive |

### Software Environment

StarRocks, ClickHouse, and Apache Druid are deployed on machines with the same configuration for testing.

- StarRocks is deployed with 1 FE and 3 BE. The FE can be deployed separately or mixed with the BE.
- ClickHouse is deployed with three nodes and establishes a distributed table.
- Apache Druid has three nodes deployed as Data Servers, and one node each for mixed deployment of Master Servers and Query Servers.

Kernel Version: Linux 3.10.0-1160.59.1.el7.x86_64

Operating System Version: CentOS Linux release 7.9.2009

Software Version: StarRocks Community Edition 3.0, ClickHouse 23.3, Apache Druid 25.0.0

## Test Data and Results

### Test Data

| Table Name | Number of Rows | Description |
| -------------- | ------ | ---------------- |
| lineorder      | 600 million   | SSB Line Order Table   |
| customer       | 3 million | SSB Customer Table       |
| part           | 1.4 million | SSB Part Table     |
| supplier       | 200,000  | SSB Supplier Table     |
| dates          | 2556   | Date Table           |
| lineorder_flat | 600 million   | Wide table after flattening the SSB |

### Test Results

> The unit of query time is ms. The comparison of query performance between StarRocks and ClickHouse/Druid is achieved by dividing the query time of ClickHouse/Druid by the query time of StarRocks. A larger result value indicates better performance of StarRocks.

|      | StarRocks-3.0 | StarRocks-3.0-index | ClickHouse-23.3 | ClickHouse vs StarRocks | Druid-25.0.0 | Druid vs StarRocks |
| ---- | ------------- | ------------------- | --------------- | ----------------------- | -------------| ------------------ |
| Q1.1 | 33            | 30                  | 48              | 1.45                    | 430          | 13.03              |
| Q1.2 | 10            | 10                  | 15              | 1.50                    | 270          | 27.00              |
| Q1.3 | 23            | 30                  | 14              | 0.61                    | 820          | 35.65              |
| Q2.1 | 186           | 116                 | 301             | 1.62                    | 760          | 4.09               |
| Q2.2 | 156           | 50                  | 273             | 1.75                    | 920          | 5.90               |
| Q2.3 | 73            | 36                  | 255             | 3.49                    | 910          | 12.47              |
| Q3.1 | 173           | 233                 | 398             | 2.30                    | 1080         | 6.24               |
| Q3.2 | 120           | 80                  | 319             | 2.66                    | 850          | 7.08               |
| Q3.3 | 123           | 30                  | 227             | 1.85                    | 890          | 7.24               |
| Q3.4 | 13            | 16                  | 18              | 1.38                    | 750          | 57.69              |
| Q4.1 | 203           | 196                 | 469             | 2.31                    | 1230         | 6.06               |
| Q4.2 | 73            | 76                  | 160             | 2.19                    | 1020         | 13.97              |
| Q4.3 | 50            | 36                  | 148             | 2.96                    | 820          | 16.40              |
| SUM  | 1236          | 939                 | 2645            | 2.14                    | 10750        | 8.70               |

## Test Steps

ClickHouse's table creation and import can be referred to in the [official documentation](https://clickhouse.tech/docs/en/getting-started/example-datasets/star-schema/). The data generation and import process for StarRocks is as follows:

### Data Generation

First, download the ssb-poc toolset and compile it.

```Bash
wget https://starrocks-public.oss-cn-zhangjiakou.aliyuncs.com/ssb-poc-1.0.zip
unzip ssb-poc-1.0.zip
cd ssb-poc-1.0/
make && make install
cd output/
```

After compilation, all related tools are installed in the output directory, and all subsequent operations are carried out in the output directory.

Generate data for the `scale factor=100` standard SSB test set.

```Bash
sh bin/gen-ssb.sh 100 data_dir
```

### Create Table Structure

Modify the configuration file `conf/starrocks.conf` to specify the cluster address for the script operation. Pay attention to `mysql_host` and `mysql_port`, then execute the table creation operation.

```SQL
sh bin/create_db_table.sh ddl_100
```

### Import Data

Use Stream Load to import data from multiple tables, then flatten multiple tables into a single table using INSERT INTO.

```Bash
sh bin/flat_insert.sh data_dir
```

### Query Data

```Bash
sh bin/benchmark.sh ssb-flat
```

### Enable Bitmap Index

StarRocks' performance is significantly improved when Bitmap Index is enabled, especially in Q2.2, Q2.3, and Q3.3. If you want to test the performance with Bitmap Index enabled, you can create Bitmap Index for all string columns. The specific steps are as follows:

1. Re-create the `lineorder_flat` table when creating all Bitmap Index.

```SQL
sh bin/create_db_table.sh ddl_100_bitmap_index
```

2. Add the following parameters to the configuration file on all BE nodes, then restart the BE.

```SQL
bitmap_max_filter_ratio=1000
```

3. Re-execute the import command.
    sh bin/flat_insert.sh data_dir
    ```


导入完成后需要等待 compaction 完成，再重新执行步骤 [4.4](#查询数据)，此时就是启用 Bitmap Index 后的查询结果。

可以通过 `select CANDIDATES_NUM from information_schema.be_compactions` 命令查看 compaction 进度。对于 3 个 BE 节点，如下结果说明 compaction 完成：

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

## 测试 SQL 与建表语句

### 测试 SQL

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

### 建表语句

#### `lineorder_flat` 默认建表

这个建表语句是为了匹配当前集群和数据规格（3 个 BE，scale factor = 100）。如果您的集群有更多的 BE 节点，或者更大的数据规格，可以调整分桶数，重新建表和导数据，可实现更好的测试效果。

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
```SQL
-- 创建 `lineorder_flat` Bitmap 索引表

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
  索引 bitmap_lo_orderpriority (lo_orderpriority)  使用 BITMAP,
  索引 bitmap_lo_shipmode (lo_shipmode)  使用 BITMAP,
  索引 bitmap_c_name (c_name)  使用 BITMAP,
  索引 bitmap_c_address (c_address)  使用 BITMAP,
  索引 bitmap_c_city (c_city)  使用 BITMAP,
  索引 bitmap_c_nation (c_nation)  使用 BITMAP,
  索引 bitmap_c_region (c_region)  使用 BITMAP,
  索引 bitmap_c_phone (c_phone)  使用 BITMAP,
  索引 bitmap_c_mktsegment (c_mktsegment)  使用 BITMAP,
  索引 bitmap_s_region (s_region)  使用 BITMAP,
  索引 bitmap_s_nation (s_nation)  使用 BITMAP,
  索引 bitmap_s_city (s_city)  使用 BITMAP,
  索引 bitmap_s_name (s_name)  使用 BITMAP,
  索引 bitmap_s_address (s_address)  使用 BITMAP,
  索引 bitmap_s_phone (s_phone)  使用 BITMAP,
  索引 bitmap_p_name (p_name)  使用 BITMAP,
  索引 bitmap_p_mfgr (p_mfgr)  使用 BITMAP,
  索引 bitmap_p_category (p_category)  使用 BITMAP,
  索引 bitmap_p_brand (p_brand)  使用 BITMAP,
  索引 bitmap_p_color (p_color)  使用 BITMAP,
  索引 bitmap_p_type (p_type)  使用 BITMAP,
  索引 bitmap_p_container (p_container)  使用 BITMAP
) ENGINE=OLAP
重复关键（`LO_ORDERDATE`，`LO_ORDERKEY`）
评论 "OLAP"
按 date_trunc('year', `LO_ORDERDATE`) 分区
散列分布（`LO_ORDERKEY`） 桶 48
属性（"replication_num" = "1");
```