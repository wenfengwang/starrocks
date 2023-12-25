---
description: Data Lakehouse with Apache Iceberg
displayed_sidebar: English
sidebar_position: 3
toc_max_heading_level: 2
---
import DataLakeIntro from '../assets/commonMarkdown/datalakeIntro.md'
import Clients from '../assets/quick-start/_clientsCompose.mdx'

# Apache Iceberg を使用したデータレイクハウス

## 概要

- Docker composeを使用してObject Storage、Apache Spark、Icebergカタログ、StarRocksをデプロイ
- 2023年5月のニューヨーク市グリーンタクシーデータをIcebergデータレイクにロード
- StarRocksがIcebergカタログにアクセスするように設定
- データが存在する場所でStarRocksを使用してデータをクエリ

<DataLakeIntro />

## 前提条件

### Docker

- [Docker](https://docs.docker.com/engine/install/)
- Dockerに割り当てられたRAM 5GB
- Dockerに割り当てられた空きディスクスペース 20GB

### SQLクライアント

Docker環境で提供されるSQLクライアントを使用することも、システム上のものを使用することもできます。多くのMySQL互換クライアントが動作しますが、このガイドではDBeaverとMySQL Workbenchの設定をカバーしています。

### curl

`curl`はデータセットをダウンロードするために使用されます。OSプロンプトで`curl`または`curl.exe`を実行してインストールされているかどうかを確認してください。curlがインストールされていない場合は、[ここからcurlを取得してください](https://curl.se/dlwiz/?type=bin)。

---

## StarRocks用語

### FE
フロントエンドノードはメタデータ管理、クライアント接続管理、クエリプランニング、クエリスケジューリングを担当します。各FEはメタデータの完全なコピーをメモリに保存し維持し、FE間でのサービスの均一性を保証します。

### BE
バックエンドノードはデータストレージとシェアードナッシングデプロイメントでのクエリプランの実行を担当します。外部カタログ（このガイドで使用されるIcebergカタログなど）を使用する場合、BEノードにはローカルデータのみが格納されます。

---

## 環境

このガイドで使用されるコンテナ（サービス）は6つあり、すべてDocker composeでデプロイされます。サービスとその責任は以下の通りです：

| サービス             | 責任                                                         |
|---------------------|--------------------------------------------------------------|
| **`starrocks-fe`**  | メタデータ管理、クライアント接続、クエリプラン、スケジューリング |
| **`starrocks-be`**  | クエリプランの実行                                            |
| **`rest`**          | Icebergカタログ（メタデータサービス）の提供                  |
| **`spark-iceberg`** | PySparkを実行するApache Spark環境                            |
| **`mc`**            | MinIO設定（MinIOコマンドラインクライアント）                 |
| **`minio`**         | MinIOオブジェクトストレージ                                   |

## Docker構成とNYCグリーンタクシーデータのダウンロード

StarRocksは必要な3つのコンテナを提供するDocker composeファイルを提供しています。curlを使用してcomposeファイルとデータセットをダウンロードしてください。

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
このコマンド、およびその他の`docker compose`コマンドは、`docker-compose.yml`ファイルが含まれているディレクトリから実行してください。
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

サービスの進行状況を確認します。FEとBEが`healthy`のステータスを示すまで約30秒かかります。

`docker compose ps`を実行し、FEとBEが`healthy`のステータスを示すまで確認してください。他のサービスにはヘルスチェックの設定はありませんが、それらと対話することで機能しているかどうかを知ることができます。

:::tip
`jq`をインストールしていて、`docker compose ps`からより短いリストを好む場合は、次のコマンドを試してください：

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

Icebergを操作する方法はいくつかありますが、このガイドではPySparkを使用します。PySparkに慣れていない場合は、「詳細情報」セクションからドキュメントにリンクされていますが、実行する必要があるすべてのコマンドは以下に提供されています。

### グリーンタクシーデータセット

データを`spark-iceberg`コンテナにコピーします。このコマンドは、データセットファイルを`spark-iceberg`サービスの`/opt/spark/`ディレクトリにコピーします。

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

### データセットをデータフレームに読み込む

データフレームはSpark SQLの一部であり、データベーステーブルやスプレッドシートに似たデータ構造を提供します。

グリーンタクシーデータはNYC Taxi and Limousine CommissionからParquet形式で提供されています。`/opt/spark`ディレクトリからファイルをロードし、データの最初の数レコードを選択して、最初の3行のデータの最初のいくつかの列を調べます。これらのコマンドは`pyspark`セッションで実行する必要があります。コマンドは以下の通りです：

- データセットファイルをディスクからデータフレーム`df`に読み込む
- Parquetファイルのスキーマを表示する

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
データの最初の数行の最初の7列を調べます：

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
### テーブルへの書き込み

このステップで作成されるテーブルは、次のステップでStarRocksで利用可能になるカタログに含まれます。

- カタログ: `demo`
- データベース: `nyc`
- テーブル: `greentaxis`

```python
df.writeTo("demo.nyc.greentaxis").create()
```

## StarRocksでIcebergカタログにアクセスする設定

PySparkから退出するか、または新しいターミナルを開いてSQLコマンドを実行することができます。新しいターミナルを開く場合は、`docker-compose.yml`ファイルが含まれている`quickstart`ディレクトリに移動してから続行してください。

### SQLクライアントを使用してStarRocksに接続

#### SQLクライアント

<Clients />

---

これでPySparkセッションを終了し、StarRocksに接続することができます。

:::tip

このコマンドは`docker-compose.yml`ファイルが含まれるディレクトリから実行してください。

mysql CLI以外のクライアントを使用している場合は、それを今開いてください。
:::


```bash
docker compose exec starrocks-fe \
  mysql -P 9030 -h 127.0.0.1 -u root --prompt="StarRocks > "
```

```plaintext
StarRocks >
```

### 外部カタログの作成

外部カタログは、StarRocksがIcebergデータをStarRocksのデータベースやテーブルにあるかのように操作できるようにする設定です。個々の設定プロパティは、コマンドの後で詳しく説明されます。

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

#### プロパティ

|    プロパティ                    |     説明                                                                               |
|:---------------------------------|:----------------------------------------------------------------------------------------------|
|`type`                            | この例ではタイプは`iceberg`です。他のオプションにはHive、Hudi、Delta Lake、JDBCがあります。|
|`iceberg.catalog.type`            | この例では`rest`が使用されます。Tabularは使用されるDockerイメージを提供し、TabularはRESTを使用します。|
|`iceberg.catalog.uri`             | RESTサーバーエンドポイント。|
|`iceberg.catalog.warehouse`       | Icebergカタログの識別子です。このケースでは、composeファイルで指定されたウェアハウス名は`warehouse`です。|
|`aws.s3.access_key`               | MinIOキーです。このケースでは、キーとパスワードはcomposeファイルで`admin`と`password`に設定されています。|
|`aws.s3.secret_key`               | そして`password`です。|
|`aws.s3.endpoint`                 | MinIOエンドポイント。|
|`aws.s3.enable_path_style_access` | オブジェクトストレージにMinIOを使用する場合、これが必要です。MinIOはこの形式を期待しています: `http://host:port/<bucket_name>/<key_name>` |
|`client.factory`                  | このプロパティを`iceberg.IcebergAwsClientFactory`に設定することで、`aws.s3.access_key`と`aws.s3.secret_key`パラメータが認証に使用されます。|

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
SET CATALOG 'iceberg';
```

```sql
SHOW DATABASES;
```
:::tip
表示されるデータベースは、PySparkセッションで作成されたものです。
`iceberg`カタログを追加したとき、データベース`nyc`はStarRocksで表示可能になりました。
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
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

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

:::tip
StarRocksが使用するスキーマを、以前のPySparkセッションでの`df.printSchema()`の出力と比較してください。Sparkの`timestamp_ntz`データ型はStarRocksの`DATETIME`などとして表されます。
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
StarRocksのドキュメントにあるSQLクエリの一部は、セミコロンの代わりに`\G`で終わります。`\G`はmysql CLIにクエリ結果を縦に表示させるよう指示します。

多くのSQLクライアントは縦表示を解釈しないため、mysql CLIを使用していない場合は`\G`を`;`に置き換えるべきです。
:::

## StarRocksでのクエリ

### ピックアップ日時の形式を確認

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
| 2023-05-01 00:24:14  |
| 2023-05-01 00:46:55  |
+----------------------+
10 rows in set (0.07 sec)
```

#### 混雑時間帯の検出

このクエリは、1日の時間帯によるトリップ数を集計し、最も混雑する時間帯が18:00であることを示しています。

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

## まとめ

このチュートリアルでは、StarRocksの外部カタログを使用して、Iceberg RESTカタログを使用してデータがどこにあるかを問い合わせる方法を紹介しました。Hive、Hudi、Delta Lake、JDBCカタログを使用した他の多くの統合が利用可能です。

このチュートリアルでは、以下のことを行いました。

- DockerにStarRocksとIceberg/PySpark/MinIO環境をデプロイしました
- Icebergカタログへのアクセスを提供するためにStarRocksの外部カタログを設定しました
- ニューヨーク市が提供するタクシーデータをIcebergデータレイクにロードしました
- データレイクからデータをコピーせずにStarRocksでSQLを使用してデータをクエリしました

## 詳細情報

[StarRocksカタログ](../data_source/catalog/catalog_overview.md)

[Apache Iceberg のドキュメント](https://iceberg.apache.org/docs/latest/) および [クイックスタート (PySpark 含む)](https://iceberg.apache.org/spark-quickstart/)

[Green Taxi Trip Records](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page) データセットは、ニューヨーク市によってこれらの[利用規約](https://www.nyc.gov/home/terms-of-use.page)および[プライバシーポリシー](https://www.nyc.gov/home/privacy-policy.page)に基づいて提供されます。
