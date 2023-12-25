---
displayed_sidebar: English
---

# SSBフラットテーブルベンチマーキング

スタースキーマベンチマーク（SSB）は、OLAPデータベース製品の基本的なパフォーマンスメトリクスをテストするために設計されています。SSBは、学術界や業界で広く適用されているスタースキーマテストセットを使用します。詳細については、論文[Star Schema Benchmark](https://www.cs.umb.edu/~poneil/StarSchemaB.PDF)を参照してください。
ClickHouseはスタースキーマをフラットなワイドテーブルに変換し、SSBをシングルテーブルベンチマークに書き換えます。詳細については、[ClickHouseのスタースキーマベンチマーク](https://clickhouse.tech/docs/en/getting-started/example-datasets/star-schema/)を参照してください。
このテストは、StarRocks、Apache Druid、ClickHouseのSSBシングルテーブルデータセットに対するパフォーマンスを比較します。

## テスト結論

- SSB標準データセットで実行された13のクエリにおいて、StarRocksの総合的なクエリパフォーマンスは**ClickHouseの2.1倍、Apache Druidの8.7倍**です。
- StarRocksのBitmap Indexingが有効になると、この機能が無効の場合と比較してパフォーマンスは1.3倍になります。StarRocksの総合的なパフォーマンスは**ClickHouseの2.8倍、Apache Druidの11.4倍**です。

![全体比較](../assets/7.1-1.png)

## テスト準備

### ハードウェア

| マシン             | 3つのクラウドホスト                                               |
| ----------------- | ------------------------------------------------------------ |
| CPU               | 16コアIntel(R) Xeon(R) Platinum 8269CY CPU @2.50GHz <br />キャッシュサイズ: 36608 KB |
| メモリ            | 64 GB                                                        |
| ネットワーク帯域幅 | 5 Gbit/s                                                     |
| ディスク          | ESSD                                                         |

### ソフトウェア

StarRocks、Apache Druid、ClickHouseは同じ構成のホストにデプロイされます。

- StarRocks: 1つのFEと3つのBE。FEはBEと別々にまたはハイブリッドでデプロイできます。
- ClickHouse: 分散テーブルを持つ3ノード
- Apache Druid: 3ノード。1つはMaster ServersとData Serversでデプロイされ、1つはQuery ServersとData Serversでデプロイされ、3つ目はData Serversのみでデプロイされます。

カーネルバージョン: Linux 3.10.0-1160.59.1.el7.x86_64

OSバージョン: CentOS Linuxリリース7.9.2009

ソフトウェアバージョン: StarRocks Communityバージョン3.0、ClickHouse 23.3、Apache Druid 25.0.0

## テストデータと結果

### テストデータ

| テーブル          | レコード数       | 説明              |
| -------------- | ------------ | ------------------------ |
| lineorder      | 6億          | Lineorderファクトテーブル |
| customer       | 300万        | 顧客ディメンションテーブル |
| part           | 140万        | パーツディメンションテーブル |
| supplier       | 20万         | サプライヤーディメンションテーブル |
| dates          | 2,556        | 日付ディメンションテーブル |
| lineorder_flat | 6億          | lineorderフラットテーブル |

### テスト結果

以下の表は13のクエリに対するパフォーマンステスト結果を示しています。クエリレイテンシーの単位はmsです。表ヘッダーの`ClickHouse vs StarRocks`は、ClickHouseのクエリ応答時間をStarRocksのクエリ応答時間で割った値を意味します。値が大きいほど、StarRocksのパフォーマンスが優れていることを示します。

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

ClickHouseテーブルの作成とデータのロード方法の詳細については、[ClickHouse公式ドキュメント](https://clickhouse.tech/docs/en/getting-started/example-datasets/star-schema/)を参照してください。以下のセクションでは、StarRocksのデータ生成とデータロードについて説明します。

### データ生成

ssb-pocツールキットをダウンロードしてコンパイルします。

```Bash
wget https://starrocks-public.oss-cn-zhangjiakou.aliyuncs.com/ssb-poc-1.0.zip
unzip ssb-poc-1.0.zip
cd ssb-poc-1.0/
make && make install
cd output/
```

コンパイル後、関連ツールは`output`ディレクトリにインストールされ、以下の操作はすべてこのディレクトリ内で行われます。

まず、SSB標準データセット`scale factor=100`のデータを生成します。

```Bash
sh bin/gen-ssb.sh 100 data_dir
```

### テーブルスキーマの作成

1. `conf/starrocks.conf`設定ファイルを変更し、クラスターアドレスを指定します。`mysql_host`と`mysql_port`に特に注意してください。

2. 次のコマンドを実行してテーブルを作成します：

    ```SQL
    sh bin/create_db_table.sh ddl_100
    ```

### データクエリ

```Bash
sh bin/benchmark.sh ssb-flat
```

### Bitmap Indexingの有効化

Bitmap Indexingを有効にするとStarRocksのパフォーマンスが向上します。特にQ2.2、Q2.3、Q3.3でBitmap Indexingを有効にしたStarRocksのパフォーマンスをテストしたい場合は、すべてのSTRINGカラムにBitmap Indexesを作成できます。

1. 別の`lineorder_flat`テーブルを作成し、Bitmap Indexesを作成します。

    ```SQL
    sh bin/create_db_table.sh ddl_100_bitmap_index
    ```

2. すべてのBEの`be.conf`ファイルに以下の設定を追加し、設定が有効になるようにBEを再起動します。

    ```SQL
    bitmap_max_filter_ratio=1000
    ```

3. データロードスクリプトを実行します。

    ```SQL
    sh bin/flat_insert.sh data_dir
    ```

データがロードされた後、データバージョンのコンパクションが完了するまで待ち、その後[4.4データクエリ](#query-data)を再度実行してBitmap Indexingが有効になった後のデータをクエリします。

データバージョンコンパクションの進行状況を確認するには、`select CANDIDATES_NUM from information_schema.be_compactions`を実行します。3つのBEノードについて、以下の結果はコンパクションが完了したことを示しています：

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

## SQLテストとテーブル作成ステートメント

### テストSQL

```SQL
--Q1.1 
SELECT sum(lo_extendedprice * lo_discount) AS `revenue` 
FROM lineorder_flat 
WHERE lo_orderdate >= '1993-01-01' AND lo_orderdate <= '1993-12-31'
AND lo_discount BETWEEN 1 AND 3 AND lo_quantity < 25; 
 
--Q1.2 
SELECT sum(lo_extendedprice * lo_discount) AS revenue FROM lineorder_flat  
WHERE lo_orderdate >= '1994-01-01' AND lo_orderdate <= '1994-01-31'
AND lo_discount BETWEEN 4 AND 6 AND lo_quantity BETWEEN 26 AND 35; 
 
--Q1.3 
SELECT sum(lo_extendedprice * lo_discount) AS revenue 
FROM lineorder_flat 
WHERE weekofyear(lo_orderdate) = 6
AND lo_orderdate >= '1994-01-01' AND lo_orderdate <= '1994-12-31' 
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
    sum(lo_revenue) AS revenue FROM lineorder_flat 
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

### テーブル作成ステートメント

#### デフォルトの `lineorder_flat` テーブル

以下のステートメントは、現在のクラスターサイズとデータサイズ（3つのBE、スケールファクター=100）に合わせています。クラスターにより多くのBEノードがある場合や、データサイズが大きい場合は、バケット数を調整し、テーブルを再作成し、データを再ロードすることで、より良いテスト結果を得ることができます。

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
(PARTITION p1 VALUES LESS THAN ('1995-01-01'),
 PARTITION p2 VALUES LESS THAN ('1996-01-01'),
 PARTITION p3 VALUES LESS THAN ('1997-01-01'),
 PARTITION p4 VALUES LESS THAN MAXVALUE)
DISTRIBUTED BY HASH(`LO_ORDERKEY`) BUCKETS 48
PROPERTIES ("replication_num" = "1");
```

#### ビットマップインデックスを持つ `lineorder_flat` テーブル

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
  INDEX `bitmap_lo_orderpriority` (`LO_ORDERPRIORITY`) USING BITMAP,
  INDEX `bitmap_lo_shipmode` (`LO_SHIPMODE`) USING BITMAP,
  INDEX `bitmap_c_name` (`C_NAME`) USING BITMAP,
  INDEX `bitmap_c_address` (`C_ADDRESS`) USING BITMAP,
  INDEX `bitmap_c_city` (`C_CITY`) USING BITMAP,
  INDEX `bitmap_c_nation` (`C_NATION`) USING BITMAP,
  INDEX `bitmap_c_region` (`C_REGION`) USING BITMAP,
  INDEX `bitmap_c_phone` (`C_PHONE`) USING BITMAP,
  INDEX `bitmap_c_mktsegment` (`C_MKTSEGMENT`) USING BITMAP,
  INDEX `bitmap_s_region` (`S_REGION`) USING BITMAP,
  INDEX `bitmap_s_nation` (`S_NATION`) USING BITMAP,
  INDEX `bitmap_s_city` (`S_CITY`) USING BITMAP,
  INDEX `bitmap_s_name` (`S_NAME`) USING BITMAP,
  INDEX `bitmap_s_address` (`S_ADDRESS`) USING BITMAP,
  INDEX `bitmap_s_phone` (`S_PHONE`) USING BITMAP,
  INDEX `bitmap_p_name` (`P_NAME`) USING BITMAP,
  INDEX `bitmap_p_mfgr` (`P_MFGR`) USING BITMAP,
  INDEX `bitmap_p_category` (`P_CATEGORY`) USING BITMAP,
  INDEX `bitmap_p_brand` (`P_BRAND`) USING BITMAP,
  INDEX `bitmap_p_color` (`P_COLOR`) USING BITMAP,
  INDEX `bitmap_p_type` (`P_TYPE`) USING BITMAP,
  INDEX `bitmap_p_container` (`P_CONTAINER`) USING BITMAP
) ENGINE=OLAP
DUPLICATE KEY(`LO_ORDERDATE`, `LO_ORDERKEY`)
COMMENT "OLAP"
PARTITION BY RANGE(`LO_ORDERDATE`)
(PARTITION p1 VALUES LESS THAN ('1995-01-01'),
 PARTITION p2 VALUES LESS THAN ('1996-01-01'),
 PARTITION p3 VALUES LESS THAN ('1997-01-01'),
 PARTITION p4 VALUES LESS THAN MAXVALUE)
DISTRIBUTED BY HASH(`LO_ORDERKEY`) BUCKETS 48
PROPERTIES ("replication_num" = "1");
) ENGINE=OLAP
DUPLICATE KEY(`LO_ORDERDATE`, `LO_ORDERKEY`)
COMMENT "OLAP"
PARTITION BY date_trunc('year', `LO_ORDERDATE`)
DISTRIBUTED BY HASH(`LO_ORDERKEY`) BUCKETS 48
PROPERTIES ("replication_num" = "1");