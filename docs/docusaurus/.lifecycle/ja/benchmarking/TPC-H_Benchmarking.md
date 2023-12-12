---
displayed_sidebar: "Japanese"
---

# TPC-H ベンチマーキング

TPC-HはTransaction Processing Performance Council（TPC）によって開発された意思決定支援のベンチマークです。ビジネス志向のアドホッククエリと並行データ変更のスイートから構成されています。
TPC-Hは、販売システムのデータウェアハウスをシミュレートするための実際の製品環境に基づくモデルを構築するために使用することができます。このテストでは、データサイズが1 GBから3 TBまでの8つのテーブルが使用されます。合計22個のクエリがテストされ、主要なパフォーマンス指標は各クエリの応答時間であり、これはクエリの送信から結果の返却までの期間です。

## 1. テスト結論

我々はTPC-H 100 GBのデータセットに対して22クエリのテストを実施します。次の図はテスト結果を示しています。
![comparison](../assets/7.2-1.png)

テストでは、StarRocksはネイティブストレージとHive外部テーブルからデータをクエリしました。StarRocksとTrinoは、Hive外部テーブルから同じデータコピーをクエリしました。データはZLIB圧縮され、ORC形式で保存されています。

StarRocksのネイティブストレージからデータをクエリするレイテンシは21秒であり、StarRocksがHive外部テーブルからクエリするレイテンシは92秒であり、TrinoがHive外部テーブルからクエリするレイテンシは**最長307秒**となります。

## 2. テスト準備

### 2.1 ハードウェア環境

| マシン           | 3つのクラウドホスト                          |
| ----------------- | ---------------------------------------------- |
| CPU               | 16コア                                         |
| メモリ            | 64 GB                                          |
| ネットワーク帯域幅 | 5 Gbit/s                                       |
| ディスク          | システムディスク: ESSD 40 GB データディスク: ESSD 100 GB |

### 2.2 ソフトウェア環境

- カーネルバージョン: Linux 3.10.0-1127.13.1.el7.x86_64
- OSバージョン: CentOS Linux released 7.8.2003
- ソフトウェアバージョン: StarRocks 2.1, Trino-357, Hive-3.1.2

> StarRocks FEは別々に展開されることも、BEとハイブリッドで展開されることもあり、これはテスト結果に影響しません。

## 3. テストデータと結果

### 3.1 テストデータ

テストにはTPC-H 100 GBのデータセットが使用されます。

| テーブル   | レコード数 | StarRocksでの圧縮後のデータサイズ  |
| -------- | ----------- | ----------------------------------- |
| customer | 1500万      | 2.4 GB                              |
| lineitem | 6億         | 75 GB                               |
| nation   | 25          | 2.2 KB                              |
| orders   | 1.5億       | 16 GB                               |
| part     | 2億         | 2.4 GB                              |
| partsupp | 8億         | 11.6 GB                             |
| region   | 5           | 389 B                               |
| supplier | 100万       | 0.14 GB                             |

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
--Q9
```
国、o_year、sum(amount) as sum_profitを選択
  (
    n_name as nation、o_orderdateからの年の抽出 as o_year、
    l_extendedprice * (1 - l_discount) - ps_supplycost * l_quantity as amount
part, supplier, lineitem, partsupp, orders, nationからの選択
    s_suppkey = l_suppkey
    およびps_suppkey = l_suppkey
    and ps_partkey = l_partkey
    およびp_partkey = l_partkey
    and o_orderkey = l_orderkey
    and s_nationkey = n_nationkey
    およびp_name LIKE '%green%'
  ) as profit
nation、o_yearによるグループ化
国、o_year descによる並べ替え;
```

--Q10
```
c_custkey、c_name、l_extendedprice * (1 - l_discount)の合計 revenue、c_acctbal、n_name、c_address、c_phone、c_commentを選択
    customer, orders, lineitem, nationからの選択
      c_custkey = o_custkey
      およびl_orderkey = o_orderkey
      and o_orderdate >= date '1993-10-01'
      およびo_orderdate < date '1993-12-01'
      およびl_returnflag = 'R'
      and c_nationkey = n_nationkey
    c_custkey、c_name、c_acctbal、c_phone、n_name、c_address、c_commentによるグループ化
    revenue descによる並べ替え;
```

