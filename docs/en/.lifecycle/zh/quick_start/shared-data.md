---
description: Separate compute and storage
displayed_sidebar: English
sidebar_position: 2
---

# 存储与计算分离
import DDL from '../assets/quick-start/_DDL.mdx'
import Clients from '../assets/quick-start/_clientsCompose.mdx'
import SQL from '../assets/quick-start/_SQL.mdx'
import Curl from '../assets/quick-start/_curl.mdx'

在将存储与计算分离的系统中，数据存储在低成本可靠的远程存储系统中，例如 Amazon S3、Google Cloud Storage、Azure Blob Storage，以及其他兼容 S3 的存储如 MinIO。热数据被本地缓存，当缓存命中时，查询性能可与存储计算耦合架构相媲美。计算节点（CN）可以在几秒钟内根据需求增加或移除。这种架构降低了存储成本，确保了更好的资源隔离，并提供了弹性和可扩展性。

本教程包括以下内容：

- 在 Docker 容器中运行 StarRocks
- 使用 MinIO 进行对象存储
- 为共享数据配置 StarRocks
- 加载两个公共数据集
- 使用 SELECT 和 JOIN 分析数据
- 基础数据转换（ETL 中的 **T**）

所使用的数据由 NYC OpenData 和 NOAA 国家环境信息中心提供。

这两个数据集都非常大，因为本教程旨在帮助您了解如何使用 StarRocks，所以我们不会加载过去 120 年的数据。您可以在分配给 Docker 的 4 GB RAM 的机器上运行 Docker 镜像并加载此数据。对于更大的容错和可扩展部署，我们有其他文档，稍后将提供。

本文档包含大量信息，开头是分步内容，结尾是技术细节。这样做是为了按以下顺序实现这些目的：

1. 允许读者在共享数据部署中加载数据并分析该数据。
2. 提供共享数据部署的配置细节。
3. 解释加载过程中数据转换的基础知识。


## 先决条件

### Docker

