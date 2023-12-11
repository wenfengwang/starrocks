---
displayed_sidebar: "Japanese"
---

# TPC-Hベンチマーク

TPC-HはTransaction Processing Performance Council (TPC) によって開発された決定支援ベンチマークです。ビジネス志向のアドホッククエリと同時データ変更のスイートで構成されています。
TPC-Hは、販売システムのデータウェアハウスをシミュレートするために、実際の製品環境に基づいたモデルを構築するために使用できます。このテストでは、1 GBから3 TBのデータサイズを持つ8つのテーブルが使用されます。合計22のクエリがテストされ、主要なパフォーマンスメトリクスは各クエリのレスポンス時間で、これはクエリが送信されてから結果が返されるまでの期間です。

## 1. テスト結論

TPC-H 100 GBのデータセットに対して22のクエリをテストします。以下の図はテスト結果を示しています。
![comparison](../assets/7.2-1.png)

テストでは、StarRocksはネイティブストレージとHive外部テーブルの両方からデータをクエリします。StarRocksとTrinoは、Hive外部テーブルから同じデータコピーをクエリします。データはZLIB圧縮され、ORC形式で保存されています。

StarRocksがネイティブストレージからデータをクエリするレイテンシは21秒であり、StarRocksがHive外部テーブルからデータをクエリするレイテンシは92秒であり、TrinoがHive外部テーブルからデータをクエリするレイテンシは**最大307秒**です。

## 2. テストの準備

### 2.1 ハードウェア環境

| マシン         | 3つのクラウドホスト                            |
| -------------- | ---------------------------------------------- |
| CPU            | 16コア                                         |
| メモリ         | 64 GB                                          |
| ネットワーク帯域 | 5 Gbit/s                                       |
| ディスク        | システムディスク：ESSD 40 GB データディスク：ESSD 100 GB |

### 2.2 ソフトウェア環境

- カーネルバージョン：Linux 3.10.0-1127.13.1.el7.x86_64
- OSバージョン：CentOS Linux リリース 7.8.2003
- ソフトウェアバージョン：StarRocks 2.1、Trino-357、Hive-3.1.2

> StarRocks FEは、別々に展開されるか、BEとハイブリッド展開されているかに関わらず、テスト結果に影響を与えません。

## 3. テストデータと結果

### 3.1 テストデータ

テストにはTPC-H 100 GBのデータセットが使用されます。

| テーブル   | レコード数     | StarRocksでの圧縮後のデータサイズ |
| --------- | ----------- | --------------------------------- |
| customer  | 1500万      | 2.4 GB                            |
| lineitem  | 6億         | 75 GB                             |
| nation    | 25          | 2.2 KB                            |
| orders    | 1.5億      | 16 GB                             |
| part      | 2億         | 2.4 GB                            |
| partsupp  | 8億         | 11.6 GB                           |
| region    | 5           | 389 B                             |
| supplier  | 100万      | 0.14 GB                           |

### 3.2 テストSQL

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
where  l_shipdate <= date '1998-12-01'
group by  l_returnflag,  l_linestatus
order by  l_returnflag,  l_linestatus;

--Q2
select  s_acctbal, s_name,  n_name,  p_partkey,  p_mfgr,  s_address,  s_phone,  s_comment
from  part,  partsupp,  supplier,  nation,  region
where
  p_partkey = ps_partkey
  and s_suppkey = ps_suppkey
  and p_size = 15
  and p_type like '%BRASS'
  and s_nationkey = n_nationkey
  and n_regionkey = r_regionkey
  and r_name = 'EUROPE'
  and ps_supplycost = (
    select  min(ps_supplycost)
    from  partsupp,  supplier,  nation,  region
    where
      p_partkey = ps_partkey
      and s_suppkey = ps_suppkey
      and s_nationkey = n_nationkey
      and n_regionkey = r_regionkey
      and r_name = 'EUROPE'
  )
