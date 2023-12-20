---
displayed_sidebar: English
---

# TPC-H 基准测试

TPC-H 是由事务处理性能委员会（TPC）开发的决策支持基准。它由一套面向业务的即席查询和并发数据修改组成。TPC-H 可用于基于真实生产环境构建模型，以模拟销售系统的数据仓库。本次测试使用 8 张表，数据大小从 1 GB 到 3 TB。总共测试了 22 个查询，主要性能指标是每个查询的响应时间，即从提交查询到返回结果之间的持续时间。

## 1. 测试结论

我们针对 TPC-H 100 GB 数据集对 22 个查询执行测试。以下图片显示了测试结果。
![comparison](../assets/7.2-1.png)

在测试中，StarRocks 从其本机存储和 Hive 外部表中查询数据。StarRocks 和 Trino 从 Hive 外部表查询相同的数据副本。数据经过 ZLIB 压缩并以 ORC 格式存储。

StarRocks 从本地存储查询数据的延迟为 21 秒，StarRocks 查询 Hive 外部表的延迟为 92 秒，Trino 查询 Hive 外部表的延迟**长达 307 秒**。

## 2. 测试准备

### 2.1 硬件环境

|机器|3 台云主机|
|---|---|
|CPU|16 核|
|内存|64 GB|
|网络带宽|5 Gbit/s|
|磁盘|系统盘：ESSD 40 GB 数据盘：ESSD 100 GB|

### 2.2 软件环境

- 内核版本：Linux 3.10.0-1127.13.1.el7.x86_64
- 操作系统版本：CentOS Linux 发布 7.8.2003
- 软件版本：StarRocks 2.1, Trino-357, Hive-3.1.2

> StarRocks FE 可以单独部署或与 BEs 混合部署，不影响测试结果。

## 3. 测试数据及结果

### 3.1 测试数据

该测试使用 TPC-H 100 GB 数据集。

|表|记录数|StarRocks 上压缩后的数据大小|
|---|---|---|
|customer|1500 万|2.4 GB|
|lineitem|6 亿|75 GB|
|nation|25|2.2 KB|
|orders|1.5 亿|16 GB|
|part|2 亿|2.4 GB|
|partsupp|8 亿|11.6 GB|
|region|5|389 B|
|supplier|100 万|0.14 GB|

### 3.2 测试 SQL

