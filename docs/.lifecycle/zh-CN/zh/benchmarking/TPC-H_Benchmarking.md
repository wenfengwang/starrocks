---
displayed_sidebar: "Chinese"
---

# TPC-H 基准测试

TPC-H 是美国交易处理效能委员会 TPC（Transaction Processing Performance Council）组织制定的用来模拟决策支持类应用的测试集。它包括一整套面向业务的 ad-hoc 查询和并发数据修改。

TPC-H 根据真实的生产运行环境来建模，模拟了一套销售系统的数据仓库。该测试共包含 8 张表，数据量可设定从 1 GB~3 TB不等。其基准测试共包含了 22 个查询，主要评价指标为各个查询的响应时间，即从提交查询到结果返回所需时间。

## 1. 测试结论

在 TPC-H 100G 规模的数据集上进行对比测试，共 22 个查询，结果如下：

![TPCH 100G结果](../assets/tpch.png)

StarRocks 测试了使用本地存储查询和 Hive 外表查询两种方式，其中 StarRocks Hive 外表查询和 Trino 查询使用相同数据，数据采用 ORC 格式存储，zlib 格式压缩。

最终，StarRocks 本地存储查询总耗时为 17s，StarRocks Hive 外表查询总耗时为 92s，Trino 查询总耗时为187s。

## 2. 测试准备

### 2.1 硬件环境与成本

| 机器     | 3台 阿里云主机                                        |
| -------- | ----------------------------------------------------- |
| CPU      | 16core Intel(R) Xeon(R) Platinum 8269CY CPU @ 2.50GHz |
| 内存     | 64 GB                                                  |
| 网络带宽 | 5 Gbits/s                                             |
| 磁盘     | ESSD 云盘                                              |

### 2.2 软件环境

- 内核版本：Linux 3.10.0-1127.13.1.el7.x86_64

- 操作系统版本：CentOS Linux release 7.8.2003  

- 软件版本：StarRocks 社区版 3.0，Trino-419，Hive-2.3.9

## 3 测试数据与结果

### 3.1 测试数据

|  表      | 行数   |
| -------- | ------ |
| customer | 1500万 |
| lineitem | 6亿    |
| nation   | 25     |
| orders   | 1.5亿  |
| part     | 2000万 |
| partsupp | 8000万 |
| region   | 5      |
| supplier | 100万  |

### 3.2 测试结果

> 查询结果的单位是 ms。

|  Query | StarRocks-native-3.0 | StarRocks-3.0-Hive external | Trino-419 |
| ---- | -------------------- | --------------------------- | --------- |
| Q1   | 1540                 | 5660                        | 8811      |
| Q2   | 100                  | 1593                        | 3009      |
| Q3   | 700                  | 5286                        | 7891      |
| Q4   | 423                  | 2110                        | 5760      |
| Q5   | 1180                 | 4453                        | 9181      |
| Q6   | 56                   | 2806                        | 4029      |
| Q7   | 903                  | 4910                        | 7158      |
| Q8   | 546                  | 4766                        | 8014      |
| Q9   | 2553                 | 8010                        | 18460     |
| Q10  | 776                  | 8190                        | 9997      |
| Q11  | 206                  | 920                         | 2088      |
| Q12  | 166                  | 2916                        | 4852      |
| Q13  | 1663                 | 3420                        | 7203      |
| Q14  | 146                  | 3286                        | 4995      |
| Q15  | 123                  | 4173                        | 9688      |
| Q16  | 353                  | 1126                        | 2545      |
| Q17  | 296                  | 3426                        | 18970     |
| Q18  | 2713                 | 7960                        | 21763     |
| Q19  | 246                  | 4406                        | 6586      |
| Q20  | 176                  | 3280                        | 6632      |
| Q21  | 1410                 | 7933                        | 16873     |
| Q22  | 350                  | 1190                        | 2788      |
| SUM  | 16625                | 91820                       | 187293    |

## 4. 测试流程

### 4.1 StarRocks 本地表测试流程

#### 4.1.1 生成数据

