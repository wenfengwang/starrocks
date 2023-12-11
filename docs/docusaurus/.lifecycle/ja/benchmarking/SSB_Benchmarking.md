---
displayed_sidebar: "Japanese"
---

# SSBフラットテーブルのベンチマーキング

スタースキーマベンチマーク（SSB）は、OLAPデータベース製品の基本的なパフォーマンスメトリクスをテストするために設計されています。SSBは、学術界や産業界で広く適用されているスタースキーマのテストセットを使用しています。詳細については、[Star Schema Benchmark](https://www.cs.umb.edu/~poneil/StarSchemaB.PDF)を参照してください。
ClickHouseは、スタースキーマをワイドなフラットテーブルに展開し、SSBを単一テーブルのベンチマークに書き換えます。詳細については、[ClickHouseのStar schema benchmark](https://clickhouse.tech/docs/en/getting-started/example-datasets/star-schema/)を参照してください。
このテストは、StarRocks、Apache Druid、およびClickHouseのパフォーマンスをSSB単一テーブルデータセットに対して比較します。

## テスト結果

- SSB標準データセットで行われた13のクエリのうち、StarRocksは全体的なクエリパフォーマンスで**ClickHouseの2.1倍、Apache Druidの8.7倍**を達成しています。
- StarRocksのBitmapインデックス化を有効にした場合、そのパフォーマンスは無効にした場合に比べて1.3倍であり、全体的なパフォーマンスは**ClickHouseの2.8倍、Apache Druidの11.4倍**となります。

![総合比較](../assets/7.1-1.png)

## テスト準備

### ハードウェア

| マシン        | 3つのクラウドホスト                                             |
| ------------- | ------------------------------------------------------------ |
| CPU           | 16コアIntel（R）Xeon（R）Platinum 8269CY CPU @2.50GHz <br />キャッシュサイズ：36608 KB |
| メモリ        | 64 GB                                                        |
| ネットワーク帯域幅 | 5 Gbit/s                                                     |
| ディスク      | ESSD                                                         |

### ソフトウェア

StarRocks、Apache Druid、およびClickHouseは、同じ構成のホスト上に展開されています。

- StarRocks：1つのFEと3つのBE。 FEは、BEとは別々またはハイブリッドで展開できます。
- ClickHouse：分散テーブルを持つ3つのノード
- Apache Druid：3つのノード。1つはマスターサーバーとデータサーバーとともに展開され、1つはクエリサーバーとデータサーバーとともに展開され、3つめはデータサーバーのみで展開されます。

カーネルバージョン：Linux 3.10.0-1160.59.1.el7.x86_64

OSバージョン：CentOS Linuxリリース7.9.2009

ソフトウェアバージョン：StarRocks Community Version 3.0、ClickHouse 23.3、Apache Druid 25.0.0

## テストデータと結果

### テストデータ

| テーブル          | レコード数         | 説明                     |
| -------------- | -------------- | ------------------------ |
| lineorder      | 6億件           | 注文ファクトテーブル           |
| customer       | 300万件         | 顧客ディメンジョンテーブル       |
| part           | 140万件         | 部品ディメンジョンテーブル       |
| supplier       | 20万件          | 仕入先ディメンジョンテーブル     |
| dates          | 2,556件         | 日付ディメンジョンテーブル       |
| lineorder_flat | 6億件           | ラインオーダーフラットテーブル     |

### テスト結果

以下の表は、13のクエリにおけるパフォーマンステストの結果を示しています。クエリ遅延の単位はミリ秒（ms）です。表ヘッダの「ClickHouse vs StarRocks」は、ClickHouseのクエリ応答時間をStarRocksのクエリ応答時間で除したものです。大きな値ほどStarRocksのパフォーマンスが良いことを示します。

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
| 合計  | 1236          | 939                 | 2645                | 2.14                    | 10750            | 8.70               |

## テスト手順

ClickHouseのテーブルの作成とデータのロード方法についての詳細については、[ClickHouse公式ドキュメント](https://clickhouse.tech/docs/en/getting-started/example-datasets/star-schema/)を参照してください。以下のセクションでは、StarRocksのデータ生成とデータのロードについて説明します。

### データ生成

ssb-pocツールキットをダウンロードし、コンパイルします。

```Bash
wget https://starrocks-public.oss-cn-zhangjiakou.aliyuncs.com/ssb-poc-1.0.zip
unzip ssb-poc-1.0.zip
cd ssb-poc-1.0/
make && make install
cd output/
```

コンパイル後、関連するツールはすべて`output`ディレクトリにインストールされます。以下の操作はすべてこのディレクトリ内で実行されます。

まず、SSB標準データセットのデータを生成します（スケールファクター=100）。

```Bash
sh bin/gen-ssb.sh 100 data_dir
```

### テーブルスキーマの作成

1. 構成ファイル`conf/starrocks.conf`を修正し、クラスターのアドレスを指定します。特に`mysql_host`と`mysql_port`に注意してください。

2. 次のコマンドを実行して、テーブルを作成します：

    ```SQL
    sh bin/create_db_table.sh ddl_100
    ```

### データのクエリ

```Bash
sh bin/benchmark.sh ssb-flat
```

### Bitmapインデックスの有効化

Bitmapインデックスを有効にすると、StarRocksのパフォーマンスが向上します。特に、Q2.2、Q2.3、およびQ3.3におけるStarRocksのパフォーマンスをテストしたい場合は、すべてのSTRINGカラムにBitmapインデックスを作成できます。

1. 別の`lineorder_flat`テーブルを作成し、Bitmapインデックスを作成します。

    ```SQL
    sh bin/create_db_table.sh ddl_100_bitmap_index
    ```

2. すべてのBEの`be.conf`ファイルに次の設定を追加し、設定が有効になるようにBEを再起動します。

    ```SQL
    bitmap_max_filter_ratio=1000
    ```

3. データローディングスクリプトを実行します。

    ```SQL
    sh bin/flat_insert.sh data_dir
    ```

データがロードされたら、データバージョンのコンパクションが完了するのを待ってから、[4.4](#データのクエリ)を再実行して、Bitmapインデックスを有効にした後のデータをクエリします。

`select CANDIDATES_NUM from information_schema.be_compactions`を実行することで、データバージョンのコンパクションの進捗状況を表示できます。3つのBEノードに対して、以下の結果がコンパクションが完了したことを示しています：

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
```
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

### テーブル作成文

#### デフォルトの `lineorder_flat` テーブル

次の文は、現在のクラスターサイズとデータサイズに一致する（BE ノード 3 台、スケールファクター 100）データを持っています。クラスターに BE ノードが複数ある場合や、より大きなデータを持つ場合は、バケツ（BUCKETS）の数を調整してテーブルを再作成し、データを再読み込みして、より良いテスト結果を得ることができます。

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

#### ビットマップインデックスを含む `lineorder_flat` テーブル

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