--Q11
```
ps_partkey、ps_supplycost * ps_availqtyの合計値 valueを選択
partsupp, supplier, nationからの選択
  ps_suppkey = s_suppkey
  およびs_nationkey = n_nationkey
  およびn_name = 'GERMANY'
  ps_partkeyによるグループ化 having
    ps_supplycost * ps_availqtyの合計値 > (
      partsupp, supplier, nationからの選択
        ps_suppkey = s_suppkey
        およびs_nationkey = n_nationkey
        およびn_name = 'GERMANY'
      からのps_supplycost * ps_availqtyの合計値 * 0.0001000000の合計値
    )
value descによる並べ替え;
```

--Q12
```
l_shipmode、case式で o_orderpriority = '1-URGENT' または o_orderpriority = '2-HIGH' のとき1を加算し、それ以外のとき0を加算した値 high_line_count、
case式で o_orderpriority <> '1-URGENT' かつ o_orderpriority <> '2-HIGH' のとき1を加算し、それ以外のとき 0を加算した値 low_line_countを選択
orders, lineitemからの選択
  o_orderkey = l_orderkey
  およびl_shipmodeが('MAIL', 'SHIP')に含まれ
  およびl_commitdate < l_receiptdate
  およびl_shipdate < l_commitdate
  およびl_receiptdate >= date '1994-01-01'
  およびl_receiptdate < date '1994-01-01' + interval '1' year
l_shipmodeによるグループ化
l_shipmodeによる並べ替え;
```

--Q13
```
c_count、count(*) as custdistを選択
(
  c_custkeyとo_custkeyが等しいものおよびo_commentが'%special%requests%'を含まないものとしてc_custkeyとo_orderkeyの数をカウントした値 c_ordersからの選択
    customerとordersの外部結合
  c_custkeyによるグループ化
) as c_orders からの選択
c_countによるグループ化
custdist desc、c_count descによる並べ替え;
```

--Q14
```
p_typeが'PROMO%'に一致するとき l_extendedprice * (1 - l_discount)の合計値 を  l_extendedprice * (1 - l_discount)の合計値の和のうちで100.00を乗じたもので割った値 promo_revenueを選択
lineitem, partからの選択
  l_partkey = p_partkey
  およびl_shipdateが date '1995-09-01'以降 および date '1995-09-01' + interval '1' monthより前に含まれ;
```

--Q15
```
s_suppkey, s_name, s_address, s_phone, total_revenueを選択
supplier, revenue0からの選択
  s_suppkey = supplier_no
  およびtotal_revenueが revenue0からの最大値であるものと等しい
s_suppkeyによるグループ化
s_suppkeyによる並べ替え;
```

--Q16
```
p_brand, p_type, p_size、distinct ps_suppkeyの数 supplier_cntをカウントした値を選択
partsupp, partからの選択
  p_partkey = ps_partkey
  およびp_brandが 'Brand#45' でないもの
  およびp_typeが 'MEDIUM POLISHED%' でないもの
  およびp_sizeが(49, 14, 23, 45, 19, 3, 36, 9)に含まれ
  およびps_suppkeyが(
    supplierからの選択
      s_suppkeyが、s_commentが'%Customer%Complaints%'を含まないもの
  )に含まれるもの
p_brand, p_type, p_sizeによるグループ化
supplier_cnt desc、p_brand、p_type、p_sizeによる並べ替え;
```

--Q17
```
lineitem, partからの選択
  p_partkey = l_partkey
  およびp_brandが 'Brand#23'で、p_containerが 'MED BOX'であるもの
  およびl_quantity が lineitemでの l_partkey ごとの平均 l_quantity（ p_partkey = l_partkey ）の 0.2 倍以下であるもの
avg_yearly を選択;
```

--Q18
```
c_name, c_custkey, o_orderkey, o_orderdate, o_totalprice, sum(l_quantity)を選択
customer, orders, lineitemからの選択
  l_orderkeyをlineitemでグループ化し、総数が300より多いものを抽出
  およびc_custkey = o_custkey
  およびo_orderkey = l_orderkey
c_name, c_custkey, o_orderkey, o_orderdate, o_totalpriceによるグループ化
o_totalprice desc、o_orderdateによる並べ替え;
```

