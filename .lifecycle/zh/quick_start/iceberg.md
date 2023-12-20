---
description: Data Lakehouse with Apache Iceberg
displayed_sidebar: English
sidebar_position: 3
toc_max_heading_level: 2
---

从 '../assets/commonMarkdown/datalakeIntro.md' 导入 DataLakeIntro，从 '../assets/quick-start/_clientsCompose.mdx' 导入 Clients

# 使用 Apache Iceberg 构建数据湖仓库

## 概览

- 使用 Docker Compose 部署对象存储、Apache Spark、Iceberg 目录和 StarRocks
- 将 2023 年 5 月的纽约市绿色出租车数据加载到 Iceberg 数据湖中
- 配置 StarRocks 以访问 Iceberg 目录
- 在数据所在地使用 StarRocks 进行数据查询

<DataLakeIntro />


## 先决条件

### Docker

- [Docker](https://docs.docker.com/engine/install/) 为 Docker 分配 5GB RAM
- 为 Docker 分配 20GB 的空闲磁盘空间
- SQL 客户端

### 您可以使用 Docker 环境中提供的 SQL 客户端，或者使用您系统上的其他 SQL 客户端。许多兼容 MySQL 的客户端都可以使用，本指南将介绍如何配置 DBeaver 和 MySQL Workbench。

curl

### curl 用于下载数据集。通过在您的操作系统提示符下运行 curl 或 curl.exe 来检查是否已安装 curl。如果没有安装 curl，请在这里下载 curl。

`curl` 用于下载数据集。通过在操作系统提示符下运行 `curl` 或 `curl.exe` 来检查是否已安装。如果未安装 curl，请[在此处获取curl](https://curl.se/dlwiz/?type=bin)。


## FE（Frontend）

### 前端节点负责元数据管理、客户端连接管理、查询规划和查询调度。每个前端节点在其内存中存储并维护一份元数据的完整副本，确保了前端节点间的服务一致性。
BE（Backend）

### 后端节点负责数据存储和在无共享部署中执行查询计划。当使用外部目录（如本指南中使用的 Iceberg 目录）时，只有本地数据存储在后端节点上。
环境配置


## 本指南使用了六个容器（服务），全部通过 Docker Compose 部署。服务及其职责如下：

| 服务         | 职责                                       |

|服务|责任|
|---|---|
|starrocks-fe|元数据管理、客户端连接、查询计划和调度|
|starrocks-be|运行查询计划|
|rest|提供 Iceberg 目录（元数据服务）|
|spark-iceberg|运行 PySpark 的 Apache Spark 环境|
|mc|MinIO 配置（MinIO 命令行客户端）|
|minio|MinIO 对象存储|

## 下载 Docker 配置和纽约市绿色出租车数据

为了提供包含三个必要容器的环境，StarRocks 提供了一个 Docker Compose 文件。使用 curl 下载 Compose 文件和数据集。

Docker Compose 文件：
```bash
mkdir iceberg
cd iceberg
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/iceberg/docker-compose.yml
```

以及数据集：
```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/iceberg/datasets/green_tripdata_2023-05.parquet
```

## 在 Docker 中启动环境

:::提示 请从包含 docker-compose.yml 文件的目录运行此命令，以及任何其他 Docker Compose 命令。:::

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

## 检查环境状态

检查服务的启动进度。前端和后端节点大约需要 30 秒时间才能变为健康状态。

运行 docker compose ps 直到前端和后端节点显示健康状态。其他服务没有健康检查配置，但您会与它们互动，因此能够知道它们是否工作正常：

:::提示 如果您安装了 jq 并且希望从 docker compose ps 获取更简短的列表，请尝试：

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


## PySpark

有多种方法可以与 Iceberg 进行交互，本指南使用 PySpark。如果您不熟悉 PySpark，可以从“更多信息”部分找到相关文档链接，但是您需要运行的每个命令都在下面提供了。

### 绿色出租车数据集

将数据复制到 spark-iceberg 容器中。此命令会将数据集文件复制到 spark-iceberg 服务的 /opt/spark/ 目录中：

```bash
docker compose \
cp green_tripdata_2023-05.parquet spark-iceberg:/opt/spark/
```

### 启动 PySpark

此命令将连接到 spark-iceberg 服务并运行 pyspark 命令：

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

### 将数据集读入 DataFrame

DataFrame 是 Spark SQL 的一部分，它提供了类似于数据库表或电子表格的数据结构。

绿色出租车数据由纽约市出租车和豪华轿车委员会以 Parquet 格式提供。从 /opt/spark 目录加载文件，并通过选择前三行数据的前几列来检查前几条记录。这些命令应在 pyspark 会话中运行。命令如下：

- 从磁盘将数据集文件读入名为 df 的 DataFrame
- 显示 Parquet 文件的结构

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
查看前几行（三行）数据的前几列（七列）：

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
### 写入表格

在此步骤中创建的表格将在下一步中在 StarRocks 中提供的目录中。

- 目录：demo
- 数据库：nyc
- 表格：greentaxis

```python
df.writeTo("demo.nyc.greentaxis").create()
```

## 配置 StarRocks 以访问 Iceberg 目录

您现在可以退出 PySpark，也可以打开一个新的终端来运行 SQL 命令。如果您打开了一个新的终端，请在继续之前将目录切换到包含 docker-compose.yml 文件的 quickstart 目录。

### 使用 SQL 客户端连接到 StarRocks

#### SQL 客户端

<Clients />



您现在可以退出 PySpark 会话并连接到 StarRocks。

:::提示 请从包含 docker-compose.yml 文件的目录运行此命令。

如果您使用的不是 mysql CLI 客户端，请现在打开它。:::

创建外部目录

```bash
docker compose exec starrocks-fe \
  mysql -P 9030 -h 127.0.0.1 -u root --prompt="StarRocks > "
```

```plaintext
StarRocks >
```

### 外部目录是配置，它允许 StarRocks 操作 Iceberg 数据，就好像数据位于 StarRocks 的数据库和表中一样。命令执行后，将详细说明每个配置属性。

属性

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

#### | 属性                          | 描述                                       |

|属性|描述|
|---|---|
|type|在此示例中，类型是iceberg。其他选项包括 Hive、Hudi、Delta Lake 和 JDBC。|
|iceberg.catalog.type|在此示例中使用了休息。 Tabular 提供了所使用的 Docker 映像，Tabular 使用 REST。|
|iceberg.catalog.uri|REST 服务器端点。|
|iceberg.catalog.warehouse|Iceberg 目录的标识符。在这种情况下，撰写文件中指定的仓库名称是仓库。|
|aws.s3.access_key|MinIO 密钥。在这种情况下，密钥和密码在撰写文件中设置为 admin|
|aws.s3.secret_key|和密码。|
|aws.s3.endpoint|MinIO 端点。|
|aws.s3.enable_path_style_access|使用 MinIO 进行对象存储时，这是必需的。 MinIO 需要这种格式 http://host:port/<bucket_name>/<key_name>|
|client.factory|通过设置此属性以使用iceberg.IcebergAwsClientFactory，aws.s3.access_key 和 aws.s3.secret_key 参数用于身份验证。|

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
:::提示 您在 PySpark 会话中创建的数据库现在在 StarRocks 中可见，因为您添加了 CATALOG iceberg。:::

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

:::提示 比较 StarRocks 使用的模式与早期 PySpark 会话中 df.printSchema() 输出的模式。Spark 中的 timestamp_ntz 数据类型在 StarRocks 中表示为 DATETIME 等。:::

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

:::提示 StarRocks 文档中的一些 SQL 查询以 \G 而不是分号结尾。使用 \G 会使 mysql CLI 垂直渲染查询结果。

许多 SQL 客户端不支持垂直格式输出，因此如果您不使用 mysql CLI，应该用分号替换 \G。:::

## 使用 StarRocks 进行查询

### 验证乘车时间格式

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

#### 找出高峰时段

此查询按小时聚合行程，并显示一天中最繁忙的时段是 18:00。

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


## 总结

本教程向您展示了如何使用 StarRocks 的外部目录，使您能够使用 Iceberg REST 目录在数据所在地进行查询。还可以使用 Hive、Hudi、Delta Lake 和 JDBC 目录进行许多其他集成。

在本教程中，您已完成：

- 在 Docker 中部署了 StarRocks 和 Iceberg/PySpark/MinIO 环境
- 配置了 StarRocks 外部目录，以提供对 Iceberg 目录的访问
- 将纽约市提供的出租车数据加载到 Iceberg 数据湖中
- 在 StarRocks 中使用 SQL 查询数据，无需将数据从数据湖中复制出来

## 更多信息

[StarRocks 目录](../data_source/catalog/catalog_overview.md)

[Apache Iceberg documentation](https://iceberg.apache.org/docs/latest/) 和 [Quickstart (includes PySpark)](https://iceberg.apache.org/spark-quickstart)

纽约市提供的[绿色出租车行程记录](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)数据集，适用于这些[使用条款](https://www.nyc.gov/home/terms-of-use.page)和[隐私政策](https://www.nyc.gov/home/privacy-policy.page)。
