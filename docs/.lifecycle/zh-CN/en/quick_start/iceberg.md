---
displayed_sidebar: "英文"
sidebar_position: 3
description: 使用 Apache Iceberg 的数据湖
toc_max_heading_level: 2
---

# 使用 Apache Iceberg 的数据湖
import Clients from '../assets/quick-start/_clientsCompose.mdx'

## 概览

- 使用 Docker compose 部署对象存储、Apache Spark、Iceberg 目录和 StarRocks
- 将 2023 年 5 月的纽约绿色出租车数据加载到 Iceberg 数据湖中
- 配置 StarRocks 访问 Iceberg 目录
- 使用 StarRocks 查询数据所在地

## 先决条件

### Docker

- [Docker](https://docs.docker.com/engine/install/)
- 将 5 GB RAM 分配给 Docker
- 将 20 GB 空闲磁盘空间分配给 Docker

### SQL 客户端

可以使用 Docker 环境中提供的 SQL 客户端，或者使用系统上的客户端。许多兼容 MySQL 的客户端都可以工作，本指南涵盖了 DBeaver 和 MySQL WorkBench 的配置。

### curl

`curl` 用于下载数据集。通过在操作系统提示符处运行 `curl` 或 `curl.exe` 来检查是否已安装 curl。如果尚未安装 curl，请 [在此处获取 curl](https://curl.se/dlwiz/?type=bin)。

---

## StarRocks 术语

### FE
前端节点负责元数据管理、客户端连接管理、查询规划和查询调度。每个 FE 在内存中存储和维护元数据的完整副本，从而保证了 FEs 之间的服务无差别。

### BE
后端节点负责在共享无事务部署中存储数据并执行查询计划。在使用外部目录（例如本指南中使用的 Iceberg 目录）时，只有本地数据存储在 BE 节点上。

---

## 环境

本指南中使用了六个容器（服务），并且都是使用 Docker compose 部署的。这些服务及其职责如下：

| 服务             | 职责                    |
|-------------------|--------------------------|
| **`starrocks-fe`**  | 元数据管理、客户端连接、查询规划和调度 |
| **`starrocks-be`**  | 运行查询计划               |
| **`rest`** | 提供 Iceberg 目录（元数据服务） |
| **`spark-iceberg`** | 用于运行 PySpark 的 Apache Spark 环境   |
| **`mc`**            | MinIO 配置（MinIO 命令行客户端）  |
| **`minio`**         | MinIO 对象存储             |

## 下载 Docker 配置文件和纽约绿色出租车数据

为了提供包含三个必要容器的环境，StarRocks 提供了一个 Docker compose 文件。使用 curl 下载 compose 文件和数据集。

Docker compose 文件：
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

:::提示
从包含 `docker-compose.yml` 文件的目录中运行此命令及其他 `docker compose` 命令。
:::

```bash
docker compose up -d
```

```plaintext
[+] Building 0.0s (0/0)                     docker:desktop-linux
[+] Running 6/6
 ✔ 容器 iceberg-rest   已启动       0.0s
 ✔ 容器 minio          已启动       0.0s
 ✔ 容器 starrocks-fe   已启动       0.0s
 ✔ 容器 mc             已启动       0.0s
 ✔ 容器 spark-iceberg  已启动
 ✔ 容器 starrocks-be   已启动
```

## 检查环境状态

检查服务的进展。FE 和 BE 成为健康状态应该大约需要 30 秒。

运行 `docker compose ps` 直到 FE 和 BE 显示状态为 `healthy`。其余服务没有健康检查配置，但您将与它们交互，并了解其是否正常工作：


:::提示
如果您已安装 `jq` 并且更喜欢从 `docker compose ps` 得到较短的列表，请尝试：

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

有几种方式可以与 Iceberg 交互，本指南使用 PySpark。如果您不熟悉 PySpark，可以从“更多信息”部分链接的文档中了解，但下面提供了您需要运行的每个命令。

### 绿色出租车数据集

将数据复制到 spark-iceberg 容器。此命令将数据集文件复制到 `spark-iceberg` 服务中的 `/opt/spark/` 目录中：

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
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   版本 3.5.0
      /_/

使用 Python 版本 3.9.18 (main, Nov  1 2023 11:04:44)
Spark 上下文 Web UI 可在 http://6ad5cb0e6335:4041 找到
Spark 上下文可用作 'sc'（master = local[*]，应用程序 ID = local-1701967093057）。
SparkSession 可用作 'spark'。
>>>
```

### 将数据集读入 dataframe

Dataframe 是 Spark SQL 的一部分，提供了类似数据库表或电子表格的数据结构。

纽约绿色出租车数据是由纽约出租车和豪华轿车委员会以 Parquet 格式提供。从磁盘加载文件到名为 `df` 的 dataframe，并通过选择数据的前几行的前几列来检查数据。这些命令应该在 `pyspark` 会话中运行。命令：

- 从磁盘读取数据集文件到名为 `df` 的 dataframe
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
检查前几行数据的前几列：

```python
df.select(df.columns[:7]).show(3)
```
```plaintext
+--------+--------------------+---------------------+------------------+----------+------------+------------+
```markdown
|厂商ID|lpep_pickup_datetime|lpep_dropoff_datetime|store_and_fwd_flag|RatecodeID|PULocationID|DOLocationID|
+-----+--------------------+---------------------+------------------+----------+------------+------------+
|   2 |2023-05-01 00:52:10 |2023-05-01 01:05:26  |N                 |1         |244         |213         |
|   2 |2023-05-01 00:29:49 |2023-05-01 00:50:11  |N                 |1         |33          |100         |
|   2 |2023-05-01 00:25:19 |2023-05-01 00:32:12  |N                 |1         |244         |244         |
+-----+--------------------+---------------------+------------------+----------+------------+------------+
只显示 3 行记录
```

```markdown

### 写入表

此步骤中创建的表将在接下来的步骤中在 StarRocks 中提供。

- 目录：`演示`
- 数据库：`纽约`
- 表：`绿色出租车`

```python

df.writeTo("demo.nyc.greentaxis").create()

```



## 配置 StarRocks 访问 Iceberg 目录

现在可以退出 PySpark，或者打开新的终端运行 SQL 命令。如果您打开了新的终端，请在继续之前切换到包含 `docker-compose.yml` 文件的 `quickstart` 目录。

### 使用 SQL 客户端连接到 StarRocks

#### SQL 客户端

<客户端 />

---



您现在可以退出 PySpark 会话并连接到 StarRocks。

::: 提示


从包含 `docker-compose.yml` 文件的目录运行此命令。

如果您使用的客户端不是 mysql CLI，则现在打开它。

:::


```bash
docker compose exec starrocks-fe \

  mysql -P 9030 -h 127.0.0.1 -u root --prompt="StarRocks > "
```

```plaintext
StarRocks >
```

### 创建外部目录

外部目录是允许 StarRocks 对 Iceberg 数据进行操作的配置。个别配置属性将在命令之后详细说明。

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


#### 属性

|    属性                            |     描述                                                                                  |

|:-----------------------------------|:-----------------------------------------------------------------------------------------|
|`type`                             | 在此示例中，类型为 `iceberg`。其他选项包括 Hive、Hudi、Delta Lake 和 JDBC。              |
|`iceberg.catalog.type`             | 在此示例中，使用了 `rest`。Tabular 提供了 Docker 图像，并且 Tabular 使用了 REST。         |
|`iceberg.catalog.uri`              | REST 服务器端点。|
|`iceberg.catalog.warehouse`        | Iceberg 目录的标识。在此示例中，compose 文件中指定的仓库名称是 `warehouse`。 |
|`aws.s3.access_key`                | MinIO 密钥。在此示例中，密钥和密码在 compose 文件中设置为 `admin` |
|`aws.s3.secret_key`                | 和 `password` 。|
|`aws.s3.endpoint`                  | MinIO 终点。|
|`aws.s3.enable_path_style_access`  | 使用 MinIO 对象存储时需要此属性。MinIO 期望此格式 `http://host:port/<bucket_name>/<key_name>` |

|`client.factory`                   | 通过将此属性设置为使用 `iceberg.IcebergAwsClientFactory`，`aws.s3.access_key` 和 `aws.s3.secret_key` 参数用于身份验证。|

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

::: 提示
您在 PySpark 中创建的数据库。当您添加了 CATALOG `iceberg` 后，`nyc` 数据库在 StarRocks 中可见。
:::

```plaintext
+----------+

| Database |
+----------+
| nyc      |

+----------+
1 行记录 (0.07 秒)
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
1 行记录 (0.05 秒)
```

```sql
DESCRIBE greentaxis;
```

:::tip
将 StarRocks 使用的架构与之前 PySpark 会话中 `df.printSchema()` 的输出进行比较。Spark 的 `timestamp_ntz` 数据类型表示为 StarRocks 的 `DATETIME` 等。
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
20 行记录 (0.04 秒)
```

```
| 2023-05-01 00:24:14  |
| 2023-05-01 00:46:55  |
+----------------------+
10 rows in set (0.07 sec)
```

#### 查找忙碌时间

此查询汇总了一天中每个小时的行程，并显示一天中最忙碌的小时是18:00。

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

本教程介绍了如何使用StarRocks外部目录来展示您可以使用Iceberg REST目录在其所在的位置查询数据。还可以使用Hive、Hudi、Delta Lake和JDBC目录进行许多其他集成。

在本教程中，您：

- 在Docker中部署了StarRocks和Iceberg/PySpark/MinIO环境
- 配置了StarRocks外部目录以提供对Iceberg目录的访问
- 将纽约市提供的出租车数据加载到Iceberg数据湖中
- 在StarRocks中使用SQL查询数据，而无需将数据从数据湖中复制

## 更多信息

[StarRocks目录](../data_source/catalog/catalog_overview.md)

[Apache Iceberg文档](https://iceberg.apache.org/docs/latest/) 和 [快速入门（包括PySpark）](https://iceberg.apache.org/spark-quickstart/)

纽约市的[绿色出租车行程记录](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)数据集由纽约市提供，受到这些[使用条款](https://www.nyc.gov/home/terms-of-use.page)和[隐私政策](https://www.nyc.gov/home/privacy-policy.page)的约束。