order by  s_acctbal desc, n_name,  s_name,  p_partkey;

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
select  o_orderpriority,  count(*) as order_count
from  orders
where
  o_orderdate >= date '1993-07-01'
  and o_orderdate < date '1993-07-01' + interval '3' month
  and exists (
    select  * from  lineitem
    where  l_orderkey = o_orderkey and l_commitdate < l_receiptdate
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
```
```plaintext
選択 nation、o_year、sum(amount) as sum_profit
から
  (
    選択
      n_name as nation,
      o_orderdate から抽出(year) as o_year,
      l_extendedprice * (1 - l_discount) - ps_supplycost * l_quantity as amount
    から part, supplier, lineitem, partsupp, orders, nation
    ところで
      s_suppkey = l_suppkey
      かつ ps_suppkey = l_suppkey
      かつ ps_partkey = l_partkey
      かつ p_partkey = l_partkey
      かつ o_orderkey = l_orderkey
      かつ s_nationkey = n_nationkey
      かつ p_name が '%green%' である。
  ) sum_profit
nation、o_year でグループ化
nation、o_year desc で並べ替え。

--Q10
選択 c_custkey、c_name、sum(l_extendedprice * (1 - l_discount)) as revenue、c_acctbal、n_name、c_address、c_phone、c_comment
  から customer、orders、lineitem、nation
  ところで
    c_custkey = o_custkey
    かつ l_orderkey = o_orderkey
    かつ o_orderdate >= date '1993-10-01'
    かつ o_orderdate < date '1993-12-01'
    かつ l_returnflag = 'R'
    かつ c_nationkey = n_nationkey
  c_custkey、c_name、c_acctbal、c_phone、n_name、c_address、c_comment でグループ化
  revenue desc で並べ替え。

--Q11
選択 ps_partkey、sum(ps_supplycost * ps_availqty) as value
から partsupp、supplier、nation
ところで
  ps_suppkey = s_suppkey
  かつ s_nationkey = n_nationkey
  かつ n_name = 'GERMANY'
ps_partkey でグループ化
    かつ sum(ps_supplycost * ps_availqty) > (
      選択 sum(ps_supplycost * ps_availqty) * 0.0001000000
      から partsupp、supplier、nation
      ところで
        ps_suppkey = s_suppkey
        かつ s_nationkey = n_nationkey
        かつ n_name = 'GERMANY'
    )
value desc で並べ替え。

--Q12
選択
  l_shipmode、
  case when o_orderpriority = '1-URGENT' または o_orderpriority = '2-HIGH' ならば 1 それ以外ならば 0 end の合計 as high_line_count、
  case when o_orderpriority <> '1-URGENT' かつ o_orderpriority <> '2-HIGH' ならば 1 それ以外ならば 0 end の合計 as low_line_count
から orders、lineitem
ところで
  o_orderkey = l_orderkey
  かつ l_shipmode が ('MAIL', 'SHIP') のいずれかに含まれ
  かつ l_commitdate < l_receiptdate
  かつ l_shipdate < l_commitdate
  かつ l_receiptdate >= date '1994-01-01'
  かつ l_receiptdate < date '1994-01-01' + interval '1' year
l_shipmode でグループ化
l_shipmode で並べ替え。

--Q13
選択  c_count、  count(*) as custdist
から (選択 c_custkey、count(o_orderkey) as c_count
      から customer を左外部結合 orders に c_custkey = o_custkey かつ o_comment が '%special%requests%' でない
      顧客者ごとにグループ化
) as c_orders
c_count でグループ化
custdist desc、c_count desc で並べ替え。

--Q14
選択 100.00 * sum(case when p_type が 'PROMO%' ならば l_extendedprice * (1 - l_discount) それ以外ならば 0 end) / sum(l_extendedprice * (1 - l_discount)) as promo_revenue
から lineitem、part
ところで
  l_partkey = p_partkey
  かつ l_shipdate >= date '1995-09-01'
  かつ l_shipdate < date '1995-09-01' + interval '1' month

--Q15
選択 s_suppkey、s_name、s_address、s_phone、total_revenue
から supplier、revenue0
ところで
  s_suppkey = supplier_no
  かつ total_revenue = (
    選択 total_revenue の最大値 from revenue0
  )
s_suppkey で並べ替え。

--Q16
選択 p_brand、p_type、p_size、distinct ps_suppkey のカウント as supplier_cnt
から partsupp、part
ところで
  p_partkey = ps_partkey
  かつ p_brand が 'Brand#45' でない
  かつ p_type が 'MEDIUM POLISHED%' でない
  かつ p_size が (49, 14, 23, 45, 19, 3, 36, 9) のいずれかに含まれ
  かつ ps_suppkey が (
    選択 s_suppkey から supplier で s_comment が '%Customer%Complaints%' でない
  ) でない
p_brand、p_type、p_size でグループ化
supplier_cnt desc、p_brand、p_type、p_size で並べ替え。

--Q17
選択 sum(l_extendedprice) / 7.0 as avg_yearly
から lineitem、part
ところで
  p_partkey = l_partkey
  かつ p_brand = 'Brand#23'
  かつ p_container = 'MED BOX'
  かつ l_quantity < (
    選択 l_quantity の平均値 * 0.2 from lineitem で l_partkey = p_partkey
  )

--Q18
選択 c_name、c_custkey、o_orderkey、o_orderdate、o_totalprice、sum(l_quantity)
から customer、orders、lineitem
ところで
  o_orderkey が (
    選択 l_orderkey から lineitem で l_orderkey でグループ化し、合計 l_quantity が 300 を超える
  )
  かつ c_custkey = o_custkey
  かつ o_orderkey = l_orderkey
顧客者ごとにグループ化
o_totalprice desc、o_orderdate で並べ替え。

--Q19
選択 sum(l_extendedprice* (1 - l_discount)) as revenue
から lineitem、part
ところで
  (
    p_partkey = l_partkey
    かつ p_brand = 'Brand#12'
    かつ p_container が ('SM CASE', 'SM BOX', 'SM PACK', 'SM PKG') のいずれかに含まれ
    かつ l_quantity が 1 以上 11 以下
    かつ p_size が 1 以上 5 以下
    かつ l_shipmode が ('AIR', 'AIR REG') のいずれかに含まれ
    かつ l_shipinstruct が 'DELIVER IN PERSON'
  )
  または
  (
    p_partkey = l_partkey
    かつ p_brand = 'Brand#23'
    かつ p_container が ('MED BAG', 'MED BOX', 'MED PKG', 'MED PACK') のいずれかに含まれ
    かつ l_quantity が 10 以上 20 以下
    かつ p_size が 1 以上 10 以下
    かつ l_shipmode が ('AIR', 'AIR REG') のいずれかに含まれ
    かつ l_shipinstruct が 'DELIVER IN PERSON'
  )
  または
  (
    p_partkey = l_partkey
    かつ p_brand = 'Brand#34'
    かつ p_container が ('LG CASE', 'LG BOX', 'LG PACK', 'LG PKG') のいずれかに含まれ
    かつ l_quantity が 20 以上 30 以下
    かつ p_size が 1 以上 15 以下
    かつ l_shipmode が ('AIR', 'AIR REG') のいずれかに含まれ
    かつ l_shipinstruct が 'DELIVER IN PERSON'
  )

--Q20
選択 s_name、s_address
から supplier、nation
ところで
  s_suppkey が (
    選択 ps_suppkey から partsupp
    ところで
      ps_partkey が (
        選択 p_partkey から part
        ところで
          p_name が 'forest%' である
      )
      かつ ps_availqty > (
        選択 l_quantity の合計値の半分 from lineitem
        ところで
          l_partkey = ps_partkey
          かつ l_suppkey = ps_suppkey
          かつ l_shipdate >= date '1994-01-01'
          かつ l_shipdate < date '1994-01-01' + interval '1' year
      )
  )
  かつ s_nationkey = n_nationkey
  かつ n_name = 'CANADA'
s_name で並べ替え。

--Q21
選択 s_name、count(*) as numwait
から  supplier、lineitem l1、orders、nation
ところで
  s_suppkey = l1.l_suppkey
  かつ o_orderkey = l1.l_orderkey
  かつ o_orderstatus = 'F'
  かつ l1.l_receiptdate > l1.l_commitdate
  かつ 他には、
    選択 lineitem l2
    ところで
      l2.l_orderkey = l1.l_orderkey
      かつ l2.l_suppkey が l1.l_suppkey でない
  かつ なく、
    選択 lineitem l3
    ところで
      l3.l_orderkey = l1.l_orderkey
      かつ l3.l_suppkey が l1.l_suppkey でない
      かつ l3.l_receiptdate > l3.l_commitdate
  かつ s_nationkey = n_nationkey
  かつ n_name = 'SAUDI ARABIA'
s_name でグループ化
numwait desc、s_name で並べ替え。

--Q22
選択
  cntrycode、
  count(*) as numcust
```
```sql
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

### 3.3 テスト結果

| **クエリー** | **StarRocks-native (ms)** | **StarRocks-Hive external (ms)** | **Trino (ms)** |
| --------- | ------------------------- | -------------------------------- | -------------- |
| q1        | 1291                      | 5349                             | 14350          |
| q2        | 257                       | 940                              | 4075           |
| q3        | 690                       | 2585                             | 7615           |
| q4        | 529                       | 2514                             | 9385           |
| q5        | 1142                      | 5844                             | 15350          |
| q6        | 115                       | 2936                             | 4045           |
| q7        | 1435                      | 5128                             | 8760           |
| q8        | 855                       | 5989                             | 9535           |
| q9        | 3028                      | 5507                             | 27310          |
| q10       | 1381                      | 3654                             | 6560           |
| q11       | 265                       | 691                              | 2090           |
| q12       | 387                       | 3827                             | 7635           |
| q13       | 2165                      | 3733                             | 14530          |
| q14       | 184                       | 3824                             | 4355           |
| q15       | 354                       | 7321                             | 8975           |
| q16       | 240                       | 1029                             | 2265           |
| q17       | 753                       | 5011                             | 46405          |
| q18       | 2909                      | 7032                             | 30975          |
| q19       | 702                       | 3905                             | 7315           |
| q20       | 963                       | 4623                             | 12500          |
| q21       | 870                       | 10016                            | 58340          |
| q22       | 545                       | 991                              | 5605           |
| sum       | 21060                     | 92449                            | 307975         |

## 4. テスト手順

### 4.1 StarRocks Native テーブルのクエリー

#### 4.1.1 データ生成

1. tpch-pocツールキットをダウンロードし、コンパイルします。

    ```Bash
    wget https://starrocks-public.oss-cn-zhangjiakou.aliyuncs.com/tpch-poc-0.1.2.zip
    unzip tpch-poc-0.1.2.zip
    cd tpch-poc-0.1.2
    cd benchmark 
    ```

    関連ツールが`bin`ディレクトリに保存され、ホームディレクトリで操作を開始できます。

2. データを生成します。

    ```Bash
    # `data_100`ディレクトリに100GBのデータを生成します。
    ./bin/gen_data/gen-tpch.sh 100 data_100
    ```

#### 4.1.2 テーブルスキーマの作成

1. 設定ファイル `conf/starrocks.conf` を修正し、クラスターアドレスを指定します。

    ```SQL
    # StarRocks configuration
    mysql_host: 192.168.1.1
    mysql_port: 9030
    mysql_user: root
    mysql_password:
    database: tpch

    # Cluster ports
    http_port: 8030
    be_heartbeat_port: 9050
    broker_port: 8000

    # parallel_fragment_exec_instance_num specifies the parallelism.
    # The recommended parallelism is half the CPU cores of a cluster. The following example uses 8.
    parallel_num: 8
    ```

2. 次のスクリプトを実行してテーブルを作成します:

    ```SQL
    # 100GBのデータ用のテーブルを作成します。
    ./bin/create_db_table.sh ddl_100
    ```

    テーブルは作成され、デフォルトのバケット数が指定されています。以前のテーブルを削除し、クラスターサイズとノード構成に基づいてバケット数を再計画し、より良いテスト結果を得るのに役立ちます。

    ```SQL
    # customerという名前のテーブルを作成します。
    if exists customer;
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
    # lineitemという名前のテーブルを作成します。
    if exists lineitem;
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
    # nationという名前のテーブルを作成します。
    if exists nation;
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
    # oerdersという名前のテーブルを作成します。
    if exists orders;
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
    # partという名前のテーブルを作成します。
    if exists part;
    CREATE TABLE part (
        p_partkey          int NOT NULL,
        p_name        VARCHAR(55) NOT NULL,
        p_mfgr        VARCHAR(25) NOT NULL,
        p_brand       VARCHAR(10) NOT NULL,
        p_type        VARCHAR(25) NOT NULL,
        p_size        int NOT NULL,
        p_container   VARCHAR(10) NOT NULL,
    ```
```sql
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
    #テーブルpartsuppを作成します。
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
    #テーブルregionを作成します。
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
    #テーブルsupplierを作成します。
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

#### 4.1.3 データの読み込み

```Python
./bin/stream_load.sh data_100
```

#### 4.1.4 統計情報の収集（オプション）

このステップはオプションです。より良いテスト結果を生成するために、完全な統計情報の収集を推奨します。StarRocksクラスタに接続した後に次のコマンドを実行してください：

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

#### 4.1.5 データのクエリ

```Python
./bin/benchmark.sh -p -d tpch
```

### 4.2 Hive外部テーブルのクエリ

#### 4.2.1 テーブルスキーマの作成

Hiveでテーブルスキーマを作成します。ストレージフォーマットはORCです。

```SQL
create database tpch_hive_orc;
use tpch_hive_orc;
--テーブルcustomerを作成します。
CREATE TABLE `customer`(
  `c_custkey` int, 
  `c_name` varchar(25), 
  `c_address` varchar(40), 
  `c_nationkey` int, 
  `c_phone` varchar(15), 
  `c_acctbal` decimal(15, 2), 
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
--テーブルlineitemを作成します。
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
--テーブルnationを作成します。
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
--テーブルordersを作成します。
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
--テーブルpartを作成します。
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
```
--StarRocksで外部テーブルを作成します。

```SQL
create database if not exists tpch_sr;
use tpch_sr;
CREATE EXTERNAL RESOURCE "hive_test" PROPERTIES (
  "type" = "hive",
  "hive.metastore.uris" = "thrift://xxx.xxx.xxx.xxx:9083"
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
    o_totalprice     DECIMAL(15,2),
    o_orderdate      DATE,
    o_orderpriority  VARCHAR(15),
    o_clerk          VARCHAR(15),
    o_shippriority   INTEGER,
    o_comment        VARCHAR(79))
ENGINE=hive
properties (
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
    p_comment     VARCHAR(23))
ENGINE=hive
properties (
    "resource" = "hive_test",
    "table" = "part",
    "database" = "tpch_hive_orc"
);
CREATE EXTERNAL TABLE IF NOT EXISTS partsupp ( 
    ps_partkey     INTEGER,
    ps_suppkey     INTEGER,
    ps_availqty    INTEGER,
    ps_supplycost  DECIMAL(15,2),
    ps_comment     VARCHAR(199))
ENGINE=hive
properties (
    "resource" = "hive_test",
    "table" = "partsupp",
    "database" = "tpch_hive_orc"
);
CREATE EXTERNAL TABLE IF NOT EXISTS region  ( 
    r_regionkey  INTEGER,
    r_name       VARCHAR(25),
    r_comment    VARCHAR(152))
ENGINE=hive
properties (
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
    s_comment     VARCHAR(101))
ENGINE=hive
properties (
    "resource" = "hive_test",
    "table" = "supplier",
    "database" = "tpch_hive_orc"
);
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
```SQL
      `l_commitdate` 日付, 
      `l_receiptdate` 日付, 
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
      --Create an external table named nation.
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
      --Create an external table named orders.
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
      --Create an external table named part.
      CREATE EXTERNAL TABLE `part`(
      `p_partkey` int, 
      `p_name` varchar(55), 
      `p_mfgr` varchar(25), 
      `p_brand` varchar(10), 
      `p_type` varchar(25), 
      `p_size` int, 
      `p_container` varchar(10), 
      `p_retailprice` double, 
      `p_comment` varchar(23))
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
      --Create an external table named partsupp.
      CREATE EXTERNAL TABLE `partsupp`(
      `ps_partkey` int, 
      `ps_suppkey` int, 
      `ps_availqty` int, 
      `ps_supplycost` double, 
      `ps_comment` varchar(199))
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
      --Create an external table named region.
      CREATE EXTERNAL TABLE `region`(
      `r_regionkey` int, 
      `r_name` varchar(25), 
      `r_comment` varchar(152))
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
      --Create an external table named supplier.
      CREATE EXTERNAL TABLE `supplier`(
      `s_suppkey` int, 
      `s_name` varchar(25), 
      `s_address` varchar(40), 
      `s_nationkey` int, 
      `s_phone` varchar(15), 
      `s_acctbal` double, 
      `s_comment` varchar(101))
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

2. それぞれのHive外部テーブルのCSVデータをORC形式のHiveテーブルに転送します。これにより、後続のStarRocksおよびTrinoでORC形式のHive外部テーブルを作成します。

    ```SQL
    use tpch_hive_csv;
    insert into tpch_hive_orc.customer  select * from customer;
    insert into tpch_hive_orc.lineitem  select * from lineitem;
    insert into tpch_hive_orc.nation  select * from nation;
    insert into tpch_hive_orc.orders  select * from orders;
    insert into tpch_hive_orc.part  select * from part;
    insert into tpch_hive_orc.partsupp  select * from partsupp;
    insert into tpch_hive_orc.region  select * from region;
    insert into tpch_hive_orc.supplier  select * from supplier;
    ```

#### 4.2.3 クエリデータ

```SQL
use tpch_sr;
--現在のセッションのシステム変数をクエリします。
show variables;
--並列処理を8に設定します。
set parallel_fragment_exec_instance_num = 8;

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
where  l_shipdate <= date '1998-12-01'
group by  l_returnflag,  l_linestatus
order by  l_returnflag,  l_linestatus;

--Q2
select  s_acctbal, s_name,  n_name,  p_partkey,  p_mfgr,  s_address,  s_phone,  s_comment
from  part,  partsupp,  supplier,  nation,  region
where
  p_partkey = ps_partkey
  and s_suppkey = ps_suppkey
  and p_size = 15
  and p_type like '%BRASS'
  and s_nationkey = n_nationkey
  and n_regionkey = r_regionkey
  and r_name = 'EUROPE'
  and ps_supplycost = (
    select  min(ps_supplycost)
    from  partsupp,  supplier,  nation,  region
    where
      p_partkey = ps_partkey
      and s_suppkey = ps_suppkey
      and s_nationkey = n_nationkey
      and n_regionkey = r_regionkey
      and r_name = 'EUROPE'
  )
order by  s_acctbal desc, n_name,  s_name,  p_partkey;

--Q3
select
  l_orderkey, sum(l_extendedprice * (1 - l_discount)) as revenue, o_orderdate, o_shippriority
from
  customer, orders, lineitem
where
  c_mktsegment = 'BUILDING'
  and c_custkey = o_custkey
```
```sql
  and l_orderkey = o_orderkey
  and o_orderdate < date '1995-03-15'
  and l_shipdate > date '1995-03-15'
group by l_orderkey, o_orderdate, o_shippriority
order by revenue desc, o_orderdate;

--Q4
select  o_orderpriority,  count(*) as order_count
from  orders
where
  o_orderdate >= date '1993-07-01'
  and o_orderdate < date '1993-07-01' + interval '3' month
  and exists (
    select  * from  lineitem
    where  l_orderkey = o_orderkey and l_commitdate < l_receiptdate
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
select  c_count,  count(*) as custdist
from (select c_custkey,count(o_orderkey) as c_count
    from customer left outer join orders on c_custkey = o_custkey and o_comment not like '%special%requests%'
    group by  c_custkey
) as c_orders
group by  c_count
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
```
```plaintext
  およびp_brand = 'Brand#23'
  およびp_container = 'MED BOX'
  およびl_quantity < (
    select 0.2 * avg(l_quantity) from lineitem where l_partkey = p_partkey
  );

--Q18
customer、orders、lineitemから、
  customer.c_custkey = orders.o_custkey
  かつ orders.o_orderkey = lineitem.l_orderkey
  かつ orders.o_orderkeyは、lineitem.l_orderkeyからの
    l_quantityの合計が300より大きいグループごとに
      l_orderkey in (
        select l_orderkey from lineitem
        group by l_orderkey having
            sum(l_quantity) > 300
      )
  となる
c_name、c_custkey、o_orderkey、o_orderdate、o_totalprice、sum(l_quantity)を
c_name、c_custkey、o_orderkey、o_orderdate、o_totalpriceの降順、o_orderdateの昇順で取得
order by o_totalprice desc, o_orderdate;

--Q19
lineitem、partから
  (
    p_partkey = l_partkey
    かつ p_brand = 'Brand#12'
    かつ p_container in ('SM CASE', 'SM BOX', 'SM PACK', 'SM PKG')
    かつ l_quantity >= 1 かつ l_quantity <= 1 + 10
    かつ p_sizeが1～5の間
    かつ l_shipmode in ('AIR', 'AIR REG')
    かつ l_shipinstruct = 'DELIVER IN PERSON'
  )
  または
  (
    p_partkey = l_partkey
    かつ p_brand = 'Brand#23'
    かつ p_container in ('MED BAG', 'MED BOX', 'MED PKG', 'MED PACK')
    かつ l_quantity >= 10 かつ l_quantity <= 10 + 10
    かつ p_sizeが1～10の間
    かつ l_shipmode in ('AIR', 'AIR REG')
    かつ l_shipinstruct = 'DELIVER IN PERSON'
  )
  または
  (
    p_partkey = l_partkey
    かつ p_brand = 'Brand#34'
    かつ p_container in ('LG CASE', 'LG BOX', 'LG PACK', 'LG PKG')
    かつ l_quantity >= 20 かつ l_quantity <= 20 + 10
    かつ p_sizeが1～15の間
    かつ l_shipmode in ('AIR', 'AIR REG')
    かつ l_shipinstruct = 'DELIVER IN PERSON'
  )
  の条件で、
  l_extendedprice* (1 - l_discount)の合計を取得

--Q20
supplier、nationから
  supplier.s_nationkey = nation.n_nationkey
  かつ nation.n_name = 'CANADA'
  かつ
  (
    supplier.s_suppkey in (
      partsupp.ps_suppkey
      from partsupp
      where
        partsupp.ps_partkey in (
          part.p_partkey from part
          where p_nameが'forest%'である
        )
        かつ partsupp.ps_availqty > (
          select 0.5 * lineitem.l_quantity
          from lineitem
          where
            l_partkey = partsupp.ps_partkey
            かつ l_suppkey = partsupp.ps_suppkey
            かつ l_shipdate >= date '1994-01-01'
            かつ l_shipdate < date '1994-01-01' + interval '1' year
        )
    )
  )
  の条件で、
  s_name、s_addressを取得
  order by s_name;

--Q21
supplier(s_suppkey = lineitem.l_suppkey)、lineitem(l_receiptdate > l_commitdate)、orders(o_orderstatus = 'F')、nation(n_name = 'SAUDI ARABIA')から、
  存在するl2(l_suppkey <> l1.l_suppkey)と、存在しないl3(l_suppkey <> l1.l_suppkeyかつl_receiptdate > l_commitdate)を持つ
  s_nameをグループ化し、numwaitを降順、s_nameの昇順で取得
  group by s_name
  order by  numwait desc, s_name;

--Q22
customerから
  substr(c_phone, 1, 2)が
    ('13', '31', '23', '29', '30', '18', '17')のいずれかである
  かつ c_acctbalが、同じcntrycodeにおけるc_acctbalの平均を上回る
  かつ o_custkey = c_custkeyを満たさない
  条件で、
  substr(c_phone, 1, 2)、numcust、totacctbalを取得
  group by
    cntrycode
  order by
    cntrycode;
```