```SQL
--Q1
select
  l_returnflag,
  l_linestatus,
  sum(l_quantity) as sum_qty,
  sum(l_extendedprice) as sum_base_price,
  sum(l_extendedprice * (1 - l_discount)) as sum_disc_price,
  sum(l_extendedprice * (1 - l_discount) * (1 + l_tax)) as sum_charge,
  avg(l_quantity) as avg_qty,
  avg(l_extendedprice) as avg_price,
  avg(l_discount) as avg_disc,
  count(*) as count_order
from lineitem
where l_shipdate <= date '1998-12-01'
group by l_returnflag, l_linestatus
order by l_returnflag, l_linestatus;

--Q2
select s_acctbal, s_name, n_name, p_partkey, p_mfgr, s_address, s_phone, s_comment
from part, partsupp, supplier, nation, region
where
  p_partkey = ps_partkey
  and s_suppkey = ps_suppkey
  and p_size = 15
  and p_type like '%BRASS'
  and s_nationkey = n_nationkey
  and n_regionkey = r_regionkey
  and r_name = 'EUROPE'
  and ps_supplycost = (
    select min(ps_supplycost)
    from partsupp, supplier, nation, region
    where
      p_partkey = ps_partkey
      and s_suppkey = ps_suppkey
      and s_nationkey = n_nationkey
      and n_regionkey = r_regionkey
      and r_name = 'EUROPE'
  )
order by s_acctbal desc, n_name, s_name, p_partkey;

--Q3
select
  l_orderkey, sum(l_extendedprice * (1 - l_discount)) as revenue, o_orderdate, o_shippriority
from
  customer, orders, lineitem
where
  c_mktsegment = 'BUILDING'
  and c_custkey = o_custkey
  and l_orderkey = o_orderkey
  and o_orderdate < date '1995-03-15'
  and l_shipdate > date '1995-03-15'
group by l_orderkey, o_orderdate, o_shippriority
order by revenue desc, o_orderdate;

--Q4
select o_orderpriority, count(*) as order_count
from orders
where
  o_orderdate >= date '1993-07-01'
  and o_orderdate < date '1993-07-01' + interval '3' month
  and exists (
    select * from lineitem
    where l_orderkey = o_orderkey and l_commitdate < l_receiptdate
  )
group by o_orderpriority
order by o_orderpriority;

--Q5
select n_name, sum(l_extendedprice * (1 - l_discount)) as revenue
from customer, orders, lineitem, supplier, nation, region
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
group by n_name
order by revenue desc;

--Q6
select
  sum(l_extendedprice * l_discount) as revenue
from
  lineitem
where
  l_shipdate >= date '1994-01-01'
  and l_shipdate < date '1994-01-01' + interval '1' year
  and l_discount between .06 - 0.01 and .06 + 0.01
  and l_quantity < 24;

--Q7
select supp_nation, cust_nation, l_year, sum(volume) as revenue
from (
    select
      n1.n_name as supp_nation,
      n2.n_name as cust_nation,
      extract(year from l_shipdate) as l_year,
      l_extendedprice * (1 - l_discount) as volume
    from supplier, lineitem, orders, customer, nation n1, nation n2
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
group by supp_nation, cust_nation, l_year
order by supp_nation, cust_nation, l_year;

--Q8
select
  o_year, sum(case when nation = 'BRAZIL' then volume else 0 end) / sum(volume) as mkt_share
from
  (
    select
      extract(year from o_orderdate) as o_year,
      l_extendedprice * (1 - l_discount) as volume,
      n2.n_name as nation
    from part, supplier, lineitem, orders, customer, nation n1, nation n2, region
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
group by o_year
order by o_year;

--Q9
select nation, o_year, sum(amount) as sum_profit
from
  (
    select
      n_name as nation,
      extract(year from o_orderdate) as o_year,
      l_extendedprice * (1 - l_discount) - ps_supplycost * l_quantity as amount
    from part, supplier, lineitem, partsupp, orders, nation
    where
      s_suppkey = l_suppkey
      and ps_suppkey = l_suppkey
      and ps_partkey = l_partkey
      and p_partkey = l_partkey
      and o_orderkey = l_orderkey
      and s_nationkey = n_nationkey
      and p_name like '%green%'
  ) as profit
group by nation, o_year
order by nation, o_year desc;

--Q10
select c_custkey, c_name, sum(l_extendedprice * (1 - l_discount)) as revenue, c_acctbal, n_name, c_address, c_phone, c_comment
from customer, orders, lineitem, nation
where
  c_custkey = o_custkey
  and l_orderkey = o_orderkey
  and o_orderdate >= date '1993-10-01'
  and o_orderdate < date '1993-12-01'
  and l_returnflag = 'R'
  and c_nationkey = n_nationkey
group by c_custkey, c_name, c_acctbal, c_phone, n_name, c_address, c_comment
order by revenue desc;

--Q11
select ps_partkey, sum(ps_supplycost * ps_availqty) as value
from partsupp, supplier, nation
where
  ps_suppkey = s_suppkey
  and s_nationkey = n_nationkey
  and n_name = 'GERMANY'
group by ps_partkey having
    sum(ps_supplycost * ps_availqty) > (
      select sum(ps_supplycost * ps_availqty) * 0.0001000000
      from partsupp, supplier, nation
      where
        ps_suppkey = s_suppkey
        and s_nationkey = n_nationkey
        and n_name = 'GERMANY'
    )
order by value desc;

--Q12
select
  l_shipmode,
  sum(case when o_orderpriority = '1-URGENT' or o_orderpriority = '2-HIGH' then 1 else 0 end) as high_line_count,
  sum(case when o_orderpriority <> '1-URGENT' and o_orderpriority <> '2-HIGH' then 1 else 0 end) as low_line_count
from orders, lineitem
where
  o_orderkey = l_orderkey
  and l_shipmode in ('MAIL', 'SHIP')
  and l_commitdate < l_receiptdate
  and l_shipdate < l_commitdate
  and l_receiptdate >= date '1994-01-01'
  and l_receiptdate < date '1994-01-01' + interval '1' year
group by l_shipmode
order by l_shipmode;

--Q13
select c_count, count(*) as custdist
from (select c_custkey, count(o_orderkey) as c_count
    from customer left outer join orders on c_custkey = o_custkey and o_comment not like '%special%requests%'
    group by c_custkey
) as c_orders
group by c_count
order by custdist desc, c_count desc;

--Q14
select
  100.00 * sum(case when p_type like 'PROMO%' then l_extendedprice * (1 - l_discount) else 0 end) / sum(l_extendedprice * (1 - l_discount)) as promo_revenue
from lineitem, part
where
  l_partkey = p_partkey
  and l_shipdate >= date '1995-09-01'
  and l_shipdate < date '1995-09-01' + interval '1' month;

--Q15
select s_suppkey, s_name, s_address, s_phone, total_revenue
from supplier, revenue0
where
  s_suppkey = supplier_no
  and total_revenue = (
    select max(total_revenue) from revenue0
  )
order by s_suppkey;

--Q16
```
```SQL
--Q16
select p_brand, p_type, p_size, count(distinct ps_suppkey) as supplier_cnt
from partsupp, part
where
  p_partkey = ps_partkey
  and p_brand <> 'Brand#45'
  and p_type not like 'MEDIUM POLISHED%'
  and p_size in (49, 14, 23, 45, 19, 3, 36, 9)
  and ps_suppkey not in (
    select s_suppkey from supplier where s_comment like '%Customer%Complaints%'
  )
group by p_brand, p_type, p_size
order by supplier_cnt desc, p_brand, p_type, p_size;

--Q17
select sum(l_extendedprice) / 7.0 as avg_yearly
from lineitem, part
where
  p_partkey = l_partkey
  and p_brand = 'Brand#23'
  and p_container = 'MED BOX'
  and l_quantity < (
    select 0.2 * avg(l_quantity) from lineitem where l_partkey = p_partkey
  );

--Q18
select c_name, c_custkey, o_orderkey, o_orderdate, o_totalprice, sum(l_quantity)
from customer, orders, lineitem
where
  o_orderkey in (
    select l_orderkey from lineitem
    group by l_orderkey having
        sum(l_quantity) > 300
  )
  and c_custkey = o_custkey
  and o_orderkey = l_orderkey
group by c_name, c_custkey, o_orderkey, o_orderdate, o_totalprice
order by o_totalprice desc, o_orderdate;

--Q19
select sum(l_extendedprice * (1 - l_discount)) as revenue
from lineitem, part
where
  (
    p_partkey = l_partkey
    and p_brand = 'Brand#12'
    and p_container in ('SM CASE', 'SM BOX', 'SM PACK', 'SM PKG')
    and l_quantity >= 1 and l_quantity <= 1 + 10
    and p_size between 1 and 5
    and l_shipmode in ('AIR', 'AIR REG')
    and l_shipinstruct = 'DELIVER IN PERSON'
  )
  or
  (
    p_partkey = l_partkey
    and p_brand = 'Brand#23'
    and p_container in ('MED BAG', 'MED BOX', 'MED PKG', 'MED PACK')
    and l_quantity >= 10 and l_quantity <= 10 + 10
    and p_size between 1 and 10
    and l_shipmode in ('AIR', 'AIR REG')
    and l_shipinstruct = 'DELIVER IN PERSON'
  )
  or
  (
    p_partkey = l_partkey
    and p_brand = 'Brand#34'
    and p_container in ('LG CASE', 'LG BOX', 'LG PACK', 'LG PKG')
    and l_quantity >= 20 and l_quantity <= 20 + 10
    and p_size between 1 and 15
    and l_shipmode in ('AIR', 'AIR REG')
    and l_shipinstruct = 'DELIVER IN PERSON'
  );

--Q20
select s_name, s_address
from supplier, nation
where
  s_suppkey in (
    select ps_suppkey
    from partsupp
    where
      ps_partkey in (
        select p_partkey from part
        where p_name like 'forest%'
      )
      and ps_availqty > (
        select 0.5 * sum(l_quantity)
        from lineitem
        where
          l_partkey = ps_partkey
          and l_suppkey = ps_suppkey
          and l_shipdate >= date '1994-01-01'
          and l_shipdate < date '1994-01-01' + interval '1' year
      )
  )
  and s_nationkey = n_nationkey
  and n_name = 'CANADA'
order by s_name;

--Q21
select s_name, count(*) as numwait
from  supplier,  lineitem l1,  orders,  nation
where
  s_suppkey = l1.l_suppkey
  and o_orderkey = l1.l_orderkey
  and o_orderstatus = 'F'
  and l1.l_receiptdate > l1.l_commitdate
  and exists (
    select * from  lineitem l2
    where
      l2.l_orderkey = l1.l_orderkey
      and l2.l_suppkey <> l1.l_suppkey
  )
  and not exists (
    select * from  lineitem l3
    where
      l3.l_orderkey = l1.l_orderkey
      and l3.l_suppkey <> l1.l_suppkey
      and l3.l_receiptdate > l3.l_commitdate
  )
  and s_nationkey = n_nationkey
  and n_name = 'SAUDI ARABIA'
group by s_name
order by  numwait desc, s_name;

--Q22
select
  cntrycode,
  count(*) as numcust,
  sum(c_acctbal) as totacctbal
from
  (
    select
      substr(c_phone, 1, 2) as cntrycode,
      c_acctbal
    from
      customer
    where
      substr(c_phone, 1, 2) in
        ('13', '31', '23', '29', '30', '18', '17')
      and c_acctbal > (
        select
          avg(c_acctbal)
        from
          customer
        where
          c_acctbal > 0.00
          and substr(c_phone, 1, 2) in
            ('13', '31', '23', '29', '30', '18', '17')
      )
      and not exists (
        select
          *
        from
          orders
        where
          o_custkey = c_custkey
      )
  ) as custsale
group by
  cntrycode
order by
  cntrycode;
```

