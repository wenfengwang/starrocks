---
displayed_sidebar: "Japanese"
sidebar_position: 3
description: Apache Icebergを使用したData Lakehouse
toc_max_heading_level: 2
---

# Apache Icebergを使用したData Lakehouse
../assets/quick-start/_clientsCompose.mdxからClientsをインポート

## 概要

- Docker composeを使用してオブジェクトストレージ、Apache Spark、Icebergカタログ、およびStarRocksを展開します
- 2023年5月のニューヨークシティグリーンタクシーデータをIcebergデータレイクにロードします
- StarRocksを構成してIcebergカタログにアクセスします
- データの問い合わせはデータが配置されている場所でStarRocksを使用します

## 必要条件

### Docker

- [Docker](https://docs.docker.com/engine/install/)
- Dockerに割り当てられたRAMが5GB
- Dockerに割り当てられた空きディスク容量が20GB

### SQLクライアント

Docker環境で提供されるSQLクライアントを使用するか、またはシステム上のクライアントを使用してください。多くのMySQL互換のクライアントが動作しますが、このガイドではDBeaverとMySQL WorkBenchの構成について説明します。

### curl

`curl`はデータセットをダウンロードするために使用されます。OSプロンプトで`curl`または`curl.exe`を実行してインストールされているかどうかを確認してください。curlがインストールされていない場合は、[ここでcurlを取得してください](https://curl.se/dlwiz/?type=bin) 。

---

## StarRocks用語

### FE
フロントエンドノードは、メタデータ管理、クライアント接続管理、クエリプランニング、およびクエリスケジューリングを担当します。各FEはそのメモリにメタデータの完全なコピーを保持および維持し、これによりFE間の無差別サービスが保証されます。

### BE
バックエンドノードは、共有無しデプロイメントでのデータストレージおよびクエリプランの実行を担当します。このガイドで使用されるIcebergカタログのような外部カタログを使用する場合、ローカルのデータのみがBEノード（複数）に保存されます。

---

## 環境

このガイドでは、Docker composeで展開される6つのコンテナ（サービス）が使用され、それぞれが以下の責任を持ちます。

| サービス           | 責任                                                              |
|-------------------|-------------------------------------------------------------------|
| **`starrocks-fe`**| メタデータ管理、クライアント接続、クエリプランとスケジューリング|
| **`starrocks-be`**| クエリプランの実行                                              |
| **`rest`**        | Icebergカタログ（メタデータサービス）の提供                                              |
| **`spark-iceberg`**| PySparkを実行するためのApache Spark環境                        |
| **`mc`**          | MinIO構成（MinIOコマンドラインクライアント）                    |
| **`minio`**       | MinIOオブジェクトストレージ                                      |

## Docker構成およびNYC Green Taxiデータのダウンロード

このガイドで必要な3つのコンテナを提供するために、StarRocksはDocker composeファイルを提供し、curlでそのファイルとデータセットをダウンロードします。

Docker composeファイル：
```bash
mkdir iceberg
cd iceberg
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/iceberg/docker-compose.yml
```

そしてデータセット：
```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/iceberg/datasets/green_tripdata_2023-05.parquet
```

## Dockerで環境を起動する

:::tip
このコマンドと他の`docker compose`コマンドは、`docker-compose.yml`ファイルを含むディレクトリから実行してください。
:::

```bash
docker compose up -d
```

```plaintext
[+] Building 0.0s (0/0)                     docker:desktop-linux
[+] Running 6/6
 ✔ Container iceberg-rest   Started                         0.0s
 ✔ Container minio          Started                         0.0s
 ✔ Container starrocks-fe   Started                         0.0s
 ✔ Container mc             Started                         0.0s
 ✔ Container spark-iceberg  Started                         0.0s
 ✔ Container starrocks-be   Started
```

## 環境の状態を確認する

サービスの進捗状況を確認してください。FEとBEの健全性を確認するには、約30秒かかります。

FEとBEが`健全`の状態を示すまで`docker compose ps`を実行してください。他のサービスにはヘルスチェックの構成がされていませんが、それらとやり取りすることになり、それが機能しているかどうかがわかります。

:::tip
`jq`をインストール済みで`docker compose ps`から短いリストを希望する場合は以下をお試しください。

```bash
docker compose ps --format json | jq '{Service: .Service, State: .State, Status: .Status}'
```

:::

```bash
docker compose ps
```

```bash
SERVICE         CREATED         STATUS                   PORTS
rest            4 minutes ago   Up 4 minutes             0.0.0.0:8181->8181/tcp
mc              4 minutes ago   Up 4 minutes
minio           4 minutes ago   Up 4 minutes             0.0.0.0:9000-9001->9000-9001/tcp
spark-iceberg   4 minutes ago   Up 4 minutes             0.0.0.0:8080->8080/tcp, 0.0.0.0:8888->8888/tcp, 0.0.0.0:10000-10001->10000-10001/tcp
starrocks-be    4 minutes ago   Up 4 minutes (healthy)   0.0.0.0:8040->8040/tcp
starrocks-fe    4 minutes ago   Up 4 minutes (healthy)   0.0.0.0:8030->8030/tcp, 0.0.0.0:9020->9020/tcp, 0.0.0.0:9030->9030/tcp
```

---

## PySpark

Icebergとのインタラクションには複数の方法がありますが、このガイドではPySparkを使用しています。PySparkをご存じでない場合は、追加情報セクションからリンクされたドキュメントをご覧ください。ただし、実行する必要のあるすべてのコマンドは以下に記載されています。

### グリーンタクシーデータセット

データをspark-icebergコンテナにコピーします。このコマンドは、データセットファイルを`spark-iceberg`サービスの`/opt/spark/`ディレクトリにコピーします。

```bash
docker compose \
cp green_tripdata_2023-05.parquet spark-iceberg:/opt/spark/
```

### PySparkを起動

このコマンドは`spark-iceberg`サービスに接続し、`pyspark`コマンドを実行します。

```bash
docker compose exec -it spark-iceberg pyspark
```

```py
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 3.5.0
      /_/

Using Python version 3.9.18 (main, Nov  1 2023 11:04:44)
Spark context Web UI available at http://6ad5cb0e6335:4041
Spark context available as 'sc' (master = local[*], app id = local-1701967093057).
SparkSession available as 'spark'.
>>>
```

### データフレームにデータセットを読み込む

データフレームはSpark SQLの一部であり、データベーステーブルやスプレッドシートに類似したデータ構造を提供します。

グリーンタクシーデータはParquet形式でNYC Taxi and Limousine Commissionが提供しています。ディスクからファイルを`df`という名前のデータフレームに読み込み、データの最初の数行の最初の数列を選択して最初の数行のデータのスキーマを表示します。これらのコマンドは`pyspark`セッションで実行する必要があります。

- データセットファイルをデータフレーム`df`に読み込む
- Parquetファイルのスキーマを表示

```py
df = spark.read.parquet("/opt/spark/green_tripdata_2023-05.parquet")
df.printSchema()
```

```plaintext
root
 |-- VendorID: integer (nullable = true)
 |-- lpep_pickup_datetime: timestamp_ntz (nullable = true)
 |-- lpep_dropoff_datetime: timestamp_ntz (nullable = true)
 |-- store_and_fwd_flag: string (nullable = true)
 |-- RatecodeID: long (nullable = true)
 |-- PULocationID: integer (nullable = true)
 |-- DOLocationID: integer (nullable = true)
 |-- passenger_count: long (nullable = true)
 |-- trip_distance: double (nullable = true)
 |-- fare_amount: double (nullable = true)
 |-- extra: double (nullable = true)
 |-- mta_tax: double (nullable = true)
 |-- tip_amount: double (nullable = true)
 |-- tolls_amount: double (nullable = true)
 |-- ehail_fee: double (nullable = true)
 |-- improvement_surcharge: double (nullable = true)
 |-- total_amount: double (nullable = true)
 |-- payment_type: long (nullable = true)
 |-- trip_type: long (nullable = true)
 |-- congestion_surcharge: double (nullable = true)

>>>
```
最初の数行の最初の数列を選択して最初の数行のデータを表示：

```python
df.select(df.columns[:7]).show(3)
```
```plaintext
+--------+--------------------+---------------------+------------------+----------+------------+------------+
|VendorID|lpep_pickup_datetime|lpep_dropoff_datetime|store_and_fwd_flag|RatecodeID|PULocationID|DOLocationID|
+--------+--------------------+---------------------+------------------+----------+------------+------------+
|       2| 2023-05-01 00:52:10|  2023-05-01 01:05:26|                 N|         1|         244|         213|
|       2| 2023-05-01 00:29:49|  2023-05-01 00:50:11|                 N|         1|          33|         100|
|       2| 2023-05-01 00:25:19|  2023-05-01 00:32:12|                 N|         1|         244|         244|
+--------+--------------------+---------------------+------------------+----------+------------+------------+
only showing top 3 rows
```
### テーブルに書き込む

このステップで作成されたテーブルは、次のステップでStarRocksで利用可能になるカタログに入ります。

- カタログ: `demo`
- データベース: `nyc`
- テーブル: `greentaxis`

```python
df.writeTo("demo.nyc.greentaxis").create()
```

## StarRocksにIcebergカタログへのアクセスを設定する

PySparkからログアウトするか、新しいターミナルを開いてSQLコマンドを実行することができます。別のターミナルを開いた場合は、続行する前に`docker-compose.yml`ファイルを含む`quickstart`ディレクトリに移動してください。

### SQLクライアントでStarRocksに接続する

#### SQLクライアント

<Clients />

---

PySparkセッションを終了してStarRocksに接続できます。

:::tip

このコマンドを`docker-compose.yml`ファイルを含むディレクトリから実行してください。

mysql CLI以外のクライアントを使用している場合は、そのクライアントを開いてください。
:::

```bash
docker compose exec starrocks-fe \
  mysql -P 9030 -h 127.0.0.1 -u root --prompt="StarRocks > "
```

```plaintext
StarRocks >
```

### 外部カタログを作成する

外部カタログは、StarRocksがIcebergデータをStarRocksデータベースとテーブルで操作できるようにする構成です。個々の構成プロパティは、コマンドの後に詳細が記載されます。

```sql
CREATE EXTERNAL CATALOG 'iceberg'
PROPERTIES
(
  "type"="iceberg",
  "iceberg.catalog.type"="rest",
  "iceberg.catalog.uri"="http://iceberg-rest:8181",
  "iceberg.catalog.warehouse"="warehouse",
  "aws.s3.access_key"="admin",
  "aws.s3.secret_key"="password",
  "aws.s3.endpoint"="http://minio:9000",
  "aws.s3.enable_path_style_access"="true",
  "client.factory"="com.starrocks.connector.iceberg.IcebergAwsClientFactory"
);
```

#### PROPERTIES

|    プロパティ                      |     説明                                                                               |
|:---------------------------------|:----------------------------------------------------------------------------------------------|
|`type`                            | この例では`iceberg`タイプです。その他のオプションには、Hive、Hudi、Delta Lake、JDBCなどがあります。|
|`iceberg.catalog.type`            | この例では`rest`が使用されています。Tabularは使用されるDockerイメージを提供し、TabularはRESTを使用します。|
|`iceberg.catalog.uri`             | RESTサーバーエンドポイント。|
|`iceberg.catalog.warehouse`       | Icebergカタログの識別子。この例ではコンポーズファイルで指定された倉庫の名前は`warehouse`です。 |
|`aws.s3.access_key`               | MinIOキー。この場合、キーとパスワードは`admin`に設定されています。|
|`aws.s3.secret_key`               | および`password`です。 |
|`aws.s3.endpoint`                 | MinIOエンドポイント。|
|`aws.s3.enable_path_style_access` | オブジェクトストレージとしてMinIOを使用する場合、これが必要です。MinIOはこのフォーマット`http://host:port/<bucket_name>/<key_name>`を期待しています。 |
|`client.factory`                  | このプロパティを`iceberg.IcebergAwsClientFactory`を使用するように設定すると、認証に`aws.s3.access_key`と`aws.s3.secret_key`パラメータが使用されます。 |

```sql
SHOW CATALOGS;
```

```plaintext
+-----------------+----------+------------------------------------------------------------------+
| Catalog         | Type     | Comment                                                          |
+-----------------+----------+------------------------------------------------------------------+
| default_catalog | Internal | 内部カタログには、このクラスターのセルフマネージドテーブルが含まれています。 |
| iceberg         | Iceberg  | NULL                                                             |
+-----------------+----------+------------------------------------------------------------------+
2 rows in set (0.03 sec)
```

```sql
SET CATALOG iceberg;
```

```sql
SHOW DATABASES;
```
:::tip
表示されるデータベースは、PySparkで作成されました。`iceberg`カタログを追加すると、StarRocksで`nyc`データベースが表示されます。
:::

```plaintext
+----------+
| Database |
+----------+
| nyc      |
+----------+
1 row in set (0.07 sec)
```

```sql
USE nyc;
```

```plaintext
次のテーブルデータベースの補完情報のためにテーブルと列名の読み取りをオフにできます

この機能をオフにして起動時間を短縮するには、-Aを使用できます
データベースが変更されました
```

```sql
SHOW TABLES;
```

```plaintext
+---------------+
| Tables_in_nyc |
+---------------+
| greentaxis    |
+---------------+
1 rows in set (0.05 sec)
```

```sql
DESCRIBE greentaxis;
```

:::tip
StarRocksが使用するスキーマと、以前のPySparkセッションの`df.printSchema()`の出力を比較してください。Sparkの`timestamp_ntz`データ型はStarRocksの`DATETIME`などとして表されます。
:::

```plaintext
+-----------------------+------------------+------+-------+---------+-------+
| Field                 | Type             | Null | Key   | Default | Extra |
+-----------------------+------------------+------+-------+---------+-------+
| VendorID              | INT              | Yes  | false | NULL    |       |
| lpep_pickup_datetime  | DATETIME         | Yes  | false | NULL    |       |
| lpep_dropoff_datetime | DATETIME         | Yes  | false | NULL    |       |
| store_and_fwd_flag    | VARCHAR(1048576) | Yes  | false | NULL    |       |
| RatecodeID            | BIGINT           | Yes  | false | NULL    |       |
| PULocationID          | INT              | Yes  | false | NULL    |       |
| DOLocationID          | INT              | Yes  | false | NULL    |       |
| passenger_count       | BIGINT           | Yes  | false | NULL    |       |
| trip_distance         | DOUBLE           | Yes  | false | NULL    |       |
| fare_amount           | DOUBLE           | Yes  | false | NULL    |       |
| extra                 | DOUBLE           | Yes  | false | NULL    |       |
| mta_tax               | DOUBLE           | Yes  | false | NULL    |       |
| tip_amount            | DOUBLE           | Yes  | false | NULL    |       |
| tolls_amount          | DOUBLE           | Yes  | false | NULL    |       |
| ehail_fee             | DOUBLE           | Yes  | false | NULL    |       |
| improvement_surcharge | DOUBLE           | Yes  | false | NULL    |       |
| total_amount          | DOUBLE           | Yes  | false | NULL    |       |
| payment_type          | BIGINT           | Yes  | false | NULL    |       |
| trip_type             | BIGINT           | Yes  | false | NULL    |       |
| congestion_surcharge  | DOUBLE           | Yes  | false | NULL    |       |
+-----------------------+------------------+------+-------+---------+-------+
20 rows in set (0.04 sec)
```

:::tip
StarRocksのドキュメントの一部のSQLクエリはセミコロンではなく、 `\G`で終わります。 `\G`はmysql CLIにクエリ結果を垂直にレンダリングさせます。

多くのSQLクライアントは垂直フォーマット出力を解釈しないため、mysql CLIを使用していない場合は、`\G`を`;`に置き換える必要があります。
:::

## StarRocksでクエリを実行する

### ピックアップの日時形式を検証する

```sql
SELECT lpep_pickup_datetime FROM greentaxis LIMIT 10;
```

```plaintext
+----------------------+
| lpep_pickup_datetime |
+----------------------+
| 2023-05-01 00:52:10  |
| 2023-05-01 00:29:49  |
| 2023-05-01 00:25:19  |
| 2023-05-01 00:07:06  |
| 2023-05-01 00:43:31  |
| 2023-05-01 00:51:54  |
| 2023-05-01 00:27:46  |
| 2023-05-01 00:27:14  |
```
| 2023-05-01 00:24:14  |
| 2023-05-01 00:46:55  |
+----------------------+
10 rows in set (0.07 sec)
```

#### 忙しい時間を見つける

このクエリは、日中のトリップを集計し、その日の一番忙しい時間が18:00であることを示しています。

```sql
SELECT COUNT(*) AS trips,
       hour(lpep_pickup_datetime) AS hour_of_day
FROM greentaxis
GROUP BY hour_of_day
ORDER BY trips DESC;
```

```plaintext
+-------+-------------+
| trips | hour_of_day |
+-------+-------------+
|  5381 |          18 |
|  5253 |          17 |
|  5091 |          16 |
|  4736 |          15 |
|  4393 |          14 |
|  4275 |          19 |
|  3893 |          12 |
|  3816 |          11 |
|  3685 |          13 |
|  3616 |           9 |
|  3530 |          10 |
|  3361 |          20 |
|  3315 |           8 |
|  2917 |          21 |
|  2680 |           7 |
|  2322 |          22 |
|  1735 |          23 |
|  1202 |           6 |
|  1189 |           0 |
|   806 |           1 |
|   606 |           2 |
|   513 |           3 |
|   451 |           5 |
|   408 |           4 |
+-------+-------------+
24 rows in set (0.08 sec)
```

---

## サマリ

このチュートリアルでは、StarRocksの外部カタログの使用方法を紹介しました。これにより、Iceberg RESTカタログを使用してデータをそのままクエリできることがわかります。Hive、Hudi、Delta Lake、JDBCカタログを使用した多くの他の統合も利用できます。

このチュートリアルでは、次のことを行いました。

- DockerでStarRocksとIceberg/PySpark/MinIO環境をデプロイしました
- StarRocks外部カタログを構成してIcebergカタログへのアクセスを提供しました
- ニューヨーク市が提供するタクシーデータをIcebergデータレイクにロードしました
- StarRocksでデータをデータレイクからコピーすることなくSQLでクエリしました

## もっと情報

[StarRocksカタログ](../data_source/catalog/catalog_overview.md)

[Apache Icebergドキュメント](https://iceberg.apache.org/docs/latest/) および [クイックスタート（PySparkを含む）](https://iceberg.apache.org/spark-quickstart/)

[グリーンタクシートリップレコード](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page) データセットは、ニューヨーク市が提供し、これらの[利用条件](https://www.nyc.gov/home/terms-of-use.page) と[プライバシーポリシー](https://www.nyc.gov/home/privacy-policy.page) に従います。