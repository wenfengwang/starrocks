---
displayed_sidebar: Chinese
---

# SSB Flat Table 性能テスト

## テスト結論

Star Schema Benchmark（以下、SSBと略）は、学術界および産業界で広く使用されている星型スキーマのベンチマークセットです（出典：[論文](https://www.cs.umb.edu/~poneil/StarSchemaB.PDF)）。このベンチマークセットを使用することで、さまざまなOLAP製品の基本的な性能指標を容易に比較することができます。ClickHouseはSSBを改変し、星型スキーマをフラット化してワイドテーブル（flat table）に変換し、単一テーブルのベンチマーク（参照：[リンク](https://clickhouse.tech/docs/en/getting-started/example-datasets/star-schema/)）に改造しました。本レポートでは、StarRocks、ClickHouse、Apache DruidのSSB単一テーブルデータセットにおける性能比較結果を記録しており、テスト結論は以下の通りです：

- 標準テストデータセットの13のクエリにおいて、StarRocksの全体的なクエリ性能はClickHouseの2.1倍、Apache Druidの8.7倍でした。
- StarRocksがBitmap Indexを有効にした後の全体的なクエリ性能は、無効時の1.3倍であり、この時の全体的なクエリ性能はClickHouseの2.8倍、Apache Druidの11.4倍でした。

![img](../assets/7.1-1.png)

本文では、SSB単一テーブルシナリオでStarRocks、ClickHouse、Apache Druidのクエリ性能を比較しました。3台の16コア64GBメモリのクラウドサーバーを使用し、6億行のデータ規模でテストを行いました。

## テスト準備

### ハードウェア環境

| マシン     | 3台のアリババクラウドサーバー                                      |
| -------- | ------------------------------------------------------------ |
| CPU      | 16コア Intel(R) Xeon(R) Platinum 8269CY CPU @ 2.50GHz <br />キャッシュサイズ：36608 KB |
| メモリ     | 64GB                                                         |
| ネットワーク帯域 | 5 Gbits/s                                                    |
| ディスク     | ESSD クラウドディスク                                          |

### ソフトウェア環境

StarRocks、ClickHouse、Apache Druidは同じ構成のマシン上でそれぞれテストを行いました。

- StarRocksは1つのFEと3つのBEをデプロイします。FEは単独でデプロイすることも、BEと組み合わせてデプロイすることもできます。
- ClickHouseは3ノードをデプロイし、分散テーブルを作成します。
- Apache Druidは3ノードすべてにData Serversをデプロイし、1ノードにMaster Serversを、別のノードにQuery Serversを組み合わせてデプロイします。

カーネルバージョン：Linux 3.10.0-1160.59.1.el7.x86_64

OSバージョン：CentOS Linux release 7.9.2009

ソフトウェアバージョン：StarRocksコミュニティ版 3.0、ClickHouse 23.3、Apache Druid 25.0.0

## テストデータと結果

### テストデータ

| テーブル名       | 行数   | 説明             |
| -------------- | ------ | ---------------- |
| lineorder      | 6億    | SSB商品注文テーブル |
| customer       | 300万  | SSB顧客テーブル   |
| part           | 140万  | SSB部品テーブル   |
| supplier       | 20万   | SSB供給者テーブル |
| dates          | 2556   | 日付テーブル       |
| lineorder_flat | 6億    | SSBフラット化されたワイドテーブル |

### テスト結果

> クエリ時間の単位はmsです。StarRocksとClickHouse、Druidのクエリ性能を比較し、ClickHouse、Druidのクエリ時間をStarRocksのクエリ時間で割った結果、数値が大きいほどStarRocksの性能が良いことを意味します。

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
| 合計  | 1236          | 939                 | 2645            | 2.14                    | 10750        | 8.70               |

## テスト手順

ClickHouseのテーブル作成とデータインポートは[公式ドキュメント](https://clickhouse.tech/docs/en/getting-started/example-datasets/star-schema/)を参照してください。StarRocksのデータ生成とインポートのプロセスは以下の通りです：

### データ生成

まず、SSB-POCツールキットをダウンロードしてコンパイルします。

```Bash
wget https://starrocks-public.oss-cn-zhangjiakou.aliyuncs.com/ssb-poc-1.0.zip
unzip ssb-poc-1.0.zip
cd ssb-poc-1.0/
make && make install
cd output/
```

コンパイルが完了すると、関連するすべてのツールがoutputディレクトリにインストールされます。以降のすべての操作はoutputディレクトリで行います。

まず、SSB標準テストセットの`scale factor=100`のデータを生成します。

```Bash
sh bin/gen-ssb.sh 100 data_dir
```

### テーブル構造の作成

`conf/starrocks.conf`設定ファイルを変更し、スクリプトが操作するクラスタのアドレスを指定します。`mysql_host`と`mysql_port`に注目し、その後テーブル作成を実行します。

```SQL
sh bin/create_db_table.sh ddl_100
```

### データインポート

Stream Loadを使用して複数のテーブルデータをインポートし、その後INSERT INTOを使用して複数のテーブルをフラット化して単一テーブルにします。

```Bash
sh bin/flat_insert.sh data_dir
```

### データクエリ

```Bash
sh bin/benchmark.sh ssb-flat
```

### Bitmap Indexの有効化

Bitmap Indexを有効にすると、StarRocksの性能はさらに向上し、特にQ2.2、Q2.3、Q3.3で顕著な改善が見られます。Bitmap Indexを有効にした状態での性能をテストしたい場合は、すべての文字列列にBitmap Indexを作成することができます。具体的な操作は以下の通りです：

1. `lineorder_flat`テーブルを再作成し、作成時にすべてのBitmap Indexを作成します。

    ```SQL
    sh bin/create_db_table.sh ddl_100_bitmap_index
    ```

2. すべてのBEノードの設定ファイルに以下のパラメータを追加し、その後BEを再起動します。

    ```SQL
    bitmap_max_filter_ratio=1000
    ```

3. データインポートコマンドを再実行します。

    ```SQL
    sh bin/flat_insert.sh data_dir
    ```

インポートが完了したら、compactionが完了するまで待ち、その後[4.4 データクエリ](#データクエリ)のステップを再実行します。これでBitmap Indexを有効にした後のクエリ結果になります。

`select CANDIDATES_NUM from information_schema.be_compactions`コマンドを使用してcompactionの進捗を確認できます。3つのBEノードについて、以下の結果がcompactionが完了したことを示します：

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
    c_nation,
    s_nation,
    year(lo_orderdate) AS year,
    sum(lo_revenue) AS revenue 
FROM lineorder_flat 
WHERE c_region = 'ASIA' AND s_region = 'ASIA' AND lo_orderdate >= '1992-01-01'
AND lo_orderdate <= '1997-12-31' 
GROUP BY c_nation, s_nation, year 
ORDER BY year ASC, revenue DESC; 
 
--Q3.2 
SELECT c_city, s_city, year(lo_orderdate) AS year, sum(lo_revenue) AS revenue
FROM lineorder_flat 
WHERE c_nation = 'UNITED STATES' AND s_nation = 'UNITED STATES'
AND lo_orderdate >= '1992-01-01' AND lo_orderdate <= '1997-12-31' 
GROUP BY c_city, s_city, year 
ORDER BY year ASC, revenue DESC; 
 
--Q3.3 
SELECT c_city, s_city, year(lo_orderdate) AS year, sum(lo_revenue) AS revenue 
FROM lineorder_flat 
WHERE c_city IN ('UNITED KI1', 'UNITED KI5') AND s_city IN ('UNITED KI1', 'UNITED KI5')
AND lo_orderdate >= '1992-01-01' AND lo_orderdate <= '1997-12-31' 
GROUP BY c_city, s_city, year 
ORDER BY year ASC, revenue DESC; 
 
--Q3.4 
SELECT c_city, s_city, year(lo_orderdate) AS year, sum(lo_revenue) AS revenue 
FROM lineorder_flat 
WHERE c_city IN ('UNITED KI1', 'UNITED KI5') AND s_city IN ('UNITED KI1', 'UNITED KI5')
AND lo_orderdate >= '1997-12-01' AND lo_orderdate <= '1997-12-31' 
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
AND lo_orderdate >= '1997-01-01' AND lo_orderdate <= '1998-12-31'
AND p_mfgr IN ('MFGR#1', 'MFGR#2') 
GROUP BY year, s_nation, p_category 
ORDER BY year ASC, s_nation ASC, p_category ASC; 
 
--Q4.3 
SELECT year(lo_orderdate) AS year, s_city, p_brand, 
    sum(lo_revenue - lo_supplycost) AS profit 
FROM lineorder_flat 
WHERE s_nation = 'UNITED STATES'
AND lo_orderdate >= '1997-01-01' AND lo_orderdate <= '1998-12-31'
AND p_category = 'MFGR#14' 
GROUP BY year, s_city, p_brand 
ORDER BY year ASC, s_city ASC, p_brand ASC; 
```

### 建表语句

#### `lineorder_flat` デフォルトのテーブル作成

このテーブル作成ステートメントは、現在のクラスタとデータ仕様（3つのBE、scale factor = 100）に合わせています。クラスタにBEノードが多い場合や、より大きなデータ仕様の場合は、バケット数を調整してテーブルを再作成し、データを再インポートすることで、より良いテスト結果を得ることができます。

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
(PARTITION p1 VALUES LESS THAN ('1992-01-01'),
 PARTITION p2 VALUES LESS THAN ('1993-01-01'),
 PARTITION p3 VALUES LESS THAN ('1994-01-01'),
 PARTITION p4 VALUES LESS THAN ('1995-01-01'),
 PARTITION p5 VALUES LESS THAN ('1996-01-01'),
 PARTITION p6 VALUES LESS THAN ('1997-01-01'),
 PARTITION p7 VALUES LESS THAN ('1998-01-01'),
 PARTITION p8 VALUES LESS THAN ('1999-01-01'),
 PARTITION p9 VALUES LESS THAN ('2000-01-01'),
 PARTITION p10 VALUES LESS THAN ('2001-01-01'),
 PARTITION p11 VALUES LESS THAN ('2002-01-01'),
 PARTITION p12 VALUES LESS THAN ('2003-01-01'),
 PARTITION p13 VALUES LESS THAN ('2004-01-01'),
 PARTITION p14 VALUES LESS THAN ('2005-01-01'),
 PARTITION p15 VALUES LESS THAN ('2006-01-01'),
 PARTITION p16 VALUES LESS THAN ('2007-01-01'),
 PARTITION p17 VALUES LESS THAN ('2008-01-01'),
 PARTITION p18 VALUES LESS THAN ('2009-01-01'),
 PARTITION p19 VALUES LESS THAN ('2010-01-01'),
 PARTITION p20 VALUES LESS THAN ('2011-01-01'),
 PARTITION p21 VALUES LESS THAN ('2012-01-01'),
 PARTITION p22 VALUES LESS THAN ('2013-01-01'),
 PARTITION p23 VALUES LESS THAN ('2014-01-01'),
 PARTITION p24 VALUES LESS THAN ('2015-01-01'),
 PARTITION p25 VALUES LESS THAN ('2016-01-01'),
 PARTITION p26 VALUES LESS THAN ('2017-01-01'),
 PARTITION p27 VALUES LESS THAN ('2018-01-01'),
 PARTITION p28 VALUES LESS THAN ('2019-01-01'),
 PARTITION p29 VALUES LESS THAN ('2020-01-01'),
 PARTITION p30 VALUES LESS THAN ('2021-01-01'),
 PARTITION p31 VALUES LESS THAN ('2022-01-01'),
 PARTITION p32 VALUES LESS THAN ('2023-01-01'),
 PARTITION p33 VALUES LESS THAN ('2024-01-01'),
 PARTITION p34 VALUES LESS THAN ('2025-01-01'),
 PARTITION p35 VALUES LESS THAN ('2026-01-01'),
 PARTITION p36 VALUES LESS THAN ('2027-01-01'),
 PARTITION p37 VALUES LESS THAN ('2028-01-01'),
 PARTITION p38 VALUES LESS THAN ('2029-01-01'),
 PARTITION p39 VALUES LESS THAN ('2030-01-01'),
 PARTITION p40 VALUES LESS THAN ('2031-01-01'),
 PARTITION p41 VALUES LESS THAN ('2032-01-01'),
 PARTITION p42 VALUES LESS THAN ('2033-01-01'),
 PARTITION p43 VALUES LESS THAN ('2034-01-01'),
 PARTITION p44 VALUES LESS THAN ('2035-01-01'),
 PARTITION p45 VALUES LESS THAN ('2036-01-01'),
 PARTITION p46 VALUES LESS THAN ('2037-01-01'),
 PARTITION p47 VALUES LESS THAN ('2038-01-01'),
 PARTITION p48 VALUES LESS THAN (MAXVALUE))
DISTRIBUTED BY HASH(`LO_ORDERKEY`) BUCKETS 48
PROPERTIES ("replication_num" = "1");
```

#### `lineorder_flat` Bitmap Index テーブルの作成

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
  INDEX bitmap_lo_orderpriority (`LO_ORDERPRIORITY`) USING BITMAP,
  INDEX bitmap_lo_shipmode (`LO_SHIPMODE`) USING BITMAP,
  INDEX bitmap_c_name (`C_NAME`) USING BITMAP,
  INDEX bitmap_c_address (`C_ADDRESS`) USING BITMAP,
  INDEX bitmap_c_city (`C_CITY`) USING BITMAP,
  INDEX bitmap_c_nation (`C_NATION`) USING BITMAP,
  INDEX bitmap_c_region (`C_REGION`) USING BITMAP,
  INDEX bitmap_c_phone (`C_PHONE`) USING BITMAP,
  INDEX bitmap_c_mktsegment (`C_MKTSEGMENT`) USING BITMAP,
  INDEX bitmap_s_region (`S_REGION`) USING BITMAP,
  INDEX bitmap_s_nation (`S_NATION`) USING BITMAP,
  INDEX bitmap_s_city (`S_CITY`) USING BITMAP,
  INDEX bitmap_s_name (`S_NAME`) USING BITMAP,
  INDEX bitmap_s_address (`S_ADDRESS`) USING BITMAP,
  INDEX bitmap_s_phone (`S_PHONE`) USING BITMAP,
  INDEX bitmap_p_name (`P_NAME`) USING BITMAP,
  INDEX bitmap_p_mfgr (`P_MFGR`) USING BITMAP,
  INDEX bitmap_p_category (`P_CATEGORY`) USING BITMAP,
  INDEX bitmap_p_brand (`P_BRAND`) USING BITMAP,
  INDEX bitmap_p_color (`P_COLOR`) USING BITMAP,
  INDEX bitmap_p_type (`P_TYPE`) USING BITMAP,
  index bitmap_p_container (p_container) USING BITMAP
) ENGINE=OLAP
DUPLICATE KEY(`LO_ORDERDATE`, `LO_ORDERKEY`)
COMMENT "OLAP"
PARTITION BY date_trunc('year', `LO_ORDERDATE`)
DISTRIBUTED BY HASH(`LO_ORDERKEY`) BUCKETS 48
PROPERTIES ("replication_num" = "1");