### 3.3 测试结果

|**查询**|**StarRocks-native (ms)**|**StarRocks-Hive 外部 (ms)**|**Trino (ms)**|
|---|---|---|---|
|q1|1291|5349|14350|
|q2|257|940|4075|
|q3|690|2585|7615|
|q4|529|2514|9385|
|q5|1142|5844|15350|
|q6|115|2936|4045|
|q7|1435|5128|8760|
|q8|855|5989|9535|
|q9|3028|5507|27310|
|q10|1381|3654|6560|
|q11|265|691|2090|
|q12|387|3827|7635|
|q13|2165|3733|14530|
|q14|184|3824|4355|
|q15|354|7321|8975|
|q16|240|1029|2265|
|q17|753|5011|46405|
|q18|2909|7032|30975|
|q19|702|3905|7315|
|q20|963|4623|12500|
|q21|870|10016|58340|
|q22|545|991|5605|
|总和|21060|92449|307975|

## 4. 测试程序

### 4.1 查询StarRocks原生表

#### 4.1.1 生成数据

1. 下载tpch-poc工具包并编译。

   ```Bash
   wget https://starrocks-public.oss-cn-zhangjiakou.aliyuncs.com/tpch-poc-0.1.2.zip
   unzip tpch-poc-0.1.2.zip
   cd tpch-poc-0.1.2
   cd benchmark 
   ```

   相关工具存放在bin目录下，您可以在home目录下发起操作。

2. 生成数据。

   ```Bash
   # 在 `data_100` 目录下生成100 GB数据。
   ./bin/gen_data/gen-tpch.sh 100 data_100
   ```

#### 4.1.2 创建表结构

1. 修改配置文件 `conf/starrocks.conf`，指定集群地址。

   ```SQL
   # StarRocks配置
   mysql_host: 192.168.1.1
   mysql_port: 9030
   mysql_user: root
   mysql_password:
   database: tpch
   
   # 集群端口
   http_port: 8030
   be_heartbeat_port: 9050
   broker_port: 8000
   
   # parallel_fragment_exec_instance_num 指定并行度。
   # 推荐的并行度是集群CPU核心数的一半。以下示例使用8。
   parallel_num: 8
   ```

2. 运行以下脚本创建表：

   ```SQL
   # 为100GB数据创建表
   ./bin/create_db_table.sh ddl_100
   ```

   表已创建，桶号已默认指定。您可以删除之前的表，根据您的集群规模和节点配置重新规划桶数，以获得更好的测试结果。

   ```SQL
   # 创建名为 customer 的表。
   drop table if exists customer;
   CREATE TABLE customer (
       c_custkey     int NOT NULL,
       c_name        VARCHAR(25) NOT NULL,
       c_address     VARCHAR(40) NOT NULL,
       c_nationkey   int NOT NULL,
       c_phone       VARCHAR(15) NOT NULL,
       c_acctbal     decimal(15, 2)   NOT NULL,
       c_mktsegment  VARCHAR(10) NOT NULL,
       c_comment     VARCHAR(117) NOT NULL
   )ENGINE=OLAP
   DUPLICATE KEY(`c_custkey`)
   COMMENT "OLAP"
   DISTRIBUTED BY HASH(`c_custkey`) BUCKETS 24
   PROPERTIES (
       "replication_num" = "1",
       "storage_format" = "DEFAULT"
   );
   # 创建名为 lineitem 的表。
   drop table if exists lineitem;
   CREATE TABLE lineitem ( 
       l_shipdate    DATE NOT NULL,
       l_orderkey    int NOT NULL,
       l_linenumber  int not null,
       l_partkey     int NOT NULL,
       l_suppkey     int not null,
       l_quantity    decimal(15, 2) NOT NULL,
       l_extendedprice  decimal(15, 2) NOT NULL,
       l_discount    decimal(15, 2) NOT NULL,
       l_tax         decimal(15, 2) NOT NULL,
       l_returnflag  VARCHAR(1) NOT NULL,
       l_linestatus  VARCHAR(1) NOT NULL,
       l_commitdate  DATE NOT NULL,
       l_receiptdate DATE NOT NULL,
       l_shipinstruct VARCHAR(25) NOT NULL,
       l_shipmode     VARCHAR(10) NOT NULL,
       l_comment      VARCHAR(44) NOT NULL
   )ENGINE=OLAP
   DUPLICATE KEY(`l_shipdate`, `l_orderkey`)
   COMMENT "OLAP"
   DISTRIBUTED BY HASH(`l_orderkey`) BUCKETS 96
   PROPERTIES (
       "replication_num" = "1",
       "storage_format" = "DEFAULT",
       "colocate_with" = "tpch2"
   );
   # 创建名为 nation 的表。
   drop table if exists nation;
   CREATE TABLE `nation` (
   `n_nationkey` int(11) NOT NULL,
   `n_name`      varchar(25) NOT NULL,
   `n_regionkey` int(11) NOT NULL,
   `n_comment`   varchar(152) NULL
   ) ENGINE=OLAP
   DUPLICATE KEY(`N_NATIONKEY`)
   COMMENT "OLAP"
   DISTRIBUTED BY HASH(`N_NATIONKEY`) BUCKETS 1
   PROPERTIES (
       "replication_num" = "3",
       "storage_format" = "DEFAULT"
   );
   # 创建名为 orders 的表。
   drop table if exists orders;
   CREATE TABLE orders  (
       o_orderkey       int NOT NULL,
       o_orderdate      DATE NOT NULL,
       o_custkey        int NOT NULL,
       o_orderstatus    VARCHAR(1) NOT NULL,
       o_totalprice     decimal(15, 2) NOT NULL,
       o_orderpriority  VARCHAR(15) NOT NULL,
       o_clerk          VARCHAR(15) NOT NULL,
       o_shippriority   int NOT NULL,
       o_comment        VARCHAR(79) NOT NULL
   )ENGINE=OLAP
   DUPLICATE KEY(`o_orderkey`, `o_orderdate`)
   COMMENT "OLAP"
   DISTRIBUTED BY HASH(`o_orderkey`) BUCKETS 96
   PROPERTIES (
       "replication_num" = "1",
       "storage_format" = "DEFAULT",
       "colocate_with" = "tpch2"
   );
   # 创建名为 part 的表。
   drop table if exists part;
   CREATE TABLE part (
       p_partkey          int NOT NULL,
       p_name        VARCHAR(55) NOT NULL,
       p_mfgr        VARCHAR(25) NOT NULL,
       p_brand       VARCHAR(10) NOT NULL,
       p_type        VARCHAR(25) NOT NULL,
       p_size        int NOT NULL,
       p_container   VARCHAR(10) NOT NULL,
       p_retailprice decimal(15, 2) NOT NULL,
       p_comment     VARCHAR(23) NOT NULL
   )ENGINE=OLAP
   DUPLICATE KEY(`p_partkey`)
   COMMENT "OLAP"
   DISTRIBUTED BY HASH(`p_partkey`) BUCKETS 24
   PROPERTIES (
       "replication_num" = "1",
       "storage_format" = "DEFAULT",
       "colocate_with" = "tpch2p"
   );
   # 创建名为 partsupp 的表。
   drop table if exists partsupp;
   CREATE TABLE partsupp ( 
       ps_partkey          int NOT NULL,
       ps_suppkey     int NOT NULL,
       ps_availqty    int NOT NULL,
       ps_supplycost  decimal(15, 2)  NOT NULL,
       ps_comment     VARCHAR(199) NOT NULL
   )ENGINE=OLAP
   DUPLICATE KEY(`ps_partkey`)
   COMMENT "OLAP"
   DISTRIBUTED BY HASH(`ps_partkey`) BUCKETS 24
   PROPERTIES (
       "replication_num" = "1",
       "storage_format" = "DEFAULT",
       "colocate_with" = "tpch2p"
   );
   # 创建名为 region 的表。
   drop table if exists region;
   CREATE TABLE region  ( 
       r_regionkey      int NOT NULL,
       r_name       VARCHAR(25) NOT NULL,
       r_comment    VARCHAR(152)
   )ENGINE=OLAP
   DUPLICATE KEY(`r_regionkey`)
   COMMENT "OLAP"
   DISTRIBUTED BY HASH(`r_regionkey`) BUCKETS 1
   PROPERTIES (
       "replication_num" = "3",
       "storage_format" = "DEFAULT"
   );
   # 创建名为 supplier 的表。
   drop table if exists supplier;
   CREATE TABLE supplier (  
       s_suppkey       int NOT NULL,
       s_name        VARCHAR(25) NOT NULL,
       s_address     VARCHAR(40) NOT NULL,
       s_nationkey   int NOT NULL,
       s_phone       VARCHAR(15) NOT NULL,
       s_acctbal     decimal(15, 2) NOT NULL,
       s_comment     VARCHAR(101) NOT NULL
   )ENGINE=OLAP
   DUPLICATE KEY(`s_suppkey`)
   COMMENT "OLAP"
   DISTRIBUTED BY HASH(`s_suppkey`) BUCKETS 12
   PROPERTIES (
       "replication_num" = "1",
       "storage_format" = "DEFAULT"
   );
   drop view if exists revenue0;
   create view revenue0 (supplier_no, total_revenue) as
   select
       l_suppkey,
       sum(l_extendedprice * (1 - l_discount))
   from
       lineitem
   where
       l_shipdate >= date '1996-01-01'
       and l_shipdate < date '1996-01-01' + interval '3' month
   group by
       l_suppkey;
   ```
