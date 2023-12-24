---
description: Data Lakehouse with Apache Iceberg
displayed_sidebar: English
sidebar_position: 3
toc_max_heading_level: 2
---
import DataLakeIntro from '../assets/commonMarkdown/datalakeIntro.md'
import Clients from '../assets/quick-start/_clientsCompose.mdx'

# 使用 Apache Iceberg 的数据湖仓库

## 概述

- 使用 Docker compose 部署对象存储、Apache Spark、Iceberg 目录和 StarRocks
- 将 2023 年 5 月的纽约市绿色出租车数据加载到 Iceberg 数据湖中
- 配置 StarRocks 访问 Iceberg 目录
- 使用 StarRocks 查询数据所在的位置

<DataLakeIntro />

## 先决条件

### Docker

- [Docker](https://docs.docker.com/engine/install/)
- 为 Docker 分配 5 GB RAM
- 为 Docker 分配 20 GB 可用磁盘空间

### SQL 客户端

您可以使用 Docker 环境中提供的 SQL 客户端，也可以使用系统上的客户端。许多兼容 MySQL 的客户端都可以使用，本指南涵盖了 DBeaver 和 MySQL WorkBench 的配置。

### curl

`curl` 用于下载数据集。通过在操作系统提示符下运行 `curl` 或 `curl.exe` 来检查是否已安装。如果尚未安装 curl，请[在此处获取 curl](https://curl.se/dlwiz/?type=bin)。

---

## StarRocks 术语

### FE
前端节点负责元数据管理、客户端连接管理、查询规划和查询调度。每个 FE 在其内存中存储并维护完整的元数据副本，从而保证了各个 FE 之间的无差别服务。

### BE
后端节点负责数据存储和在无共享部署中执行查询计划。在使用外部目录（如本指南中使用的 Iceberg 目录）时，只有本地数据存储在 BE 节点上。

---

## 环境

本指南中使用了六个容器（服务），所有容器（服务）都使用 Docker compose 进行部署。这些服务及其职责如下：

| 服务             | 责任                                                    |
|---------------------|---------------------------------------------------------------------|
| **`starrocks-fe`**  | 元数据管理、客户端连接、查询规划和调度 |
| **`starrocks-be`**  | 运行查询计划                                                 |
| **`rest`** | 提供 Iceberg 目录（元数据服务）                             |
| **`spark-iceberg`** | 运行 PySpark 的 Apache Spark 环境                       |
| **`mc`**            | MinIO 配置（MinIO 命令行客户端）                     |
| **`minio`**         | MinIO 对象存储                                                |

## 下载 Docker 配置和 NYC Green Taxi 数据

为了提供包含三个必要容器的环境，StarRocks 提供了一个 Docker compose 文件。 使用 curl 下载 compose 文件和数据集。

Docker compose 文件：
```bash
mkdir iceberg
cd iceberg
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/iceberg/docker-compose.yml
```

数据集：
```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/iceberg/datasets/green_tripdata_2023-05.parquet
```

## 在 Docker 中启动环境

:::提示
请从包含 `docker-compose.yml` 文件的目录中运行此命令以及任何其他 `docker compose` 命令。
:::

```bash
docker compose up -d
```

```plaintext
[+] Building 0.0s (0/0)                     docker:desktop-linux
[+] Running 6/6
 ✔ 容器 iceberg-rest   已启动                         0.0s
 ✔ 容器 minio          已启动                         0.0s
 ✔ 容器 starrocks-fe   已启动                         0.0s
 ✔ 容器 mc             已启动                         0.0s
 ✔ 容器 spark-iceberg  已启动                         0.0s
 ✔ 容器 starrocks-be   已启动
```

## 检查环境的状态

检查服务的进度。FE 和 BE 的状态变为 `healthy` 大约需要 30 秒。

运行 `docker compose ps` 直到 FE 和 BE 显示状态 `healthy`。其余服务没有健康检查配置，但您将与它们进行交互，并且会知道它们是否正常工作：

:::提示
如果已安装 `jq` 并且希望从 `docker compose ps` 获取较短的列表，请尝试：

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

有几种与 Iceberg 交互的方法，本指南使用 PySpark。如果您对 PySpark 不熟悉，可以从“更多信息”部分链接的文档中了解，但下面提供了您需要运行的每个命令。

### 绿色出租车数据集

将数据复制到 `spark-iceberg` 容器。此命令将数据集文件复制到 `spark-iceberg` 服务中的 `/opt/spark/` 目录：

```bash
docker compose \
cp green_tripdata_2023-05.parquet spark-iceberg:/opt/spark/
```

### 启动 PySpark

此命令将连接到 `spark-iceberg` 服务并运行 `pyspark` 命令：

```bash
docker compose exec -it spark-iceberg pyspark
```

```py
欢迎使用
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   版本 3.5.0
      /_/

使用 Python 版本 3.9.18 (main, Nov  1 2023 11:04:44)
Spark 上下文 Web UI 可在 http://6ad5cb0e6335:4041 找到
Spark 上下文可用作 'sc'（主节点 = local[*]，应用程序 ID = local-1701967093057）。
SparkSession 可用作 'spark'。
>>>
```

### 将数据集读入数据框

数据框是 Spark SQL 的一部分，提供了类似于数据库表或电子表格的数据结构。

纽约市出租车和豪华轿车委员会以 Parquet 格式提供了绿色出租车数据。从 `/opt/spark` 目录加载文件，并通过选择数据的前几行的前几列来检查数据。这些命令应在 `pyspark` 会话中运行。命令：

- 从磁盘读取数据集文件到名为 `df` 的数据框
- 显示 Parquet 文件的模式

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

检查数据的前几行的前几列：

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

### 写入表

在此步骤中创建的表将位于接下来的 StarRocks 目录中。

- 目录： `demo`
- 数据库： `nyc`
- 表： `greentaxis`

```python
df.writeTo("demo.nyc.greentaxis").create()
```

## 配置 StarRocks 访问 Iceberg 目录

您现在可以退出 PySpark 会话并连接到 StarRocks，或者您可以打开一个新的终端来运行 SQL 命令。如果您选择打开新的终端，请在继续之前将您的目录切换到包含 `docker-compose.yml` 文件的 `quickstart` 目录。

### 使用 SQL 客户端连接 StarRocks

#### SQL 客户端

<Clients />

---

您现在可以退出 PySpark 会话并连接到 StarRocks。

:::提示

请从包含 `docker-compose.yml` 文件的目录运行此命令。

如果您使用的是 mysql CLI 之外的客户端，请立即打开它。
:::


```bash
docker compose exec starrocks-fe \
  mysql -P 9030 -h 127.0.0.1 -u root --prompt="StarRocks > "
```

```plaintext
StarRocks >
```

### 创建外部目录

外部目录是允许 StarRocks 在 Iceberg 数据上操作的配置，就像它在 StarRocks 数据库和表中一样。在命令之后将详细介绍各个配置属性。

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

#### 配置属性

|    属性                      |     描述                                                                               |
|:---------------------------------|:----------------------------------------------------------------------------------------------|
|`type`                            | 在此示例中，类型为 `iceberg`。其他选项包括 Hive、Hudi、Delta Lake 和 JDBC。|
|`iceberg.catalog.type`            | 在此示例中使用 `rest` 。Tabular 提供了使用的 Docker 镜像，Tabular 使用 REST。|
|`iceberg.catalog.uri`             | REST 服务器终端。|
|`iceberg.catalog.warehouse`       | Iceberg 目录的标识符。在此示例中，compose 文件中指定的仓库名称为 `warehouse`。 |
|`aws.s3.access_key`               | MinIO 密钥。在此示例中，密钥和密码在 compose 文件中设置为 `admin` |
|`aws.s3.secret_key`               | 和 `password`. |
|`aws.s3.endpoint`                 | MinIO 终端。 |
|`aws.s3.enable_path_style_access` | 在使用 MinIO 对象存储时需要。MinIO 期望此格式 `http://host:port/<bucket_name>/<key_name>` |
|`client.factory`                  | 通过将此属性设置为使用 `iceberg.IcebergAwsClientFactory`，`aws.s3.access_key` 和 `aws.s3.secret_key` 参数用于身份验证。 |

```sql
SHOW CATALOGS;
```

```plaintext
+-----------------+----------+------------------------------------------------------------------+
| Catalog         | Type     | Comment                                                          |
+-----------------+----------+------------------------------------------------------------------+
| default_catalog | Internal | 内部目录包含此集群的自管理表。 |
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

:::提示
您在 PySpark 会话中创建的数据库现在已经可在 StarRocks 中看到。当您添加了 `iceberg` 目录后，`nyc` 数据库就会在 StarRocks 中可见。
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
正在读取表和列名称的完成表信息
您可以关闭此功能以获得更快的启动速度 -A

数据库已更改
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

:::提示
将 StarRocks 使用的模式与之前 PySpark 会话中的 `df.printSchema()` 输出进行比较。Spark 的 `timestamp_ntz` 数据类型在 StarRocks 中表示为 `DATETIME` 等。
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

:::提示
StarRocks 文档中的一些 SQL 查询以 `\G` 结尾，而不是分号。`\G` 会导致 mysql CLI 垂直呈现查询结果。

许多 SQL 客户端不解释垂直格式化输出，因此如果您不使用 mysql CLI，则应将 `\G` 替换为 `;`。
:::

## 使用 StarRocks 进行查询

### 验证取件日期时间格式

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

#### 查找繁忙时间

此查询按小时聚合行程，并显示一天中最繁忙的小时是 18:00。

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

## 总结

本教程向您展示了如何使用 StarRocks 外部目录，以及如何使用 Iceberg REST 目录查询数据所在的位置。还有许多其他集成可用，包括 Hive、Hudi、Delta Lake 和 JDBC 目录。

在本教程中，您：

- 在 Docker 中部署了 StarRocks 和 Iceberg/PySpark/MinIO 环境
- 配置了 StarRocks 外部目录，以便访问 Iceberg 目录
- 将纽约市提供的出租车数据加载到 Iceberg 数据湖中
- 在 StarRocks 中使用 SQL 查询数据，而无需从数据湖复制数据

## 更多信息
[StarRocks 目录](../data_source/catalog/catalog_overview.md)

[Apache Iceberg 文档](https://iceberg.apache.org/docs/latest/) 和 [快速入门（包括 PySpark）](https://iceberg.apache.org/spark-quickstart/)

[纽约市绿色出租车行程记录](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page) 数据集由纽约市提供，受到以下[使用条款](https://www.nyc.gov/home/terms-of-use.page)和[隐私政策](https://www.nyc.gov/home/privacy-policy.page)的约束。