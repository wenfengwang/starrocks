---
displayed_sidebar: "Japanese"
---

# SSBフラットテーブルのベンチマーク

スタースキーマベンチマーク（SSB）はOLAPデータベース製品の基本的なパフォーマンスメトリクスをテストするために設計されています。SSBは広く学術界や産業界で使用されているスタースキーマテストセットを使用しています。詳細については、[Star Schema Benchmark](https://www.cs.umb.edu/~poneil/StarSchemaB.PDF)の論文を参照してください。
ClickHouseはスタースキーマをワイドなフラットテーブルに展開し、SSBをシングルテーブルベンチマークへと書き換えます。詳細については、[ClickHouseのスタースキーマベンチマーク](https://clickhouse.tech/docs/en/getting-started/example-datasets/star-schema/)を参照してください。
このテストは、StarRocks、Apache Druid、ClickHouseの性能をSSBのシングルテーブルデータセットに対して比較します。

## テストの結論

- SSB標準データセットに対して実行された13のクエリの中で、StarRocksは全体的なクエリパフォーマンスで**ClickHouseの2.1倍、Apache Druidの8.7倍**を示しました。
- StarRocksのBitmap Indexingが有効になると、パフォーマンスは無効の場合と比較して1.3倍となります。StarRocksの全体的なパフォーマンスは**ClickHouseの2.8倍、Apache Druidの11.4倍**です。

![全体的な比較](../assets/7.1-1.png)

## テストの準備

### ハードウェア

| マシン            | 3つのクラウドホスト                                 |
| ----------------- | --------------------------------------------------- |
| CPU               | 16コアインテル（R）Xeon（R）Platinum 8269CY CPU @2.50GHz <br />キャッシュサイズ：36608 KB |
| メモリ            | 64 GB                                               |
| ネットワーク帯域幅 | 5 Gbit/s                                            |
| ディスク          | ESSD                                                |

### ソフトウェア

StarRocks、Apache Druid、ClickHouseは同じ構成のホストに展開されています。

- StarRocks：FE 1つ、BE 3つ。FEは個別またはハイブリッドでBEと展開できます。
- ClickHouse：分散テーブルを持つ3つのノード
- Apache Druid：3つのノード。1つはマスターサーバーとデータサーバーと共に展開され、1つはクエリサーバーとデータサーバーと共に展開され、3つ目はデータサーバーのみで展開されます。

カーネルバージョン：Linux 3.10.0-1160.59.1.el7.x86_64

OSバージョン：CentOS Linuxリリース7.9.2009

ソフトウェアバージョン：StarRocks Community Version 3.0、ClickHouse 23.3、Apache Druid 25.0.0

## テストデータと結果

### テストデータ

| テーブル         | レコード数   | 説明                    |
| -------------- | ------------ | ---------------------- |
| lineorder      | 6億           | 受注事実テーブル           |
| customer       | 300万         | 顧客ディメンションテーブル  |
| part           | 140万         | パーツディメンションテーブル |
| supplier       | 20万          | サプライヤーディメンションテーブル |
| dates          | 2,556        | 日付ディメンションテーブル   |
| lineorder_flat | 6億           | 縦持ちフラットテーブル       |

### テスト結果

以下の表は、13のクエリに対するパフォーマンステスト結果を示しています。クエリレイテンシの単位はミリ秒です。表ヘッダーの`ClickHouse vs StarRocks`は、ClickHouseのクエリ応答時間をStarRocksのクエリ応答時間で割ったものです。大きな値ほどStarRocksのパフォーマンスが良いことを示します。

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
| 合計 | 1236          | 939                 | 2645                | 2.14                    | 10750            | 8.70               |

## テスト手順

ClickHouseテーブルの作成方法やデータのロード方法の詳細については、[ClickHouse公式ドキュメント](https://clickhouse.tech/docs/en/getting-started/example-datasets/star-schema/)を参照してください。以下のセクションでは、StarRocksのデータ生成とデータロードについて説明します。

### データ生成

ssb-pocツールキットをダウンロードし、コンパイルします。

```Bash
wget https://starrocks-public.oss-cn-zhangjiakou.aliyuncs.com/ssb-poc-1.0.zip
unzip ssb-poc-1.0.zip
cd ssb-poc-1.0/
make && make install
cd output/
```

コンパイル後、関連するツールが`output`ディレクトリにインストールされ、以下の操作はすべてこのディレクトリで行います。

まず、SSB標準データセット`スケールファクター=100`のデータを生成します。

```Bash
sh bin/gen-ssb.sh 100 data_dir
```

### テーブルスキーマの作成

1. 構成ファイル`conf/starrocks.conf`を修正し、クラスターアドレスを指定します。`mysql_host`と`mysql_port`に特に注意してください。

2. 次のコマンドを実行してテーブルを作成します。

    ```SQL
    sh bin/create_db_table.sh ddl_100
    ```

### データのクエリ

```Bash
sh bin/benchmark.sh ssb-flat
```

### ビットマップインデックスの有効化

StarRocksはビットマップインデックスを有効にするとより良いパフォーマンスを発揮します。特にQ2.2、Q2.3、およびQ3.3のパフォーマンスをStarRocksのビットマップインデックスが有効な状態でテストしたい場合は、すべてのSTRING列にビットマップインデックスを作成することができます。

1. 別の`lineorder_flat`テーブルを作成し、ビットマップインデックスを作成します。

    ```SQL
    sh bin/create_db_table.sh ddl_100_bitmap_index
    ```

2. すべてのBEの`be.conf`ファイルに以下の構成を追加し、構成が有効になるようにBEを再起動します。

    ```SQL
    bitmap_max_filter_ratio=1000
    ```

3. データの読み込みスクリプトを実行します。

    ```SQL
    sh bin/flat_insert.sh data_dir
    ```

データがロードされたら、データバージョンのコンパクションが完了するのを待ち、[4.4](#データのクエリ)を再度実行して、ビットマップインデックスが有効になった状態でデータをクエリしてください。

データバージョンのコンパクションの進行状況は、`select CANDIDATES_NUM from information_schema.be_compactions`を実行して確認できます。BEノードの3つについて、以下の結果がコンパクションが完了したことを示します。

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

## テストSQLおよびテーブル作成ステートメント

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
```SQL
SELECT c_city, s_city, year(lo_orderdate) AS year, sum(lo_revenue) AS revenue
FROM lineorder_flat 
WHERE c_nation = 'UNITED STATES' AND s_nation = 'UNITED STATES'
AND lo_orderdate  >= '1992-01-01' AND lo_orderdate <= '1997-12-31' 
GROUP BY c_city, s_city, year 
ORDER BY year ASC, revenue DESC; 

SELECT c_city, s_city, year(lo_orderdate) AS year, sum(lo_revenue) AS revenue 
FROM lineorder_flat 
WHERE c_city in ( 'UNITED KI1' ,'UNITED KI5') AND s_city in ('UNITED KI1', 'UNITED KI5')
AND lo_orderdate  >= '1992-01-01' AND lo_orderdate <= '1997-12-31' 
GROUP BY c_city, s_city, year 
ORDER BY year ASC, revenue DESC; 

SELECT c_city, s_city, year(lo_orderdate) AS year, sum(lo_revenue) AS revenue 
FROM lineorder_flat 
WHERE c_city in ('UNITED KI1', 'UNITED KI5') AND s_city in ('UNITED KI1', 'UNITED KI5')
AND lo_orderdate  >= '1997-12-01' AND lo_orderdate <= '1997-12-31' 
GROUP BY c_city, s_city, year 
ORDER BY year ASC, revenue DESC; 

SELECT year(lo_orderdate) AS year, c_nation, sum(lo_revenue - lo_supplycost) AS profit
FROM lineorder_flat 
WHERE c_region = 'AMERICA' AND s_region = 'AMERICA' AND p_mfgr in ('MFGR#1', 'MFGR#2') 
GROUP BY year, c_nation 
ORDER BY year ASC, c_nation ASC; 

SELECT year(lo_orderdate) AS year, 
    s_nation, p_category, sum(lo_revenue - lo_supplycost) AS profit 
FROM lineorder_flat 
WHERE c_region = 'AMERICA' AND s_region = 'AMERICA'
AND lo_orderdate >= '1997-01-01' and lo_orderdate <= '1998-12-31'
AND p_mfgr in ( 'MFGR#1' , 'MFGR#2') 
GROUP BY year, s_nation, p_category 
ORDER BY year ASC, s_nation ASC, p_category ASC; 

SELECT year(lo_orderdate) AS year, s_city, p_brand, 
    sum(lo_revenue - lo_supplycost) AS profit 
FROM lineorder_flat 
WHERE s_nation = 'UNITED STATES'
AND lo_orderdate >= '1997-01-01' and lo_orderdate <= '1998-12-31'
AND p_category = 'MFGR#14' 
GROUP BY year, s_city, p_brand 
ORDER BY year ASC, s_city ASC, p_brand ASC; 
```
```
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