```markdown
   PROPERTIES (
       "replication_num" = "1",
       "storage_format" = "DEFAULT",
       "colocate_with" = "tpch2p"
   );
   # 创建名为 partsupp 的表。
   drop table if exists partsupp;
   CREATE TABLE partsupp ( 
       ps_partkey          int NOT NULL,
       ps_suppkey     int NOT NULL,
       ps_availqty    int NOT NULL,
       ps_supplycost  decimal(15, 2)  NOT NULL,
       ps_comment     VARCHAR(199) NOT NULL
   )ENGINE=OLAP
   DUPLICATE KEY(`ps_partkey`)
   COMMENT "OLAP"
   DISTRIBUTED BY HASH(`ps_partkey`) BUCKETS 24
   PROPERTIES (
       "replication_num" = "1",
       "storage_format" = "DEFAULT",
       "colocate_with" = "tpch2p"
   );
   # 创建名为 region 的表。
   drop table if exists region;
   CREATE TABLE region  ( 
       r_regionkey      int NOT NULL,
       r_name       VARCHAR(25) NOT NULL,
       r_comment    VARCHAR(152)
   )ENGINE=OLAP
   DUPLICATE KEY(`r_regionkey`)
   COMMENT "OLAP"
   DISTRIBUTED BY HASH(`r_regionkey`) BUCKETS 1
   PROPERTIES (
       "replication_num" = "3",
       "storage_format" = "DEFAULT"
   );
   # 创建名为 supplier 的表。
   drop table if exists supplier;
   CREATE TABLE supplier (  
       s_suppkey       int NOT NULL,
       s_name        VARCHAR(25) NOT NULL,
       s_address     VARCHAR(40) NOT NULL,
       s_nationkey   int NOT NULL,
       s_phone       VARCHAR(15) NOT NULL,
       s_acctbal     decimal(15, 2) NOT NULL,
       s_comment     VARCHAR(101) NOT NULL
   )ENGINE=OLAP
   DUPLICATE KEY(`s_suppkey`)
   COMMENT "OLAP"
   DISTRIBUTED BY HASH(`s_suppkey`) BUCKETS 12
   PROPERTIES (
       "replication_num" = "1",
       "storage_format" = "DEFAULT"
   );
   drop view if exists revenue0;
   create view revenue0 (supplier_no, total_revenue) as
   select
       l_suppkey,
       sum(l_extendedprice * (1 - l_discount))
   from
       lineitem
   where
       l_shipdate >= date '1996-01-01'
       and l_shipdate < date '1996-01-01' + interval '3' month
   group by
       l_suppkey;
   ```

#### 4.1.3 加载数据

```Python
./bin/stream_load.sh data_100
```

#### 4.1.4 收集统计信息（可选）

此步骤是可选的。我们建议收集完整的统计信息以生成更好的测试结果。连接到 StarRocks 集群后运行以下命令：

```SQL
ANALYZE FULL TABLE customer;
ANALYZE FULL TABLE lineitem;
ANALYZE FULL TABLE nation;
ANALYZE FULL TABLE orders;
ANALYZE FULL TABLE part;
ANALYZE FULL TABLE partsupp;
ANALYZE FULL TABLE region;
ANALYZE FULL TABLE supplier;
```

#### 4.1.5 查询数据

```Python
./bin/benchmark.sh -p -d tpch
```

### 4.2 查询 Hive 外部表

#### 4.2.1 创建表架构

在 Hive 中创建表架构。存储格式为 ORC。

