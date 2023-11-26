---
displayed_sidebar: "Japanese"
---

# SSBフラットテーブルベンチマーク

スタースキーマベンチマーク（SSB）は、OLAPデータベース製品の基本的なパフォーマンスメトリックをテストするために設計されています。SSBは、学術界や産業界で広く適用されているスタースキーマテストセットを使用しています。詳細については、[スタースキーマベンチマークの論文](https://www.cs.umb.edu/~poneil/StarSchemaB.PDF)を参照してください。
ClickHouseは、スタースキーマをワイドフラットテーブルに変換し、SSBを単一テーブルベンチマークに書き換えます。詳細については、[ClickHouseのスタースキーマベンチマーク](https://clickhouse.tech/docs/en/getting-started/example-datasets/star-schema/)を参照してください。
このテストでは、StarRocks、Apache Druid、およびClickHouseのパフォーマンスをSSB単一テーブルデータセットと比較します。

## テスト結論

- SSB標準データセットで実行された13のクエリのうち、StarRocksのクエリパフォーマンスは**ClickHouseの2.1倍、Apache Druidの8.7倍**です。
- StarRocksのBitmapインデックスを有効にすると、パフォーマンスは無効にした場合と比較して1.3倍になります。StarRocksの総合パフォーマンスは**ClickHouseの2.8倍、Apache Druidの11.4倍**です。

![全体の比較](../assets/7.1-1.png)

## テストの準備

### ハードウェア

| マシン           | クラウドホスト3台                                          |
| ----------------- | ------------------------------------------------------------ |
| CPU               | 16コアIntel (R) Xeon (R) Platinum 8269CY CPU @2.50GHz <br />キャッシュサイズ：36608 KB |
| メモリ            | 64 GB                                                        |
| ネットワーク帯域幅 | 5 Gbit/s                                                     |
| ディスク          | ESSD                                                         |

### ソフトウェア

StarRocks、Apache Druid、およびClickHouseは、同じ構成のホストに展開されています。

- StarRocks：1つのFEと3つのBE。FEは、BEとは別々にまたはハイブリッドに展開できます。
- ClickHouse：分散テーブルを持つ3つのノード
- Apache Druid：3つのノード。1つはマスターサーバーとデータサーバーで展開され、1つはクエリサーバーとデータサーバーで展開され、3つ目はデータサーバーのみで展開されます。

カーネルバージョン：Linux 3.10.0-1160.59.1.el7.x86_64

OSバージョン：CentOS Linux release 7.9.2009

ソフトウェアバージョン：StarRocks Community Version 3.0、ClickHouse 23.3、Apache Druid 25.0.0

## テストデータと結果

### テストデータ

| テーブル          | レコード数       | 説明              |
| -------------- | ------------ | ------------------------ |
| lineorder      | 6億件  | 注文明細テーブル     |
| customer       | 300万件    | 顧客次元テーブル |
| part           | 140万件  | 部品次元テーブル     |
| supplier       | 20万件 | 仕入先次元テーブル |
| dates          | 2,556件        | 日付次元テーブル     |
| lineorder_flat | 6億件  | フラットな注文明細テーブル     |

### テスト結果

以下の表は、13のクエリに対するパフォーマンステスト結果を示しています。クエリのレイテンシの単位はミリ秒です。表のヘッダーの`ClickHouse vs StarRocks`は、ClickHouseのクエリ応答時間をStarRocksのクエリ応答時間で割ったものです。値が大きいほど、StarRocksのパフォーマンスが良いことを示します。

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
| sum  | 1236          | 939                 | 2645                | 2.14                    | 10750            | 8.70               |

## テスト手順

ClickHouseテーブルの作成方法とデータのロード方法の詳細については、[ClickHouse公式ドキュメント](https://clickhouse.tech/docs/en/getting-started/example-datasets/star-schema/)を参照してください。以下のセクションでは、StarRocksのデータ生成とデータロードについて説明します。

### データの生成

ssb-pocツールキットをダウンロードしてコンパイルします。

```Bash
wget https://starrocks-public.oss-cn-zhangjiakou.aliyuncs.com/ssb-poc-1.0.zip
unzip ssb-poc-1.0.zip
cd ssb-poc-1.0/
make && make install
cd output/
```

コンパイルが完了すると、関連するツールが`output`ディレクトリにインストールされ、以下の操作はすべてこのディレクトリで実行されます。

まず、SSB標準データセット`スケールファクター=100`のデータを生成します。

```Bash
sh bin/gen-ssb.sh 100 data_dir
```

### テーブルスキーマの作成

1. 設定ファイル`conf/starrocks.conf`を編集し、クラスターアドレスを指定します。特に`mysql_host`と`mysql_port`に注意してください。

2. 次のコマンドを実行してテーブルを作成します：

    ```SQL
    sh bin/create_db_table.sh ddl_100
    ```

### データのクエリ

```Bash
sh bin/benchmark.sh ssb-flat
```

### Bitmapインデックスの有効化

Bitmapインデックスを有効にすると、StarRocksのパフォーマンスが向上します。特に、Q2.2、Q2.3、およびQ3.3のパフォーマンスをテストしたい場合は、すべてのSTRING列にBitmapインデックスを作成できます。

1. 別の`lineorder_flat`テーブルを作成し、Bitmapインデックスを作成します。

    ```SQL
    sh bin/create_db_table.sh ddl_100_bitmap_index
    ```

2. すべてのBEの`be.conf`ファイルに以下の設定を追加し、BEを再起動して設定が有効になるようにします。

    ```SQL
    bitmap_max_filter_ratio=1000
    ```

3. データロードスクリプトを実行します。

    ```SQL
    sh bin/flat_insert.sh data_dir
    ```

データのロードが完了したら、データバージョンのコンパクションが完了するのを待ってから、[4.4](#query-data)を再度実行して、Bitmapインデックスが有効になった後のデータをクエリします。

データバージョンのコンパクションの進行状況は、`select CANDIDATES_NUM from information_schema.be_compactions`を実行して表示できます。3つのBEノードの場合、以下の結果はコンパクションが完了していることを示しています：

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

## テストSQLとテーブル作成ステートメント

### テストSQL

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

### テーブル作成ステートメント

#### デフォルトの`lineorder_flat`テーブル

以下のステートメントは、現在のクラスターサイズとデータサイズ（3つのBE、スケールファクター=100）に一致します。クラスターにより多くのBEノードやより大きなデータサイズがある場合は、バケットの数を調整してテーブルを再作成し、データを再度ロードしてより良いテスト結果を得ることができます。

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

#### Bitmapインデックスを持つ`lineorder_flat`テーブル

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