--Q19
```
l_extendedprice* (1 - l_discount)の合計値 revenueを選択
lineitem, partからの選択
  (
    p_partkey = l_partkey
    およびp_brandが 'Brand#12'で、p_containerが('SM CASE', 'SM BOX', 'SM PACK', 'SM PKG')のいずれかであり
    およびl_quantityが1以上10以下で、p_sizeが1以上5以下であり
    およびl_shipmodeが('AIR', 'AIR REG')で、l_shipinstructが'DELIVER IN PERSON'であるもの
  )
  OR
  (
    p_partkey = l_partkey
    およびp_brandが 'Brand#23'で、p_containerが('MED BAG', 'MED BOX', 'MED PKG', 'MED PACK')のいずれかであり
    およびl_quantityが10以上20以下で、p_sizeが1以上10以下であり
    およびl_shipmodeが('AIR', 'AIR REG')で、l_shipinstructが'DELIVER IN PERSON'であるもの
  )
  OR
  (
    p_partkey = l_partkey
    およびp_brandが 'Brand#34'で、p_containerが('LG CASE', 'LG BOX', 'LG PACK', 'LG PKG')のいずれかであり
    およびl_quantityが20以上30以下で、p_sizeが1以上15以下であり
    およびl_shipmodeが('AIR', 'AIR REG')で、l_shipinstructが'DELIVER IN PERSON'であるもの
  );
```

--Q20
```
s_name, s_addressを選択
supplier, nationからの選択
  s_suppkeyが(
    partsuppからの選択
      ps_partkeyが(
        partからの選択
          p_nameが'forest%'を含むもの
      )
      およびps_availqtyが(
        lineitemからの選択
          l_partkey = ps_partkey
          およびl_suppkey = ps_suppkey
          およびl_shipdateが date '1994-01-01' 以降 および date '1994-01-01' + interval '1' yearより前に含まれ
        のl_quantityの総和の0.5倍より多いもの
      )
  )
  およびs_nationkey = n_nationkey
  およびn_name = 'CANADA'
s_nameによる並べ替え;
```

--Q21
```
s_name, count(*) as numwaitを選択
  supplier, lineitem l1, orders, nationからの選択
    s_suppkey = l1.l_suppkey
    およびo_orderkey = l1.l_orderkey
    およびo_orderstatusが'F'であるもの
    およびl1.l_receiptdate > l1.l_commitdate
    および l1.l_orderkey と異なる l_suppkey を持つ lineitem l2 が存在するもの
    および l1.l_orderkey を持つ lineitem l3 で、 l3.l_suppkey が l1.l_suppkey と異なるもの および l3.l_receiptdate > l3.l_commitdateでないもの
    およびs_nationkeyがn_nationkeyと等しいもの
    およびn_nameが'SAUDI ARABIA'であるもの
s_nameによるグループ化
numwait desc、s_nameによる並べ替え;
```

--Q22
```
cntrycode, count(*) as numcustを選択;
```sql
      + クエリの実行結果
      + StarRocks-native (ms)
      + StarRocks-Hive外部 (ms)
      + Trino (ms)
    + q1
      + 1291
      + 5349
      + 14350
    + q2
      + 257
      + 940
      + 4075
    + q3
      + 690
      + 2585
      + 7615
    + q4
      + 529
      + 2514
      + 9385
    + q5
      + 1142
      + 5844
      + 15350
    + q6
      + 115
      + 2936
      + 4045
    + q7
      + 1435
      + 5128
      + 8760
    + q8
      + 855
      + 5989
      + 9535
    + q9
      + 3028
      + 5507
      + 27310
    + q10
      + 1381
      + 3654
      + 6560
    + q11
      + 265
      + 691
      + 2090
    + q12
      + 387
      + 3827
      + 7635
    + q13
      + 2165
      + 3733
      + 14530
    + q14
      + 184
      + 3824
      + 4355
    + q15
      + 354
      + 7321
      + 8975
    + q16
      + 240
      + 1029
      + 2265
    + q17
      + 753
      + 5011
      + 46405
    + q18
      + 2909
      + 7032
      + 30975
    + q19
      + 702
      + 3905
      + 7315
    + q20
      + 963
      + 4623
      + 12500
    + q21
      + 870
      + 10016
      + 58340
    + q22
      + 545
      + 991
      + 5605
    + sum
      + 21060
      + 92449
      + 307975
  + 4. テスト手順
    + 4.1 StarRocksネイティブテーブルのクエリ
      + 4.1.1 データの生成
        + 1. tpch-pocツールキットをダウンロードしてコンパイルします。
          + ```Bash
            wget https://starrocks-public.oss-cn-zhangjiakou.aliyuncs.com/tpch-poc-0.1.2.zip
            unzip tpch-poc-0.1.2.zip
            cd tpch-poc-0.1.2
            cd benchmark 
            ```
        + 2. データを生成します。
          + ```Bash
            # `data_100`ディレクトリに100GBのデータを生成します。
            ./bin/gen_data/gen-tpch.sh 100 data_100
            ```
      + 4.1.2 テーブルスキーマの作成
        + 1. 構成ファイル`conf/starrocks.conf`を変更し、クラスターアドレスを指定します。
          + ```SQL
            # StarRocksの構成
            mysql_host: 192.168.1.1
            mysql_port: 9030
            mysql_user: root
            mysql_password:
            database: tpch

            # クラスターポート
            http_port: 8030
            be_heartbeat_port: 9050
            broker_port: 8000

            # parallel_fragment_exec_instance_numは並列性を指定します。
            # 推奨される並列性はクラスターのCPUコア数の半分です。以下の例では8を使用しています。
            parallel_num: 8
            ```
        + 2. 以下のスクリプトを実行してテーブルを作成します。
          + ```SQL
            # 100GBのデータ用のテーブルを作成します
            ./bin/create_db_table.sh ddl_100
            ```
          テーブルが作成され、デフォルトのバケット数が指定されます。前のテーブルを削除して、クラスターサイズとノード構成に基づいてバケット数を再計画し、より良いテスト結果を得るのに役立ちます。

          ```SQL
            # customerという名前のテーブルを作成します。
            もしも前のテーブルが存在したら削除します。
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
            ```