```SQL
create database tpch_hive_orc;
use tpch_hive_orc;
-- 创建名为 customer 的表。
CREATE TABLE `customer`(
  `c_custkey` int, 
  `c_name` varchar(25), 
  `c_address` varchar(40), 
  `c_nationkey` int, 
  `c_phone` varchar(15), 
  `c_acctbal` decimal(15,2), 
  `c_mktsegment` varchar(10), 
  `c_comment` varchar(117))
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
-- 创建名为 lineitem 的表。
  CREATE TABLE `lineitem`(
  `l_orderkey` bigint, 
  `l_partkey` int, 
  `l_suppkey` int, 
  `l_linenumber` int, 
  `l_quantity` decimal(15,2), 
  `l_extendedprice` decimal(15,2), 
  `l_discount` decimal(15,2), 
  `l_tax` decimal(15,2), 
  `l_returnflag` varchar(1), 
  `l_linestatus` varchar(1), 
  `l_shipdate` date, 
  `l_commitdate` date, 
  `l_receiptdate` date, 
  `l_shipinstruct` varchar(25), 
  `l_shipmode` varchar(10), 
  `l_comment` varchar(44))
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
-- 创建名为 nation 的表。
CREATE TABLE `nation`(
  `n_nationkey` int, 
  `n_name` varchar(25), 
  `n_regionkey` int, 
  `n_comment` varchar(152))
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
-- 创建名为 orders 的表。
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
-- 创建名为 part 的表。
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
-- 创建名为 partsupp 的表。
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
-- 创建名为 region 的表。
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
-- 创建名为 supplier 的表。
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

在 StarRocks 中创建外部表。

```SQL
create database if not exists tpch_sr;
use tpch_sr;
CREATE EXTERNAL RESOURCE "hive_test" PROPERTIES (
  "type" = "hive",
  "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
);
CREATE EXTERNAL TABLE IF NOT EXISTS nation (
    n_nationkey int,
    n_name VARCHAR(25),
    n_regionkey int,
    n_comment varchar(152))
ENGINE=hive
properties (
    "resource" = "hive_test",
    "table" = "nation",
    "database" = "tpch_hive_orc"
);
CREATE EXTERNAL TABLE IF NOT EXISTS customer (
    c_custkey     INTEGER,
    c_name        VARCHAR(25),
    c_address     VARCHAR(40),
    c_nationkey   INTEGER,
    c_phone       VARCHAR(15),
    c_acctbal     DECIMAL(15,2),
    c_mktsegment  VARCHAR(10),
    c_comment     VARCHAR(117))
ENGINE=hive
properties (
    "resource" = "hive_test",
    "table" = "customer",
    "database" = "tpch_hive_orc"
);
CREATE EXTERNAL TABLE IF NOT EXISTS lineitem (
    l_orderkey    BIGINT,
    l_partkey     INTEGER,
    l_suppkey     INTEGER,
    l_linenumber  INTEGER,
    l_quantity    DECIMAL(15,2),
    l_extendedprice  DECIMAL(15,2),
    l_discount    DECIMAL(15,2),
    l_tax         DECIMAL(15,2),
    l_returnflag  VARCHAR(1),
    l_linestatus  VARCHAR(1),
    l_shipdate    DATE,
    l_commitdate  DATE,
    l_receiptdate DATE,
    l_shipinstruct VARCHAR(25),
    l_shipmode     VARCHAR(10),
    l_comment      VARCHAR(44))