- [Docker](https://docs.docker.com/engine/install/)
- 分配给 Docker 的 4 GB RAM
- 分配给 Docker 的 10 GB 空闲磁盘空间

### SQL 客户端

您可以使用 Docker 环境中提供的 SQL 客户端，或者使用您系统上的客户端。许多兼容 MySQL 的客户端都可以工作，本指南涵盖了 DBeaver 和 MySQL Workbench 的配置。

### curl

`curl` 用于向 StarRocks 发出数据加载作业，并下载数据集。通过在操作系统提示符下运行 `curl` 或 `curl.exe` 来检查是否已安装它。如果未安装 curl，[请在此处获取 curl](https://curl.se/dlwiz/?type=bin)。


## 术语

### FE
前端节点负责元数据管理、客户端连接管理、查询规划和查询调度。每个 FE 在其内存中存储并维护元数据的完整副本，这保证了 FE 之间的无差别服务。

### CN
计算节点负责在共享数据部署中执行查询计划。

### BE
后端节点负责在无共享部署中的数据存储和执行查询计划。

:::note
本指南不使用 BE，此信息包含在此处，以便您了解 BE 和 CN 之间的区别。
:::


## 启动 StarRocks

要使用对象存储运行具有共享数据的 StarRocks，我们需要：

- 一个前端引擎（FE）
- 一个计算节点（CN）
- 对象存储

本指南使用 MinIO，这是一个兼容 S3 的对象存储，根据 GNU Affero 通用公共许可证提供。

为了提供一个包含三个必要容器的环境，StarRocks 提供了一个 Docker compose 文件。

```bash
mkdir quickstart
cd quickstart
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/docker-compose.yml
```

```bash
docker compose up -d
```

检查服务的进度。FE 和 CN 大约需要 30 秒才能变为健康状态。MinIO 容器不会显示健康指示器，但您将使用 MinIO Web UI，这将验证其健康状况。

运行 `docker compose ps` 直到 FE 和 CN 显示 `healthy` 状态：

```bash
docker compose ps
```

```plaintext
SERVICE        CREATED          STATUS                    PORTS
starrocks-cn   25 seconds ago   Up 24 seconds (healthy)   0.0.0.0:8040->8040/tcp
starrocks-fe   25 seconds ago   Up 24 seconds (healthy)   0.0.0.0:8030->8030/tcp, 0.0.0.0:9020->9020/tcp, 0.0.0.0:9030->9030/tcp
minio          25 seconds ago   Up 24 seconds             0.0.0.0:9000-9001->9000-9001/tcp
```


## 生成 MinIO 凭证

为了使用 MinIO 进行对象存储与 StarRocks，您需要生成一个**访问密钥**。

### 打开 MinIO Web UI

浏览到 http://localhost:9001/access-keys。用户名和密码在 Docker compose 文件中指定，分别是 `minioadmin` 和 `minioadmin`。您应该看到还没有访问密钥。点击 **创建访问密钥 +**。

MinIO 会生成一个密钥，点击 **创建** 并下载密钥。

![确保点击创建](../assets/quick-start/MinIO-create.png)

:::note
在点击 **创建** 之前，访问密钥不会被保存，不要只是复制密钥然后离开页面。
:::


## SQL 客户端

<Clients />



## 下载数据

将这两个数据集下载到您的 FE 容器中。

### 在 FE 容器上打开一个 shell

打开一个 shell 并为下载的文件创建一个目录：

```bash
docker compose exec starrocks-fe bash
```

```bash
mkdir quickstart
cd quickstart
```

### 纽约市碰撞数据

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/NYPD_Crash_Data.csv
```

### 天气数据

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/72505394728.csv
```


## 为共享数据配置 StarRocks

此时，您已经运行了 StarRocks 和 MinIO。MinIO 访问密钥用于连接 StarRocks 和 Minio。

### 使用 SQL 客户端连接到 StarRocks

:::tip

从包含 `docker-compose.yml` 文件的目录运行此命令。

如果您使用的是 mysql CLI 以外的客户端，请现在打开它。
:::

```sql
docker compose exec starrocks-fe \
mysql -P9030 -h127.0.0.1 -uroot --prompt="StarRocks > "
```

### 创建存储卷

配置详细信息如下：

- MinIO 服务器可通过 URL `http://minio:9000` 访问
- 上面创建的存储桶名为 `starrocks`
- 写入此卷的数据将存储在存储桶 `starrocks` 中名为 `shared` 的文件夹内
:::tip
首次将数据写入卷时将创建 `shared` 文件夹
:::
- MinIO 服务器未使用 SSL
- MinIO 密钥和密钥分别作为 `aws.s3.access_key` 和 `aws.s3.secret_key` 输入。使用您之前在 MinIO Web UI 中创建的访问密钥。
- 卷 `shared` 是默认卷

:::tip
编辑命令之前，请用您在 MinIO 中创建的访问密钥和密钥替换突出显示的访问密钥信息。
:::

```bash
CREATE STORAGE VOLUME shared
TYPE = S3
LOCATIONS = ("s3://starrocks/shared/")
PROPERTIES
(
    "enabled" = "true",
    "aws.s3.endpoint" = "http://minio:9000",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.enable_ssl" = "false",
    "aws.s3.use_instance_profile" = "false",
    # highlight-start
    "aws.s3.access_key" = "IA2UYcx3Wakpm6sHoFcl",
    "aws.s3.secret_key" = "E33cdRM9MfWpP2FiRpc056Zclg6CntXWa3WPBNMy"
    # highlight-end
);

SET shared AS DEFAULT STORAGE VOLUME;
```

```sql
DESC STORAGE VOLUME shared\G
```

:::tip
本文档以及 StarRocks 文档中的许多其他文档中的 SQL 以 `\G` 结尾，而不是分号。`\G` 使 mysql CLI 垂直呈现查询结果。

许多 SQL 客户端不解释垂直格式输出，因此您应该将 `\G` 替换为 `;`。
:::

```plaintext
*************************** 1. row ***************************
     Name: shared
     Type: S3
IsDefault: true
# highlight-start
 Location: s3://starrocks/shared/
   Params: {"aws.s3.access_key":"******","aws.s3.secret_key":"******","aws.s3.endpoint":"http://minio:9000","aws.s3.region":"us-east-1","aws.s3.use_instance_profile":"false","aws.s3.use_aws_sdk_default_behavior":"false"}
# highlight-end
  Enabled: true
  Comment:
1 row in set (0.03 sec)
```

:::note
直到有数据写入存储桶，`shared` 文件夹才会在 MinIO 对象列表中可见。
:::


## 创建一些表

<DDL />



## 加载两个数据集

有许多方法可以将数据加载到 StarRocks 中。对于本教程，最简单的方法是使用 curl 和 StarRocks Stream Load。

:::tip

从下载数据集的目录中的 FE shell 运行这些 curl 命令。

系统将提示您输入密码。您可能还没有为 MySQL `root` 用户分配密码，因此只需按 Enter 键即可。

:::

curl 命令看起来很复杂，但在教程的最后有详细解释。现在，我们建议您运行命令并运行一些 SQL 来分析数据，然后在最后阅读有关数据加载的详细信息。

### 纽约市碰撞数据 - 碰撞

```bash
curl --location-trusted -u root             \
    -T ./NYPD_Crash_Data.csv                \
    -H "label:crashdata-0"                  \
    -H "column_separator:,"                 \
    -H "skip_header:1"                      \
    -H "enclose:\""                         \
    -H "max_filter_ratio:1"                 \
```
- 使用 SQL JOIN 分析数据，发现在能见度低或结冰的街道上驾驶是一个坏主意

还有更多要学习的内容；我们故意忽略了在 Stream Load 过程中进行的数据转换的细节。有关详细信息，请参阅下面的 curl 命令的注释。

## curl 命令的注释

<Curl />


## 更多信息

[StarRocks 表设计](../table_design/StarRocks_table_design.md)

[材料化视图](../cover_pages/mv_use_cases.mdx)

[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)

[机动车碰撞 - 碰撞](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95) 数据集由纽约市提供，受到这些[使用条款](https://www.nyc.gov/home/terms-of-use.page)和[隐私政策](https://www.nyc.gov/home/privacy-policy.page)的约束。

[本地气候数据](https://www.ncdc.noaa.gov/cdo-web/datatools/lcd)（LCD）由 NOAA 提供，附带此[免责声明](https://www.noaa.gov/disclaimer)和此[隐私政策](https://www.noaa.gov/protecting-your-privacy)。
- 使用 SQL JOIN 分析数据，发现在能见度低或结冰的街道上驾驶是一个坏主意

还有更多内容需要学习；我们故意没有详细介绍在 Stream Load 过程中完成的数据转换。有关详细信息，请参阅下面关于 curl 命令的注释。

## 关于 curl 命令的说明

<Curl />

## 更多信息

[StarRocks 表设计](../table_design/StarRocks_table_design.md)

[物化视图](../cover_pages/mv_use_cases.mdx)

[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)

[机动车辆碰撞 - 碰撞](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95)数据集由纽约市提供，使用受这些[使用条款](https://www.nyc.gov/home/terms-of-use.page)和[隐私政策](https://www.nyc.gov/home/privacy-policy.page)的约束。