下载 tpch-poc 工具包生成 TPC-H 标准测试集 `scale factor=100` 的数据。

```Bash
wget https://starrocks-public.oss-cn-zhangjiakou.aliyuncs.com/tpch-poc-1.0.zip
unzip tpch-poc-1.0
cd tpch-poc-1.0

sh bin/gen_data/gen-tpch.sh 100 data_100
```

#### 4.1.2 创建表结构

修改配置文件 `conf/starrocks.conf`，指定脚本操作的集群地址，重点关注 `mysql_host` 和 `mysql_port`，然后执行建表操作。

```SQL
sh bin/create_db_table.sh ddl_100
```

#### 4.1.3 导入数据

```Python
sh bin/stream_load.sh data_100
```

#### 4.1.4 查询数据

```Python
sh bin/benchmark.sh
```

### 4.2 StarRocks Hive 外表测试流程

#### 4.2.1 创建表结构

在 Hive 中创建外部表，外部表存储格式是 ORC，压缩格式是 zlib，详细建表语句见 [5.3](#53-hive-外部表建表orc-存储格式)，这个 Hive 外部表就是StarRocks 和 Trino 对比测试查询的表。

#### 4.2.2 导入数据

将步骤 [4.1.1](#411-生成数据) 中生成的 TPC-H CSV 原始数据上传到 HDFS 指定路径上，本文以路径 /user/tmp/csv/ 举例，然后在 Hive 中创建外部表，详细建表语句见 [5.4](#54-hive-外部表建表csv-存储格式)。这个 Hive 外部表存储格式为 CSV，存储路径为 CSV 原始数据上传的路径 /user/tmp/csv/。

通过 insert into 将 CSV 格式外部表的数据导入到 ORC 格式的外部表中，这样就得到了存储格式为 ORC，压缩格式为 zlib 的数据，导入命令如下：

```SQL
use tpch_hive_csv；

insert into tpch_hive_orc.customer  select * from customer;
insert into tpch_hive_orc.lineitem  select * from lineitem;
insert into tpch_hive_orc.nation  select * from nation;
insert into tpch_hive_orc.orders  select * from orders;
insert into tpch_hive_orc.part  select * from part;
insert into tpch_hive_orc.partsupp  select * from partsupp;
insert into tpch_hive_orc.region  select * from region;
insert into tpch_hive_orc.supplier  select * from supplier;
```

#### 4.2.3 查询数据

StarRocks 使用 Catalog 功能查询 Hive 外部表数据，详细操作见 [Hive catalog](../data_source/catalog/hive_catalog.md)。

## 5. 查询 SQL 和建表语句

### 5.1 TPC-H 查询 SQL

```SQL
--Q1
select
  l_returnflag,
  l_linestatus,
  sum(l_quantity) as sum_qty,
  sum(l_extendedprice) as sum_base_price,
select
sum(l_extendedprice * (1 - l_discount)) as sum_disc_price,
sum(l_extendedprice * (1 - l_discount) * (1 + l_tax)) as sum_charge,
avg(l_quantity) as avg_qty,
avg(l_extendedprice) as avg_price,
avg(l_discount) as avg_disc,
count(*) as count_order
from
lineitem
where
l_shipdate <= date '1998-12-01' - interval '90' day
group by
l_returnflag,
l_linestatus
order by
l_returnflag,
l_linestatus;

-- Q2
select
s_acctbal,
s_name,
n_name,
p_partkey,
p_mfgr,
s_address,
s_phone,
s_comment
from
part,
supplier,
partsupp,
nation,
region
where
p_partkey = ps_partkey
and s_suppkey = ps_suppkey
and p_size = 15
and p_type like '%BRASS'
and s_nationkey = n_nationkey
and n_regionkey = r_regionkey
and r_name = 'EUROPE'
and ps_supplycost = ( 
select
min(ps_supplycost)
from
partsupp,
supplier,
nation,
region
where
p_partkey = ps_partkey
and s_suppkey = ps_suppkey
and s_nationkey = n_nationkey
and n_regionkey = r_regionkey
and r_name = 'EUROPE'
)
order by
s_acctbal desc,
n_name,
s_name,
p_partkey
limit 100;

-- Q3
select
l_orderkey,
sum(l_extendedprice * (1 - l_discount)) as revenue,
o_orderdate,
o_shippriority
from
customer,
orders,
lineitem
where
c_mktsegment = 'BUILDING'
and c_custkey = o_custkey
and l_orderkey = o_orderkey
and o_orderdate < date '1995-03-15'
and l_shipdate > date '1995-03-15'
group by
l_orderkey,
o_orderdate,
o_shippriority
order by
revenue desc,
o_orderdate
limit 10;

-- Q4
select
o_orderpriority,
count(*) as order_count
from
orders
where
o_orderdate >= date '1993-07-01'
and o_orderdate < date '1993-07-01' + interval '3' month
and exists (
select
*   
from
lineitem
where
l_orderkey = o_orderkey
and l_commitdate < l_receiptdate
)
group by
o_orderpriority
order by
o_orderpriority;

-- Q5
select
n_name,
sum(l_extendedprice * (1 - l_discount)) as revenue
from
customer,
orders,
lineitem,
supplier,
nation,
region
where
c_custkey = o_custkey
and l_orderkey = o_orderkey
and l_suppkey = s_suppkey
and c_nationkey = s_nationkey
and s_nationkey = n_nationkey
and n_regionkey = r_regionkey
and r_name = 'ASIA'
and o_orderdate >= date '1994-01-01'
and o_orderdate < date '1994-01-01' + interval '1' year
group by
n_name
order by
revenue desc;

-- Q6
select
sum(l_extendedprice * l_discount) as revenue
from
lineitem
where
l_shipdate >= date '1994-01-01'
and l_shipdate < date '1994-01-01' + interval '1' year
and l_discount between .06 - 0.01 and .06 + 0.01
and l_quantity < 24;

-- Q7
select
supp_nation,
cust_nation,
l_year,
sum(volume) as revenue
from
(
select
n1.n_name as supp_nation,
n2.n_name as cust_nation,
extract(year from l_shipdate) as l_year,
l_extendedprice * (1 - l_discount) as volume
from
supplier,
lineitem,
orders,
customer,
nation n1, 
nation n2
where
s_suppkey = l_suppkey
and o_orderkey = l_orderkey
and c_custkey = o_custkey
and s_nationkey = n1.n_nationkey
and c_nationkey = n2.n_nationkey
and (
(n1.n_name = 'FRANCE' and n2.n_name = 'GERMANY')
or (n1.n_name = 'GERMANY' and n2.n_name = 'FRANCE')
)   
and l_shipdate between date '1995-01-01' and date '1996-12-31'
) as shipping
group by
supp_nation,
cust_nation,
l_year
order by
supp_nation,
cust_nation,
l_year;

-- Q8
select
o_year,
sum(case
when nation = 'BRAZIL' then volume
else 0
end) / sum(volume) as mkt_share
from
(
select
extract(year from o_orderdate) as o_year,
l_extendedprice * (1 - l_discount) as volume,
n2.n_name as nation
from
part,
supplier,
lineitem,
orders,
customer,
nation n1, 
nation n2, 
region
where
p_partkey = l_partkey
and s_suppkey = l_suppkey
and l_orderkey = o_orderkey
and o_custkey = c_custkey
and c_nationkey = n1.n_nationkey
and n1.n_regionkey = r_regionkey
and r_name = 'AMERICA'
and s_nationkey = n2.n_nationkey
and o_orderdate between date '1995-01-01' and date '1996-12-31'
and p_type = 'ECONOMY ANODIZED STEEL'
) as all_nations
group by
o_year
order by
o_year;

-- Q9
select
nation,
o_year,
sum(amount) as sum_profit
from
(
select
n_name as nation,
extract(year from o_orderdate) as o_year,
l_extendedprice * (1 - l_discount) - ps_supplycost * l_quantity as amount
from
part,
supplier,
lineitem,
partsupp,
orders,
nation
where
s_suppkey = l_suppkey
and ps_suppkey = l_suppkey
and ps_partkey = l_partkey
and p_partkey = l_partkey
and o_orderkey = l_orderkey
and s_nationkey = n_nationkey
and p_name like '%green%'
) as profit
group by
nation,
o_year
order by
nation,
o_year desc;

-- Q10
select
c_custkey,
c_name,
sum(l_extendedprice * (1 - l_discount)) as revenue,
c_acctbal,
n_name,
c_address,
c_phone,
c_comment
from
customer,
orders,
lineitem,
nation
where
c_custkey = o_custkey
and l_orderkey = o_orderkey
and o_orderdate >= date '1993-10-01'
and o_orderdate < date '1993-10-01' + interval '3' month
and l_returnflag = 'R' 
and c_nationkey = n_nationkey
group by
c_custkey,
c_name,
c_acctbal,
```SQL
# 创建表 customer
drop table if exists customer;
CREATE TABLE customer (
    c_custkey     int NOT NULL,
    c_name        VARCHAR(25) NOT NULL,
    c_address     VARCHAR(40) NOT NULL,
    c_nationkey   int NOT NULL,
    c_phone       VARCHAR(15) NOT NULL,
    c_acctbal     decimal(15, 2)   NOT NULL,
```
```SQL
create database tpch_hive_orc;
use tpch_hive_orc;

-- 创建表 customer
CREATE TABLE `customer`(
  `c_custkey` int, 
  `c_name` varchar(25), 
  `c_address` varchar(40), 
  `c_nationkey` int, 
  `c_phone` varchar(15), 
  `c_acctbal` decimal(15,2), 
  `c_mktsegment` varchar(10) NOT NULL, 
  `c_comment` varchar(117) NOT NULL)
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.ql.io.orc.OrcSerde' 
WITH SERDEPROPERTIES ( 
  'field.delim'='|', 
  'serialization.format'='|') 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.orc.OrcInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.orc.OrcOutputFormat'
LOCATION
  'hdfs://emr-header-1.cluster-49146:9000/user/hive/warehouse/tpch_hive_orc.db/customer';

-- 创建表 lineitem
  CREATE TABLE `lineitem`(
  `l_orderkey` bigint, 
  `l_partkey` int, 
  `l_suppkey` int, 
  `l_linenumber` int, 
  `l_quantity` decimal(15,2), 
  `l_extendedprice` decimal(15,2), 
  `l_discount` decimal(15,2), 
  `l_tax` decimal(15,2), 
  `l_returnflag` varchar(1) NOT NULL, 
  `l_linestatus` varchar(1) NOT NULL, 
  `l_shipdate` date NOT NULL, 
  `l_commitdate` date NOT NULL, 
  `l_receiptdate` date NOT NULL, 
  `l_shipinstruct` varchar(25) NOT NULL, 
  `l_shipmode` varchar(10) NOT NULL, 
  `l_comment` varchar(44) NOT NULL)
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.ql.io.orc.OrcSerde' 
WITH SERDEPROPERTIES ( 
  'field.delim'='|', 
  'serialization.format'='|') 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.orc.OrcInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.orc.OrcOutputFormat'
LOCATION
  'hdfs://emr-header-1.cluster-49146:9000/user/hive/warehouse/tpch_hive_orc.db/lineitem';

-- 创建表 nation
CREATE TABLE `nation`(
  `n_nationkey` int, 
  `n_name` varchar(25) NOT NULL, 
  `n_regionkey` int, 
  `n_comment` varchar(152) NULL)
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.ql.io.orc.OrcSerde' 
WITH SERDEPROPERTIES ( 
  'field.delim'='|', 
  'serialization.format'='|') 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.orc.OrcInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.orc.OrcOutputFormat'
LOCATION
  'hdfs://emr-header-1.cluster-49146:9000/user/hive/warehouse/tpch_hive_orc.db/nation';
```
-- 创建表 orders
CREATE TABLE `orders`(
  `o_orderkey` bigint, 
  `o_custkey` int, 
  `o_orderstatus` varchar(1), 
  `o_totalprice` decimal(15,2), 
  `o_orderdate` date, 
  `o_orderpriority` varchar(15), 
  `o_clerk` varchar(15), 
  `o_shippriority` int, 
  `o_comment` varchar(79))
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.ql.io.orc.OrcSerde' 
WITH SERDEPROPERTIES ( 
  'field.delim'='|', 
  'serialization.format'='|') 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.orc.OrcInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.orc.OrcOutputFormat'
LOCATION
  'hdfs://emr-header-1.cluster-49146:9000/user/hive/warehouse/tpch_hive_orc.db/orders';

-- 创建表 part
CREATE TABLE `part`(
  `p_partkey` int, 
  `p_name` varchar(55), 
  `p_mfgr` varchar(25), 
  `p_brand` varchar(10), 
  `p_type` varchar(25), 
  `p_size` int, 
  `p_container` varchar(10), 
  `p_retailprice` decimal(15,2), 
  `p_comment` varchar(23))
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.ql.io.orc.OrcSerde' 
WITH SERDEPROPERTIES ( 
  'field.delim'='|', 
  'serialization.format'='|') 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.orc.OrcInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.orc.OrcOutputFormat'
LOCATION
  'hdfs://emr-header-1.cluster-49146:9000/user/hive/warehouse/tpch_hive_orc.db/part';

-- 创建表 partsupp
CREATE TABLE `partsupp`(
  `ps_partkey` int, 
  `ps_suppkey` int, 
  `ps_availqty` int, 
  `ps_supplycost` decimal(15,2), 
  `ps_comment` varchar(199))
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.ql.io.orc.OrcSerde' 
WITH SERDEPROPERTIES ( 
  'field.delim'='|', 
  'serialization.format'='|') 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.orc.OrcInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.orc.OrcOutputFormat'
LOCATION
  'hdfs://emr-header-1.cluster-49146:9000/user/hive/warehouse/tpch_hive_orc.db/partsupp';

-- 创建表 region
CREATE TABLE `region`(
  `r_regionkey` int, 
  `r_name` varchar(25), 
  `r_comment` varchar(152))
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.ql.io.orc.OrcSerde' 
WITH SERDEPROPERTIES ( 
  'field.delim'='|', 
  'serialization.format'='|') 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.orc.OrcInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.orc.OrcOutputFormat'
LOCATION
  'hdfs://emr-header-1.cluster-49146:9000/user/hive/warehouse/tpch_hive_orc.db/region';

-- 创建表 supplier
CREATE TABLE `supplier`(
  `s_suppkey` int, 
  `s_name` varchar(25), 
  `s_address` varchar(40), 
  `s_nationkey` int, 
  `s_phone` varchar(15), 
  `s_acctbal` decimal(15,2), 
  `s_comment` varchar(101))
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.ql.io.orc.OrcSerde' 
WITH SERDEPROPERTIES ( 
  'field.delim'='|', 
  'serialization.format'='|') 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.orc.OrcInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.orc.OrcOutputFormat'
LOCATION
  'hdfs://emr-header-1.cluster-49146:9000/user/hive/warehouse/tpch_hive_orc.db/supplier';
```

### 5.4 Hive External Table Creation (CSV Storage Format)

```SQL
create database tpch_hive_csv;
use tpch_hive_csv;

-- Create external table customer
CREATE EXTERNAL TABLE `customer`(
  `c_custkey` int, 
  `c_name` varchar(25), 
  `c_address` varchar(40), 
  `c_nationkey` int, 
  `c_phone` varchar(15), 
  `c_acctbal` double, 
  `c_mktsegment` varchar(10), 
  `c_comment` varchar(117))
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe' 
WITH SERDEPROPERTIES ( 
  'field.delim'='|', 
  'serialization.format'='|') 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://emr-header-1.cluster-49146:9000/user/tmp/csv/customer_csv';
 
-- Create external table lineitem
CREATE EXTERNAL TABLE `lineitem`(
  `l_orderkey` int, 
  `l_partkey` int, 
  `l_suppkey` int, 
  `l_linenumber` int, 
  `l_quantity` double, 
  `l_extendedprice` double, 
  `l_discount` double, 
  `l_tax` double, 
  `l_returnflag` varchar(1), 
  `l_linestatus` varchar(1), 
  `l_shipdate` date, 
  `l_commitdate` date, 
  `l_receiptdate` date, 
  `l_shipinstruct` varchar(25), 
  `l_shipmode` varchar(10), 
  `l_comment` varchar(44))
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe' 
WITH SERDEPROPERTIES ( 
  'field.delim'='|', 
  'serialization.format'='|') 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://emr-header-1.cluster-49146:9000/user/tmp/csv/lineitem_csv';
 

-- Create external table nation
CREATE EXTERNAL TABLE `nation`(
  `n_nationkey` int, 
  `n_name` varchar(25), 
  `n_regionkey` int, 
  `n_comment` varchar(152))
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe' 
WITH SERDEPROPERTIES ( 
  'field.delim'='|', 
  'serialization.format'='|') 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://emr-header-1.cluster-49146:9000/user/tmp/csv/nation_csv';
  
-- Create external table orders
CREATE EXTERNAL TABLE `orders`(
  `o_orderkey` int, 
  `o_custkey` int, 
  `o_orderstatus` varchar(1), 
  `o_totalprice` double, 
  `o_orderdate` date, 
  `o_orderpriority` varchar(15), 
  `o_clerk` varchar(15), 
  `o_shippriority` int, 
  `o_comment` varchar(79))
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe' 
WITH SERDEPROPERTIES ( 
  'field.delim'='|', 
  'serialization.format'='|') 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://emr-header-1.cluster-49146:9000/user/tmp/csv/orders_csv';
  
-- Create external table part
CREATE EXTERNAL TABLE `part`(
  `p_partkey` int, 
  `p_name` varchar(55), 
  `p_mfgr` varchar(25), 
  `p_brand` varchar(10), 
  `p_type` varchar(25), 
  `p_size` int, 
  `p_container` varchar(10), 
  `p_retailprice` double,
```sql
`p_comment` varchar(23))
行格式 SERDE 
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe' 
具有 SERDEPROPERTIES ( 
  'field.delim'='|', 
  'serialization.format'='|') 
存储为 INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://emr-header-1.cluster-49146:9000/user/tmp/csv/part_csv';
  
-- 创建 partsupp 外表
创建外部表 `partsupp`(
  `ps_partkey` int, 
  `ps_suppkey` int, 
  `ps_availqty` int, 
  `ps_supplycost` double, 
  `ps_comment` varchar(199))
行格式 SERDE 
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe' 
具有 SERDEPROPERTIES ( 
  'field.delim'='|', 
  'serialization.format'='|') 
存储为 INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://emr-header-1.cluster-49146:9000/user/tmp/csv/partsupp_csv';
  
-- 创建 region 外表
创建外部表 `region`(
  `r_regionkey` int, 
  `r_name` varchar(25), 
  `r_comment` varchar(152))
行格式 SERDE 
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe' 
具有 SERDEPROPERTIES ( 
  'field.delim'='|', 
  'serialization.format'='|') 
存储为 INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://emr-header-1.cluster-49146:9000/user/tmp/csv/region_csv';
  
-- 创建 supplier 外表
创建外部表 `supplier`(
  `s_suppkey` int, 
  `s_name` varchar(25), 
  `s_address` varchar(40), 
  `s_nationkey` int, 
  `s_phone` varchar(15), 
  `s_acctbal` double, 
  `s_comment` varchar(101))
行格式 SERDE 
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe' 
具有 SERDEPROPERTIES ( 
  'field.delim'='|', 
  'serialization.format'='|') 
存储为 INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://emr-header-1.cluster-49146:9000/user/tmp/csv/supplier_csv';
```