ENGINE=hive
properties (
    "resource" = "hive_test",
    "table" = "lineitem",
    "database" = "tpch_hive_orc"
);
CREATE EXTERNAL TABLE IF NOT EXISTS orders  ( 
    o_orderkey       BIGINT,
    o_custkey        INTEGER,
    o_orderstatus    VARCHAR(1),
```
```SQL
CREATE EXTERNAL TABLE IF NOT EXISTS orders  ( 
    o_orderkey     INTEGER,
    o_custkey      INTEGER,
    o_orderstatus  VARCHAR(1),
    o_totalprice   DECIMAL(15,2),
    o_orderdate    DATE,
    o_orderpriority VARCHAR(15),
    o_clerk        VARCHAR(15),
    o_shippriority INTEGER,
    o_comment      VARCHAR(79)
) ENGINE=hive
PROPERTIES (
    "resource" = "hive_test",
    "table" = "orders",
    "database" = "tpch_hive_orc"
);
CREATE EXTERNAL TABLE IF NOT EXISTS part  ( 
    p_partkey     INTEGER,
    p_name        VARCHAR(55),
    p_mfgr        VARCHAR(25),
    p_brand       VARCHAR(10),
    p_type        VARCHAR(25),
    p_size        INTEGER,
    p_container   VARCHAR(10),
    p_retailprice DECIMAL(15,2),
    p_comment     VARCHAR(23)
) ENGINE=hive
PROPERTIES (
    "resource" = "hive_test",
    "table" = "part",
    "database" = "tpch_hive_orc"
);
CREATE EXTERNAL TABLE IF NOT EXISTS partsupp ( 
    ps_partkey     INTEGER,
    ps_suppkey     INTEGER,
    ps_availqty    INTEGER,
    ps_supplycost  DECIMAL(15,2),
    ps_comment     VARCHAR(199)
) ENGINE=hive
PROPERTIES (
    "resource" = "hive_test",
    "table" = "partsupp",
    "database" = "tpch_hive_orc"
);
CREATE EXTERNAL TABLE IF NOT EXISTS region  ( 
    r_regionkey  INTEGER,
    r_name       VARCHAR(25),
    r_comment    VARCHAR(152)
) ENGINE=hive
PROPERTIES (
    "resource" = "hive_test",
    "table" = "region",
    "database" = "tpch_hive_orc"
);
CREATE EXTERNAL TABLE IF NOT EXISTS supplier (
    s_suppkey     INTEGER,
    s_name        VARCHAR(25),
    s_address     VARCHAR(40),
    s_nationkey   INTEGER,
    s_phone       VARCHAR(15),
    s_acctbal     DECIMAL(15,2),
    s_comment     VARCHAR(101)
) ENGINE=hive
PROPERTIES (
    "resource" = "hive_test",
    "table" = "supplier",
    "database" = "tpch_hive_orc"
);
CREATE VIEW revenue0 (supplier_no, total_revenue) AS
SELECT
    l_suppkey,
    SUM(l_extendedprice * (1 - l_discount))
FROM
    lineitem
WHERE
    l_shipdate >= DATE '1996-01-01'
    AND l_shipdate < DATE '1996-01-01' + INTERVAL '3' MONTH
GROUP BY
    l_suppkey;
```

#### 4.2.2 加载数据

1. 在 Hive 中创建 Hive 外部表。表格格式为 CSV。将 CSV 测试数据上传到 HDFS 上的数据存储目录。本例中 Hive 外部表的 HDFS 数据存储路径为 /user/tmp/csv/。

   ```SQL
   CREATE DATABASE tpch_hive_csv;
   USE tpch_hive_csv;
   
   -- 创建名为 customer 的外部表。
   CREATE EXTERNAL TABLE `customer`(
   `c_custkey` INT, 
   `c_name` VARCHAR(25), 
   `c_address` VARCHAR(40), 
   `c_nationkey` INT, 
   `c_phone` VARCHAR(15), 
   `c_acctbal` DOUBLE, 
   `c_mktsegment` VARCHAR(10), 
   `c_comment` VARCHAR(117))
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
   -- 创建名为 lineitem 的外部表。
   CREATE EXTERNAL TABLE `lineitem`(
   `l_orderkey` INT, 
   `l_partkey` INT, 
   `l_suppkey` INT, 
   `l_linenumber` INT, 
   `l_quantity` DOUBLE, 
   `l_extendedprice` DOUBLE, 
   `l_discount` DOUBLE, 
   `l_tax` DOUBLE, 
   `l_returnflag` VARCHAR(1), 
   `l_linestatus` VARCHAR(1), 
   `l_shipdate` DATE, 
   `l_commitdate` DATE, 
   `l_receiptdate` DATE, 
   `l_shipinstruct` VARCHAR(25), 
   `l_shipmode` VARCHAR(10), 
   `l_comment` VARCHAR(44))
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
   -- 创建名为 nation 的外部表。
   CREATE EXTERNAL TABLE `nation`(
   `n_nationkey` INT, 
   `n_name` VARCHAR(25), 
   `n_regionkey` INT, 
   `n_comment` VARCHAR(152))
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
   -- 创建名为 orders 的外部表。
   CREATE EXTERNAL TABLE `orders`(
   `o_orderkey` INT, 
   `o_custkey` INT, 
   `o_orderstatus` VARCHAR(1), 
   `o_totalprice` DOUBLE, 
   `o_orderdate` DATE, 
   `o_orderpriority` VARCHAR(15), 
   `o_clerk` VARCHAR(15), 
   `o_shippriority` INT, 
   `o_comment` VARCHAR(79))
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
   -- 创建名为 part 的外部表。
   CREATE EXTERNAL TABLE `part`(
   `p_partkey` INT, 
   `p_name` VARCHAR(55), 
   `p_mfgr` VARCHAR(25), 
   `p_brand` VARCHAR(10), 
   `p_type` VARCHAR(25), 
   `p_size` INT, 
   `p_container` VARCHAR(10), 
   `p_retailprice` DOUBLE, 
   `p_comment` VARCHAR(23))
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
   'hdfs://emr-header-1.cluster-49146:9000/user/tmp/csv/part_csv';
   -- 创建名为 partsupp 的外部表。
   CREATE EXTERNAL TABLE `partsupp`(
   `ps_partkey` INT, 
   `ps_suppkey` INT, 
   `ps_availqty` INT, 
   `ps_supplycost` DOUBLE, 
   `ps_comment` VARCHAR(199))
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
   'hdfs://emr-header-1.cluster-49146:9000/user/tmp/csv/partsupp_csv';
   -- 创建名为 region 的外部表。
   CREATE EXTERNAL TABLE `region`(
   `r_regionkey` INT, 
   `r_name` VARCHAR(25), 
   `r_comment` VARCHAR(152))
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
   'hdfs://emr-header-1.cluster-49146:9000/user/tmp/csv/region_csv';
   -- 创建名为 supplier 的外部表。
   CREATE EXTERNAL TABLE `supplier`(
   `s_suppkey` INT, 
   `s_name` VARCHAR(25), 
   `s_address` VARCHAR(40), 
   `s_nationkey` INT, 
   `s_phone` VARCHAR(15), 
   `s_acctbal` DOUBLE, 
   `s_comment` VARCHAR(101))
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
   'hdfs://emr-header-1.cluster-49146:9000/user/tmp/csv/supplier_csv';
   ```

2. 将每个 Hive 外部表中的 CSV 数据传输到 ORC 格式的 Hive 表中。这有助于在 StarRocks 和 Trino 中创建 ORC 格式的 Hive 外部表以供后续测试。

   ```SQL
   USE tpch_hive_csv;
   INSERT INTO tpch_hive_orc.customer SELECT * FROM customer;
   INSERT INTO tpch_hive_orc.lineitem SELECT * FROM lineitem;
   INSERT INTO tpch_hive_orc.nation SELECT * FROM nation;
   INSERT INTO tpch_hive_orc.orders SELECT * FROM orders;
   INSERT INTO tpch_hive_orc.part SELECT * FROM part;
   INSERT INTO tpch_hive_orc.partsupp SELECT * FROM partsupp;
   INSERT INTO tpch_hive_orc.region SELECT * FROM region;
   INSERT INTO tpch_hive_orc.supplier SELECT * FROM supplier;
   ```

#### 4.2.3 查询数据

```SQL
USE tpch_sr;
-- 查询当前会话的系统变量。
SHOW VARIABLES;
-- 将并行度设置为 8。
SET parallel_fragment_exec_instance_num = 8;

--Q1
SELECT
  l_returnflag,
  l_linestatus,
  SUM(l_quantity) AS sum_qty,
  SUM(l_extendedprice) AS sum_base_price,
  SUM(l_extendedprice * (1 - l_discount)) AS sum_disc_price,
  SUM(l_extendedprice * (1 - l_discount) * (1 + l_tax)) AS sum_charge,
  AVG(l_quantity) AS avg_qty,
  AVG(l_extendedprice) AS avg_price,
  AVG(l_discount) AS avg_disc,
  COUNT(*) AS count_order
FROM lineitem
WHERE l_shipdate <= DATE '1998-12-01'
GROUP BY l_returnflag, l_linestatus
ORDER BY l_returnflag, l_linestatus;

--Q2
SELECT s_acctbal, s_name, n_name, p_partkey, p_mfgr, s_address, s_phone, s_comment
FROM part, partsupp, supplier, nation, region
WHERE
  p_partkey = ps_partkey
  AND s_suppkey = ps_suppkey
  AND p_size = 15
  AND p_type LIKE '%BRASS'
  AND s_nationkey = n_nationkey
  AND n_regionkey = r_regionkey
  AND r_name = 'EUROPE'
  AND ps_supplycost = (
    SELECT MIN(ps_supplycost)
    FROM partsupp, supplier, nation, region
    WHERE
      p_partkey = ps_partkey
      AND s_suppkey = ps_suppkey
      AND s_nationkey = n_nationkey
      AND n_regionkey = r_regionkey
      AND r_name = 'EUROPE'
  )
ORDER BY s_acctbal DESC, n_name, s_name, p_partkey;

--Q3
SELECT
  l_orderkey, SUM(l_extendedprice * (1 - l_discount)) AS revenue, o_orderdate, o_shippriority
FROM
  customer, orders, lineitem
WHERE
  c_mktsegment = 'BUILDING'
  AND c_custkey = o_custkey
  AND l_orderkey = o_orderkey
```
--Q3
选择
  l_orderkey，sum(l_extendedprice * (1 - l_discount)) as revenue，o_orderdate，o_shippriority
来自
  customer，orders，lineitem
哪里
  c_mktsegment = 'BUILDING'
  和 c_custkey = o_custkey
  和 l_orderkey = o_orderkey
  和 o_orderdate < date '1995-03-15'
  和 l_shipdate > date '1995-03-15'
按 l_orderkey，o_orderdate，o_shippriority 分组
按收入降序，o_orderdate 排序；

--Q4
选择 o_orderpriority，count(*) as order_count
来自 orders
哪里
  o_orderdate >= date '1993-07-01'
  和 o_orderdate < date '1993-07-01' + interval '3' month
  和存在 (
    选择 * 来自 lineitem
    哪里 l_orderkey = o_orderkey 和 l_commitdate < l_receiptdate
  )
按 o_orderpriority 分组
按 o_orderpriority 排序；

--Q5
选择 n_name，sum(l_extendedprice * (1 - l_discount)) as revenue
来自 customer，orders，lineitem，supplier，nation，region
哪里
  c_custkey = o_custkey
  和 l_orderkey = o_orderkey
  和 l_suppkey = s_suppkey
  和 c_nationkey = s_nationkey
  和 s_nationkey = n_nationkey
  和 n_regionkey = r_regionkey
  和 r_name = 'ASIA'
  和 o_orderdate >= date '1994-01-01'
  和 o_orderdate < date '1994-01-01' + interval '1' year
按 n_name 分组
按收入降序排序；

--Q6
选择
  sum(l_extendedprice * l_discount) as revenue
来自
  lineitem
哪里
  l_shipdate >= date '1994-01-01'
  和 l_shipdate < date '1994-01-01' + interval '1' year
  和 l_discount 在 .06 - 0.01 和 .06 + 0.01 之间
  和 l_quantity < 24;

--Q7
选择 supp_nation，cust_nation，l_year，sum(volume) as revenue
来自 (
    选择
      n1.n_name as supp_nation，
      n2.n_name as cust_nation，
      extract(year from l_shipdate) as l_year，
      l_extendedprice * (1 - l_discount) as volume
    来自 supplier，lineitem，orders，customer，nation n1，nation n2
    哪里
      s_suppkey = l_suppkey
      和 o_orderkey = l_orderkey
      和 c_custkey = o_custkey
      和 s_nationkey = n1.n_nationkey
      和 c_nationkey = n2.n_nationkey
      和 (
        (n1.n_name = 'FRANCE' 和 n2.n_name = 'GERMANY')
        或 (n1.n_name = 'GERMANY' 和 n2.n_name = 'FRANCE')
      )
      和 l_shipdate 在 date '1995-01-01' 和 date '1996-12-31' 之间
  ) as shipping
按 supp_nation，cust_nation，l_year 分组
按 supp_nation，cust_nation，l_year 排序；

--Q8
选择
  o_year，sum(case when nation = 'BRAZIL' then volume else 0 end) / sum(volume) as mkt_share
来自
  (
    选择
      extract(year from o_orderdate) as o_year，
      l_extendedprice * (1 - l_discount) as volume，
      n2.n_name as nation
    来自 part，supplier，lineitem，orders，customer，nation n1，nation n2，region
    哪里
      p_partkey = l_partkey
      和 s_suppkey = l_suppkey
      和 l_orderkey = o_orderkey
      和 o_custkey = c_custkey
      和 c_nationkey = n1.n_nationkey
      和 n1.n_regionkey = r_regionkey
      和 r_name = 'AMERICA'
      和 s_nationkey = n2.n_nationkey
      和 o_orderdate 在 date '1995-01-01' 和 date '1996-12-31' 之间
      和 p_type = 'ECONOMY ANODIZED STEEL'
  ) as all_nations
按 o_year 分组
按 o_year 排序；

--Q9
选择 nation，o_year，sum(amount) as sum_profit
来自
  (
    选择
      n_name as nation，
      extract(year from o_orderdate) as o_year，
      l_extendedprice * (1 - l_discount) - ps_supplycost * l_quantity as amount
    来自 part，supplier，lineitem，partsupp，orders，nation
    哪里
      s_suppkey = l_suppkey
      和 ps_suppkey = l_suppkey
      和 ps_partkey = l_partkey
      和 p_partkey = l_partkey
      和 o_orderkey = l_orderkey
      和 s_nationkey = n_nationkey
      和 p_name like '%green%'
  ) as profit
按 nation，o_year 分组
按 nation，o_year 降序排序；

--Q10
选择 c_custkey，c_name，sum(l_extendedprice * (1 - l_discount)) as revenue，c_acctbal，n_name，c_address，c_phone，c_comment
    来自 customer，orders，lineitem，nation
    哪里
      c_custkey = o_custkey
      和 l_orderkey = o_orderkey
      和 o_orderdate >= date '1993-10-01'
      和 o_orderdate < date '1993-12-01'
      和 l_returnflag = 'R'
      和 c_nationkey = n_nationkey
    按 c_custkey，c_name，c_acctbal，c_phone，n_name，c_address，c_comment 分组
    按收入降序排序；

--Q11
选择 ps_partkey，sum(ps_supplycost * ps_availqty) as value
来自 partsupp，supplier，nation
哪里
  ps_suppkey = s_suppkey
  和 s_nationkey = n_nationkey
  和 n_name = 'GERMANY'
按 ps_partkey 分组，有
    sum(ps_supplycost * ps_availqty) > (
      选择 sum(ps_supplycost * ps_availqty) * 0.0001000000
      来自 partsupp，supplier，nation
      哪里
        ps_suppkey = s_suppkey
        和 s_nationkey = n_nationkey
        和 n_name = 'GERMANY'
    )
按价值降序排序；

--Q12
选择
  l_shipmode，
  sum(case when o_orderpriority = '1-URGENT' 或 o_orderpriority = '2-HIGH' then 1 else 0 end) as high_line_count，
  sum(case when o_orderpriority <> '1-URGENT' 和 o_orderpriority <> '2-HIGH' then 1 else 0 end) as low_line_count
来自 orders，lineitem
哪里
  o_orderkey = l_orderkey
  和 l_shipmode 在 ('MAIL', 'SHIP') 中
  和 l_commitdate < l_receiptdate
  和 l_shipdate < l_commitdate
  和 l_receiptdate >= date '1994-01-01'
  和 l_receiptdate < date '1994-01-01' + interval '1' year
按 l_shipmode 分组
按 l_shipmode 排序；

--Q13
选择 c_count，count(*) as custdist
来自 (选择 c_custkey，count(o_orderkey) as c_count
    来自 customer left outer join orders on c_custkey = o_custkey 和 o_comment not like '%special%requests%'
    按 c_custkey 分组
) as c_orders
按 c_count 分组
按 custdist 降序，c_count 降序排序；

--Q14
选择
  100.00 * sum(case when p_type like 'PROMO%' then l_extendedprice * (1 - l_discount) else 0 end) / sum(l_extendedprice * (1 - l_discount)) as promo_revenue
来自 lineitem，part
哪里
  l_partkey = p_partkey
  和 l_shipdate >= date '1995-09-01'
  和 l_shipdate < date '1995-09-01' + interval '1' month；

--Q15
选择 s_suppkey，s_name，s_address，s_phone，total_revenue
来自 supplier，revenue0
哪里
  s_suppkey = supplier_no
  和 total_revenue = (
    选择 max(total_revenue) 来自 revenue0
  )
按 s_suppkey 排序；

--Q16
选择 p_brand，p_type，p_size，count(distinct ps_suppkey) as supplier_cnt
来自 partsupp，part
哪里
  p_partkey = ps_partkey
  和 p_brand <> 'Brand#45'
  和 p_type not like 'MEDIUM POLISHED%'
  和 p_size 在 (49, 14, 23, 45, 19, 3, 36, 9) 中
  和 ps_suppkey not in (
    选择 s_suppkey 来自 supplier 哪里 s_comment like '%Customer%Complaints%'
  )
按 p_brand，p_type，p_size 分组
按 supplier_cnt 降序，p_brand，p_type，p_size 排序；

--Q17
选择 sum(l_extendedprice) / 7.0 as avg_yearly
来自 lineitem，part
哪里
  p_partkey = l_partkey
  和 p_brand = 'Brand#23'
  和 p_container = 'MED BOX'
  和 l_quantity < (
    选择 0.2 * avg(l_quantity) 来自 lineitem 哪里 l_partkey = p_partkey
  )；

--Q18
选择 c_name，c_custkey，o_orderkey，o_orderdate，o_totalprice，sum(l_quantity)
来自 customer，orders，lineitem
哪里
  o_orderkey 在 (
    选择 l_orderkey 来自 lineitem
    按 l_orderkey 分组，有
        sum(l_quantity) > 300
  )
  和 c_custkey = o_custkey
  和 o_orderkey = l_orderkey
按 c_name，c_custkey，o_orderkey，o_orderdate，o_totalprice 分组
按 o_totalprice 降序，o_orderdate 排序；

--Q19
选择 sum(l_extendedprice* (1 - l_discount)) as revenue
来自 lineitem，part
哪里
  (
    p_partkey = l_partkey
    和 p_brand = 'Brand#12'
    和 p_container 在 ('SM CASE', 'SM BOX', 'SM PACK', 'SM PKG') 中
    和 l_quantity >= 1 和 l_quantity <= 1 + 10
    和 p_size 在 1 和 5 之间
    和 l_shipmode 在 ('AIR', 'AIR REG') 中
    和 l_shipinstruct = 'DELIVER IN PERSON'
  )
  或
  (
    p_partkey = l_partkey
    和 p_brand = 'Brand#23'
    和 p_container 在 ('MED BAG', 'MED BOX', 'MED PKG', 'MED PACK') 中
    和 l_quantity >= 10 和 l_quantity <= 10 + 10
    和 p_size 在 1 和 10 之间
    和 l_shipmode 在 ('AIR', 'AIR REG') 中
    和 l_shipinstruct = 'DELIVER IN PERSON'
  )
  或
  (
    p_partkey = l_partkey
    和 p_brand = 'Brand#34'
    和 p_container 在 ('LG CASE', 'LG BOX', 'LG PACK', 'LG PKG') 中
    和 l_quantity >= 20 和 l_quantity <= 20 + 10
    和 p_size 在 1 和 15 之间
    和 l_shipmode 在 ('AIR', 'AIR REG') 中
    和 l_shipinstruct = 'DELIVER IN PERSON'
  )；

--Q20
选择 s_name，s_address
来自 supplier，nation
哪里
  s_suppkey 在 (
    选择 ps_suppkey
    来自 partsupp
    哪里
      ps_partkey 在 (
        选择 p_partkey 来自 part
        哪里 p_name like 'forest%'
      )
      和 ps_availqty > (
        选择 0.5 * sum(l_quantity)
        来自 lineitem
        哪里
          l_partkey = ps_partkey
          和 l_suppkey = ps_suppkey
          和 l_shipdate >= date '1994-01-01'
          和 l_shipdate < date '1994-01-01' + interval '1' year
      )
  )
  和 s_nationkey = n_nationkey
  和 n_name = 'CANADA'
按 s_name 排序；

--Q21
选择 s_name，count(*) as numwait
来自 supplier，lineitem l1，orders，nation
哪里
  s_suppkey = l1.l_suppkey
  和 o_orderkey = l1.l_orderkey
  和 o_orderstatus = 'F'
  和 l1.l_receiptdate > l1.l_commitdate
  和存在 (
    选择 * 来自 lineitem l2
    哪里
      l2.l_orderkey = l1.l_orderkey
      和 l2.l_suppkey <> l1.l_suppkey
  )
  和不存在 (
    选择 * 来自 lineitem l3
    哪里
      l3.l_orderkey = l1.l_orderkey
      和 l3.l_suppkey <> l1.l_suppkey
      和 l3.l_receiptdate > l3.l_commitdate
  )
  和 s_nationkey = n_nationkey
  和 n_name = 'SAUDI ARABIA'
按 s_name 分组
按 numwait 降序，s_name 排序；

--Q22
选择
  cntrycode，
  count(*) as numcust，
  sum(c_acctbal) as totacctbal
来自
  (
    选择
      substr(c_phone, 1, 2) as cntrycode，
      c_acctbal
    来自
      customer
    哪里
      substr(c_phone, 1, 2) 在
        ('13', '31', '23', '29', '30', '18', '17') 中
      和 c_acctbal > (
        选择
          avg(c_acctbal)
        来自
          customer
        哪里
          c_acctbal > 0.00
          和 substr(c_phone, 1, 2) 在
            ('13', '31', '23', '29', '30', '18', '17') 中
      )
      和不存在 (
        选择
          *
        来自
          orders
        哪里
          o_custkey = c_custkey
      )
  ) as custsale
按 cntrycode 分组
按 cntrycode 排序；
```SQL
select
  cntrycode,
  count(*) as numcust,
  sum(c_acctbal) as totacctbal
from
  (
    select
      substr(c_phone, 1, 2) as cntrycode,
      c_acctbal
    from
      customer
    where
      substr(c_phone, 1, 2) in
        ('13', '31', '23', '29', '30', '18', '17')
      and c_acctbal > (
        select
          avg(c_acctbal)
        from
          customer
        where
          c_acctbal > 0.00
          and substr(c_phone, 1, 2) in
            ('13', '31', '23', '29', '30', '18', '17')
      )
      and not exists (
        select
          *
        from
          orders
        where
          o_custkey = c_custkey
      )
  ) as custsale
group by
  cntrycode
order by
  cntrycode;