```SQL
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

    #部品という名前のテーブルを作成します。
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

    #地域という名前のテーブルを作成します。
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

    #サプライヤーという名前のテーブルを作成します。
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

    集計ビューrevenue0を作成し、supplier_noと総収益を定義します。
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

#### 4.1.3 データのロード

```Python
./bin/stream_load.sh data_100
```

#### 4.1.4 統計情報の収集（オプション）

このステップはオプションです。より良いテスト結果を得るために、完全な統計情報を収集することをお勧めします。StarRocksクラスタに接続した後、次のコマンドを実行します。

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

Hiveでテーブルスキーマを作成します。ストレージ形式はORCです。

```SQL
create database tpch_hive_orc;
use tpch_hive_orc;
--顧客という名前のテーブルを作成します。
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
--lineitemという名前のテーブルを作成します。
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
--nationという名前のテーブルを作成します。
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
--ordersという名前のテーブルを作成します。
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
--部品という名前のテーブルを作成します。
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
```SQL
-- partsuppという名前のテーブルを作成します。
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
-- regionという名前のテーブルを作成します。
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
-- supplierという名前のテーブルを作成します。
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

StarRocksで外部テーブルを作成します。

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

#### 4.2.2 データのロード

1. Hiveで外部テーブルを作成します。テーブルフォーマットはCSVです。CSVのテストデータをHDFSのデータ保存ディレクトリにアップロードします。この例では、Hiveの外部テーブルのHDFSデータ保存パスは/user/tmp/csv/です。

    ```SQL
    create database tpch_hive_csv;
    use tpch_hive_csv;

    -- customerという名前の外部テーブルを作成します。
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
    -- lineitemという名前の外部テーブルを作成します。
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
```
```SQL
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
    --Create an external table named nation.
    外部テーブル `nation` を作成する(
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
    外部テーブル `orders` を作成する(
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
    外部テーブル `part` を作成する(
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
    外部テーブル `partsupp` を作成する(
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
    外部テーブル `region` を作成する(
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
    外部テーブル `supplier` を作成する(
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

2. 各Hive外部テーブルのCSVデータをORC形式のHiveテーブルに変換します。これにより、後続のStarRocksおよびTrinoでのテストにORC形式のHive外部テーブルを作成できます。

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

#### 4.2.3 データのクエリ

```SQL
use tpch_sr;
--現在のセッションのシステム変数をクエリします。
show variables;
--並列実行を8に設定します。
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
  p_partkey = l_partkey;
```
  and p_brand = 'Brand#23'
  and p_container = 'MED BOX'
  and l_quantity < (
    select 0.2 * avg(l_quantity) from lineitem where l_partkey = p_partkey
  );

--Q18
顧客名, 顧客キー, 注文キー, 注文日, 注文合計額, sum(l_quantity)
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
select sum(l_extendedprice* (1 - l_discount)) as revenue
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
    select      *     from    lineitem l2
    where
      l2.l_orderkey = l1.l_orderkey
      and l2.l_suppkey <> l1.l_suppkey
  )
  and not exists (
    select      *       from    lineitem l3
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