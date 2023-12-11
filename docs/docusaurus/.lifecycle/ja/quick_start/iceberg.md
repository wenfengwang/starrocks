---
displayed_sidebar: "Japanese"
sidebar_position: 3
description: Apache Iceberg を使用した Data Lakehouse
toc_max_heading_level: 2
---

# Apache Iceberg を使用した Data Lakehouse
import Clients from '../assets/quick-start/_clientsCompose.mdx'

## 概要

- Docker compose を使用して Object Storage、Apache Spark、Iceberg カタログ、StarRocks をデプロイします
- Iceberg データレイクに 2023 年 5 月のニューヨーク市の緑のタクシーデータをロードします
- StarRocks を Iceberg カタログにアクセスできるように構成します
- データが存在する場所で StarRocks を使用してデータをクエリします

## 前提条件

### Docker

- [Docker](https://docs.docker.com/engine/install/)
- Docker に割り当てられた RAM が 5 GB
- Docker に割り当てられた空きディスク容量が 20 GB

### SQL クライアント

Docker 環境で提供される SQL クライアントを使用するか、システム上の SQL クライアントを使用します。多くの MySQL 互換クライアントが動作します。本ガイドでは DBeaver および MySQL WorkBench の構成について説明します。

### curl

`curl` はデータセットをダウンロードするために使用されます。OS プロンプトで `curl` または `curl.exe` を実行してインストールされているかどうかを確認します。curl がインストールされていない場合は、[ここから curl を取得してください](https://curl.se/dlwiz/?type=bin)。

---

## StarRocks の用語

### FE
フロントエンドノードはメタデータ管理、クライアント接続管理、クエリプランニング、およびクエリスケジューリングに責任を持ちます。各 FE は、そのメモリにメタデータの完全なコピーを保存・維持するため、F E 間のサービスは無差別です。

### BE
バックエンドノードは共有なしのデプロイメントでのデータ保存およびクエリプランの実行に責任を持ちます。このガイドで使用されている Iceberg カタログのような外部カタログを使用する場合、ローカルデータのみが BE ノードに保存されます。

---

## 環境

このガイドでは 6 つのコンテナ（サービス）が使用され、すべてが Docker compose で展開されています。サービスおよびそれぞれの責任は次のとおりです:

| サービス               | 責任                                                    |
|----------------------|----------------------------------------------------------|
| **`starrocks-fe`**   | メタデータ管理、クライアント接続、クエリプランおよびスケジューリング |
| **`starrocks-be`**   | クエリプランの実行                                      |
| **`rest`**            | Iceberg カタログ (メタデータサービス) の提供                   |
| **`spark-iceberg`**  | Apache Spark 環境で PySpark を実行する                      |
| **`mc`**             | MinIO 構成 (MinIO コマンドラインクライアント)                  |
| **`minio`**          | MinIO オブジェクトストレージ                                  |

## Docker 構成と NYC グリーンタクシーデータをダウンロード

StarRocks は Docker compose ファイルを提供し、3 つの必要なコンテナを含む環境を提供するためにデータセットをダウンロードします。

Docker compose ファイル:
```bash
mkdir iceberg
cd iceberg
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/iceberg/docker-compose.yml
```

データセット:
```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/iceberg/datasets/green_tripdata_2023-05.parquet
```

## Docker 環境を起動する

:::tip
このコマンドおよび他の `docker compose` コマンドはすべて、`docker-compose.yml` ファイルを含むディレクトリから実行します。
:::

```bash
docker compose up -d
```

```plaintext
[+] Building 0.0s (0/0)                     docker:desktop-linux
[+] Running 6/6
 ✔ コンテナ iceberg-rest   Started                         0.0s
 ✔ コンテナ minio          Started                         0.0s
 ✔ コンテナ starrocks-fe   Started                         0.0s
 ✔ コンテナ mc             Started                         0.0s
 ✔ コンテナ spark-iceberg  Started                         0.0s
 ✔ コンテナ starrocks-be   Started
```

## 環境のステータスを確認する

サービスの進捗状況を確認します。FE および BE が健全になるまで約 30 秒かかるはずです。

FE および BE が `healthy` の状態を表示するまで `docker compose ps` を実行します。その他のサービスにはヘルスチェックの構成がありませんが、これらとやり取りしているため、その状況を把握します:

:::tip
`jq` がインストールされており、`docker compose ps` からの短いリストを希望する場合は次のコマンドを試してください:

```bash
docker compose ps --format json | jq '{Service: .Service, State: .State, Status: .Status}'
```

:::

```bash
docker compose ps
```

```bash
SERVICE         CREATED         STATUS                   PORTS
rest            4 分前         Up 4 分                   0.0.0.0:8181->8181/tcp
mc              4 分前         Up 4 分
minio           4 分前         Up 4 分                   0.0.0.0:9000-9001->9000-9001/tcp
spark-iceberg   4 分前         Up 4 分                   0.0.0.0:8080->8080/tcp, 0.0.0.0:8888->8888/tcp, 0.0.0.0:10000-10001->10000-10001/tcp
starrocks-be    4 分前         Up 4 分 (healthy)         0.0.0.0:8040->8040/tcp
starrocks-fe    4 分前         Up 4 分 (healthy)         0.0.0.0:8030->8030/tcp, 0.0.0.0:9020->9020/tcp, 0.0.0.0:9030->9030/tcp
```

---

## PySpark

Iceberg とやり取りするためにはいくつかの方法がありますが、このガイドでは PySpark を使用します。PySpark が馴染みがない場合は、「詳細情報」からリンクされたドキュメントをご覧ください。ただし、実行するすべてのコマンドが以下に提供されています。

### グリーンタクシーデータセット

データを spark-iceberg コンテナにコピーします。このコマンドは、データセットファイルを `spark-iceberg` サービスの `/opt/spark/` ディレクトリにコピーします:

```bash
docker compose cp green_tripdata_2023-05.parquet spark-iceberg:/opt/spark/
```

### PySpark を起動する

このコマンドは `spark-iceberg` サービスに接続し、`pyspark` コマンドを実行します:

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

### データセットをデータフレームに読み込む

データフレームは Spark SQL の一部であり、データベーステーブルやスプレッドシートに類似したデータ構造を提供します。

グリーンタクシーデータは NYC Taxi and Limousine Commission によって Parquet 形式で提供されます。`df` という名前のデータフレームにファイルを `/opt/spark` ディレクトリから読み込み、データの最初の数行と最初の数列のスキーマを表示します。これらのコマンドは `pyspark` セッションで実行する必要があります。コマンド:

- ディスクからデータセットファイルを `df` という名前のデータフレームに読み込む
- Parquet ファイルのスキーマを表示する

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
データの最初の数列の最初の数行を表示します:

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
表示している3行のみ
```
### テーブルに書き込む

この手順で作成されたテーブルは、次の手順でStarRocksで利用可能になるカタログになります。

- カタログ: `demo`
- データベース: `nyc`
- テーブル: `greentaxis`

```python
df.writeTo("demo.nyc.greentaxis").create()
```

## StarRocksへのアクセスを構成する

PySparkから抜けることもできますし、新しいターミナルを開いてSQLコマンドを実行することもできます。他のクライアントを使用している場合は、そのクライアントを開いてください。

```bash
docker compose exec starrocks-fe \
  mysql -P 9030 -h 127.0.0.1 -u root --prompt="StarRocks > "
```

```plaintext
StarRocks >
```

### 外部カタログを作成する

外部カタログは、StarRocksがStarRocksデータベースとテーブル内にあるかのようにIcebergデータ上で操作するための構成です。

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

|    Property                      |     Description                                                                               |
|:---------------------------------|:----------------------------------------------------------------------------------------------|
|`type`                            | In this example the type is `iceberg`. Other options include Hive, Hudi, Delta Lake, and JDBC.|
|`iceberg.catalog.type`            | In this example `rest` is used. Tabular provides the Docker image used and Tabular uses REST.|
|`iceberg.catalog.uri`             | The REST server endpoint.|
|`iceberg.catalog.warehouse`       | The identifier of the Iceberg catalog. In this case the warehouse name specified in the compose file is `warehouse`. |
|`aws.s3.access_key`               | The MinIO key. In this case the key and password are set in the compose file to `admin` |
|`aws.s3.secret_key`               | and `password`. |
|`aws.s3.endpoint`                 | The MinIO endpoint. |
|`aws.s3.enable_path_style_access` | When using MinIO for Object Storage this is required. MinIO expects this format `http://host:port/<bucket_name>/<key_name>` |
|`client.factory`                  | By setting this property to use `iceberg.IcebergAwsClientFactory` the `aws.s3.access_key` and `aws.s3.secret_key` parameters are used for authentication. |

```sql
SHOW CATALOGS;
```

```plaintext
+-----------------+----------+------------------------------------------------------------------+
| Catalog         | Type     | Comment                                                          |
+-----------------+----------+------------------------------------------------------------------+
| default_catalog | Internal | An internal catalog contains this cluster's self-managed tables. |
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
Database changed
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

## StarRocksでクエリを実行する

### ピックアップの日時の形式を確認する

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
```plaintext
| 2023-05-01 00:24:14  |
| 2023-05-01 00:46:55  |
+----------------------+
10 rows in set (0.07 sec)
```

#### 忙しい時間を見つける

このクエリは、一日の中の時刻ごとにトリップを集計し、一番忙しい時間が18:00であることを示しています。

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

## 要約

このチュートリアルでは、StarRocks外部カタログの使用とIceberg RESTカタログを使用してデータをクエリすることができることを示しました。Hive、Hudi、Delta Lake、およびJDBCカタログを使用した多くの他の統合も利用可能です。

このチュートリアルでは：

- StarRocksとIceberg/PySpark/MinIO環境をDockerに展開しました
- StarRocks外部カタログを構成してIcebergカタログへのアクセスを提供しました
- ニューヨーク市の提供するタクシーデータをIcebergデータレイクにロードしました
- データをデータレイクからコピーすることなくStarRocksでSQLを使用してデータをクエリしました

## さらに詳しい情報

[StarRocksカタログ](../data_source/catalog/catalog_overview.md)

[Apache Icebergドキュメント](https://iceberg.apache.org/docs/latest/)および[クイックスタート（PySparkを含む）](https://iceberg.apache.org/spark-quickstart/)

[グリーンタクシートリップレコード](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)データセットは、これらの[利用条件](https://www.nyc.gov/home/terms-of-use.page)および[プライバシーポリシー](https://www.nyc.gov/home/privacy-policy.page)に準拠してニューヨーク市によって提供されています。