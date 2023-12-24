---
description: Separate compute and storage
displayed_sidebar: English
sidebar_position: 2
---

# 分离存储和计算
从 '../assets/quick-start/_DDL.mdx' 导入 DDL
import Clients from '../assets/quick-start/_clientsCompose.mdx'
从 '../assets/quick-start/_SQL.mdx' 导入 SQL
从 '../assets/quick-start/_curl.mdx' 导入 Curl


在将存储与计算分开的系统中，数据存储在低成本、可靠的远程存储系统中，例如 Amazon S3、Google Cloud Storage、Azure Blob Storage 和其他与 S3 兼容的存储（如 MinIO）。热数据缓存在本地，当缓存命中时，查询性能媲美存算耦合架构。计算节点 （CN） 可以在几秒钟内按需添加或删除。此体系结构可降低存储成本，确保更好的资源隔离，并提供弹性和可伸缩性。

本教程涵盖：

- 在 Docker 容器中运行 StarRocks
- 使用 MinIO 进行对象存储
- 配置 StarRocks 共享数据
- 加载两个公共数据集
- 使用 SELECT 和 JOIN 分析数据
- 基本数据转换（ ** ETL 中的** T）

使用的数据由 NYC OpenData 和 NOAA 的国家环境信息中心提供。

这两个数据集都非常大，因为本教程旨在帮助您了解 StarRocks 的使用，因此我们不会加载过去 120 年的数据。您可以运行 Docker 映像，并在分配给 Docker 的 4 GB RAM 的计算机上加载此数据。对于更大的容错和可扩展部署，我们还有其他文档，稍后会提供。

本文档中有很多信息，开头是分步内容，结尾是技术细节。这样做是为了按以下顺序实现这些目的：

1. 允许读取器在共享数据部署中加载数据并分析该数据。
2. 提供共享数据部署的配置详细信息。
3. 解释加载期间数据转换的基础知识。

---

## 先决条件

### Docker

- [Docker](https://docs.docker.com/engine/install/)
- 分配给 Docker 的 4 GB RAM
- 分配给 Docker 的 10 GB 可用磁盘空间

### SQL 客户端

您可以使用 Docker 环境中提供的 SQL 客户端，也可以在系统上使用一个客户端。许多兼容MySQL的客户端都可以使用，本指南涵盖了DBeaver和MySQL WorkBench的配置。

### curl

`curl` 用于向 StarRocks 下发数据加载作业，并下载数据集。检查您是否通过运行 `curl` 或 `curl.exe` 在操作系统提示符下安装了它。如果未安装  curl，[请在此处获取 curl](https://curl.se/dlwiz/?type=bin)。

---

## 术语

### FE
前端节点负责元数据管理、客户端连接管理、查询规划和查询调度。每个 FE 在其内存中存储并维护一个完整的元数据副本，保证了 FE 之间的无差别服务。

### CN
计算节点负责在共享数据部署中执行查询计划。

### BE
后端节点负责数据存储和在无共享部署中执行查询计划。

:::注意
本指南不使用 BE，此处包含此信息，以便您了解 BE 和 CN 之间的区别。
:::

---

## 启动 StarRocks

要使用对象存储运行具有共享数据的 StarRocks，我们需要：

- 前端引擎 （FE）
- 计算节点 （CN）
- 对象存储

本指南使用 MinIO，它是根据 GNU Affero 通用公共许可证提供的 S3 兼容对象存储。

为了给环境提供三个必要的容器，StarRocks 提供了一个 Docker compose 文件。 

```bash
mkdir quickstart
cd quickstart
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/docker-compose.yml
```

```bash
docker compose up -d
```

检查服务进度。FE 和 CN 大约需要 30 秒才能恢复健康。MinIO 容器不会显示运行状况指示器，但你将使用 MinIO Web UI，这将验证其运行状况。

运行 `docker compose ps` 直到 FE 和 CN 显示以下状态 `healthy`：

```bash
docker compose ps
```

```plaintext
SERVICE        CREATED          STATUS                    PORTS
starrocks-cn   25 seconds ago   Up 24 seconds (healthy)   0.0.0.0:8040->8040/tcp
starrocks-fe   25 seconds ago   Up 24 seconds (healthy)   0.0.0.0:8030->8030/tcp, 0.0.0.0:9020->9020/tcp, 0.0.0.0:9030->9030/tcp
minio          25 seconds ago   Up 24 seconds             0.0.0.0:9000-9001->9000-9001/tcp
```

---

## 生成 MinIO 凭据

为了在 StarRocks 中使用 MinIO for Object Storage，您需要生成一个 **访问密钥**。

### 打开 MinIO Web UI

浏览到 http://localhost:9001/access-keys 用户名和密码在 Docker 复合文件中指定，并且是 `minioadmin` 和 `minioadmin`。您应该看到还没有访问密钥。单击创建 **访问密钥 +**。

MinIO 将生成一个密钥，单击“ **创建** ”并下载密钥。

![确保单击“创建”](../assets/quick-start/MinIO-create.png)

:::注意
在单击“创建”之前，不会保存访问密钥****，不要只是复制密钥并离开页面
:::

---

## SQL 客户端

<Clients />

---

## 下载数据

将这两个数据集下载到您的 FE 容器中。

### 在 FE 容器上打开 shell

打开 shell 并为下载的文件创建一个目录：

```bash
docker compose exec starrocks-fe bash
```

```bash
mkdir quickstart
cd quickstart
```

### 纽约市车祸数据

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/NYPD_Crash_Data.csv
```

### 天气数据

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/72505394728.csv
```

---

## 配置 StarRocks 共享数据

此时，StarRocks 正在运行，MinIO 也在运行。MinIO 访问密钥用于连接 StarRocks 和 Minio。

### 使用 SQL 客户端连接 StarRocks

:::tip

从包含该文件的目录运行此命令 `docker-compose.yml` 。

如果您使用的是mysql CLI以外的客户端，请立即打开它。
:::

```sql
docker compose exec starrocks-fe \
mysql -P9030 -h127.0.0.1 -uroot --prompt="StarRocks > "
```

### 创建存储卷

配置的详细信息如下所示：

- MinIO 服务器位于 URL 上 `http://minio:9000`
- 上面创建的存储桶名为 `starrocks`
- 写入此卷的数据将存储在 `shared` 存储桶`starrocks`中名为的文件夹中
:::tip
`shared` 首次将数据写入卷时，将创建该文件夹
:::
- MinIO 服务器未使用 SSL
- MinIO 密钥和密钥以 和 的形式输入 `aws.s3.access_key` `aws.s3.secret_key`。使用您之前在 MinIO Web UI 中创建的访问密钥。
- 卷 `shared` 是默认卷
:::tip
在运行命令之前对其进行编辑，并将突出显示的访问密钥信息替换为您在 MinIO 中创建的访问密钥和密钥。
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
本文档中的一些 SQL，以及 StarRocks 文档中的许多其他文档，以及 `\G` 
分号。这 `\G` 会导致mysql CLI垂直呈现查询结果。

许多 SQL 客户端不解释垂直格式设置输出，因此应将 `\G` `;`替换为 .
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
在 `shared` 将数据写入存储桶之前，该文件夹在 MinIO 对象列表中不可见。
:::

---

## 创建一些表

<DDL />

---

## 加载两个数据集

向 StarRocks 加载数据的方法有很多种。在本教程中，最简单的方法是使用 curl 和 StarRocks Stream Load。

:::tip

从下载数据集的目录中的 FE shell 运行这些 curl 命令。

系统将提示您输入密码。您可能尚未为MySQL用户分配密码 `root` ，因此只需按回车键即可。

:::

这些 `curl` 命令看起来很复杂，但在本教程的末尾进行了详细解释。现在，我们建议运行命令并运行一些 SQL 来分析数据，然后在最后阅读有关数据加载的详细信息。

### 纽约市碰撞数据 - 车祸

```bash
curl --location-trusted -u root             \
    -T ./NYPD_Crash_Data.csv                \
    -H "label:crashdata-0"                  \
    -H "column_separator:,"                 \
    -H "skip_header:1"                      \
    -H "enclose:\""                         \
    -H "max_filter_ratio:1"                 \

    -H "columns:tmp_CRASH_DATE, tmp_CRASH_TIME, CRASH_DATE=str_to_date(concat_ws(' ', tmp_CRASH_DATE, tmp_CRASH_TIME), '%m/%d/%Y %H:%i'),BOROUGH,ZIP_CODE,LATITUDE,LONGITUDE,LOCATION,ON_STREET_NAME,CROSS_STREET_NAME,OFF_STREET_NAME,NUMBER_OF_PERSONS_INJURED,NUMBER_OF_PERSONS_KILLED,NUMBER_OF_PEDESTRIANS_INJURED,NUMBER_OF_PEDESTRIANS_KILLED,NUMBER_OF_CYCLIST_INJURED,NUMBER_OF_CYCLIST_KILLED,NUMBER_OF_MOTORIST_INJURED,NUMBER_OF_MOTORIST_KILLED,CONTRIBUTING_FACTOR_VEHICLE_1,CONTRIBUTING_FACTOR_VEHICLE_2,CONTRIBUTING_FACTOR_VEHICLE_3,CONTRIBUTING_FACTOR_VEHICLE_4,CONTRIBUTING_FACTOR_VEHICLE_5,COLLISION_ID,VEHICLE_TYPE_CODE_1,VEHICLE_TYPE_CODE_2,VEHICLE_TYPE_CODE_3,VEHICLE_TYPE_CODE_4,VEHICLE_TYPE_CODE_5" \
    -XPUT http://localhost:8030/api/quickstart/crashdata/_stream_load
```

以下是上述命令的输出。第一个突出显示的部分显示了您应该期望看到的内容（OK 和除一行外的所有行均已插入）。有一行被过滤掉，因为它的列数不正确。

```bash
输入用户 root 的主机密码：
{
    "TxnId": 2,
    "Label": "crashdata-0",
    "Status": "Success",
    # 突出显示-开始
    "Message": "OK",
    "NumberTotalRows": 423726,
    "NumberLoadedRows": 423725,
    # 突出显示-结束
    "NumberFilteredRows": 1,
    "NumberUnselectedRows": 0,
    "LoadBytes": 96227746,
    "LoadTimeMs": 1013,
    "BeginTxnTimeMs": 21,
    "StreamLoadPlanTimeMs": 63,
    "ReadDataTimeMs": 563,
    "WriteDataTimeMs": 870,
    "CommitAndPublishTimeMs": 57,
    # 突出显示-开始
    "ErrorURL": "http://10.5.0.3:8040/api/_load_error_log?file=error_log_da41dd88276a7bfc_739087c94262ae9f"
    # 突出显示-结束
}%
```

如果出现错误，输出将提供一个 URL 以查看错误消息。由于容器具有私有 IP 地址，您将需要通过在容器中运行 curl 来查看它。

```bash
curl http://10.5.0.3:8040/api/_load_error_log<details from ErrorURL>
```

展开摘要以查看开发本教程时看到的内容：

<details>

<summary>在浏览器中阅读错误消息</summary>

```bash
错误：值计数与列计数不匹配。期望 29，但得到 32。

列分隔符：44，行分隔符：10.. 行：09/06/2015,14:15,,,40.6722269,-74.0110059,"(40.6722269, -74.0110059)",,,"R/O 1 BEARD ST. ( IKEA'S 
09/14/2015,5:30,BRONX,10473,40.814551,-73.8490955,"(40.814551, -73.8490955)",TORRY AVENUE                    ,NORTON AVENUE                   ,,0,0,0,0,0,0,0,0,Driver Inattention/Distraction,Unspecified,,,,3297457,PASSENGER VEHICLE,PASSENGER VEHICLE,,,
```

</details>

### 天气数据

以与加载碰撞数据相同的方式加载天气数据集。

```bash
curl --location-trusted -u root             \
    -T ./72505394728.csv                    \
    -H "label:weather-0"                    \
    -H "column_separator:,"                 \
    -H "skip_header:1"                      \
    -H "enclose:\""                         \
    -H "max_filter_ratio:1"                 \
    -H "columns: STATION, DATE, LATITUDE, LONGITUDE, ELEVATION, NAME, REPORT_TYPE, SOURCE, HourlyAltimeterSetting, HourlyDewPointTemperature, HourlyDryBulbTemperature, HourlyPrecipitation, HourlyPresentWeatherType, HourlyPressureChange, HourlyPressureTendency, HourlyRelativeHumidity, HourlySkyConditions, HourlySeaLevelPressure, HourlyStationPressure, HourlyVisibility, HourlyWetBulbTemperature, HourlyWindDirection, HourlyWindGustSpeed, HourlyWindSpeed, Sunrise, Sunset, DailyAverageDewPointTemperature, DailyAverageDryBulbTemperature, DailyAverageRelativeHumidity, DailyAverageSeaLevelPressure, DailyAverageStationPressure, DailyAverageWetBulbTemperature, DailyAverageWindSpeed, DailyCoolingDegreeDays, DailyDepartureFromNormalAverageTemperature, DailyHeatingDegreeDays, DailyMaximumDryBulbTemperature, DailyMinimumDryBulbTemperature, DailyPeakWindDirection, DailyPeakWindSpeed, DailyPrecipitation, DailySnowDepth, DailySnowfall, DailySustainedWindDirection, DailySustainedWindSpeed, DailyWeather, MonthlyAverageRH, MonthlyDaysWithGT001Precip, MonthlyDaysWithGT010Precip, MonthlyDaysWithGT32Temp, MonthlyDaysWithGT90Temp, MonthlyDaysWithLT0Temp, MonthlyDaysWithLT32Temp, MonthlyDepartureFromNormalAverageTemperature, MonthlyDepartureFromNormalCoolingDegreeDays, MonthlyDepartureFromNormalHeatingDegreeDays, MonthlyDepartureFromNormalMaximumTemperature, MonthlyDepartureFromNormalMinimumTemperature, MonthlyDepartureFromNormalPrecipitation, MonthlyDewpointTemperature, MonthlyGreatestPrecip, MonthlyGreatestPrecipDate, MonthlyGreatestSnowDepth, MonthlyGreatestSnowDepthDate, MonthlyGreatestSnowfall, MonthlyGreatestSnowfallDate, MonthlyMaxSeaLevelPressureValue, MonthlyMaxSeaLevelPressureValueDate, MonthlyMaxSeaLevelPressureValueTime, MonthlyMaximumTemperature, MonthlyMeanTemperature, MonthlyMinSeaLevelPressureValue, MonthlyMinSeaLevelPressureValueDate, MonthlyMinSeaLevelPressureValueTime, MonthlyMinimumTemperature, MonthlySeaLevelPressure, MonthlyStationPressure, MonthlyTotalLiquidPrecipitation, MonthlyTotalSnowfall, MonthlyWetBulb, AWND, CDSD, CLDD, DSNW, HDSD, HTDD, NormalsCoolingDegreeDay, NormalsHeatingDegreeDay, ShortDurationEndDate005, ShortDurationEndDate010, ShortDurationEndDate015, ShortDurationEndDate020, ShortDurationEndDate030, ShortDurationEndDate045, ShortDurationEndDate060, ShortDurationEndDate080, ShortDurationEndDate100, ShortDurationEndDate120, ShortDurationEndDate150, ShortDurationEndDate180, ShortDurationPrecipitationValue005, ShortDurationPrecipitationValue010, ShortDurationPrecipitationValue015, ShortDurationPrecipitationValue020, ShortDurationPrecipitationValue030, ShortDurationPrecipitationValue045, ShortDurationPrecipitationValue060, ShortDurationPrecipitationValue080, ShortDurationPrecipitationValue100, ShortDurationPrecipitationValue120, ShortDurationPrecipitationValue150, ShortDurationPrecipitationValue180, REM, BackupDirection, BackupDistance, BackupDistanceUnit, BackupElements, BackupElevation, BackupEquipment, BackupLatitude, BackupLongitude, BackupName, WindEquipmentChangeDate" \
    -XPUT http://localhost:8030/api/quickstart/weatherdata/_stream_load
```

---

## 验证数据是否存储在 MinIO 中

打开 MinIO [http://localhost:9001/browser/starrocks/](http://localhost:9001/browser/starrocks/)，并验证 `starrocks/shared/` 下每个目录中的 `data`、`metadata` 和 `schema` 条目。

:::提示
加载数据时，`starrocks/shared/` 下的文件夹名称是动态生成的。您应该在 `shared` 下面看到一个目录，然后在下面看到另外两个目录。在每个目录中，您将找到 `data`、`metadata` 和 `schema` 条目。

![MinIO 对象浏览器](../assets/quick-start/MinIO-data.png)
:::

---

## 回答一些问题

<SQL />

---

## 配置 StarRocks 共享数据

现在，您已经体验过将 StarRocks 与共享数据一起使用，了解其配置非常重要。 

### CN 配置

此处使用的 CN 配置是默认配置，因为 CN 是为共享数据使用而设计的。默认配置如下。您无需进行任何更改。

```bash
sys_log_level = INFO

be_port = 9060
be_http_port = 8040
heartbeat_service_port = 9050
brpc_port = 8060
```

### FE 配置

FE 配置与默认配置略有不同，因为 FE 必须配置为期望数据存储在对象存储中，而不是存储在 BE 节点上的本地磁盘上。

该文件 `docker-compose.yml` 在 . `command`

```yml
    command: >
      bash -c "echo run_mode=shared_data >> /opt/starrocks/fe/conf/fe.conf &&
      echo cloud_native_meta_port=6090 >> /opt/starrocks/fe/conf/fe.conf &&
      echo aws_s3_path=starrocks >> /opt/starrocks/fe/conf/fe.conf &&
      echo aws_s3_endpoint=minio:9000 >> /opt/starrocks/fe/conf/fe.conf &&
      echo aws_s3_use_instance_profile=false >> /opt/starrocks/fe/conf/fe.conf &&
      echo cloud_native_storage_type=S3 >> /opt/starrocks/fe/conf/fe.conf &&
      echo aws_s3_use_aws_sdk_default_behavior=true >> /opt/starrocks/fe/conf/fe.conf &&
      sh /opt/starrocks/fe/bin/start_fe.sh"
```

这将产生以下配置文件：

```bash title='fe/fe.conf'
LOG_DIR = ${STARROCKS_HOME}/log

DATE = "$(date +%Y%m%d-%H%M%S)"
JAVA_OPTS="-Dlog4j2.formatMsgNoLookups=true -Xmx8192m -XX:+UseMembar -XX:SurvivorRatio=8 -XX:MaxTenuringThreshold=7 -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+CMSClassUnloadingEnabled -XX:-CMSParallelRemarkEnabled -XX:CMSInitiatingOccupancyFraction=80 -XX:SoftRefLRUPolicyMSPerMB=0 -Xloggc:${LOG_DIR}/fe.gc.log.$DATE -XX:+PrintConcurrentLocks"

JAVA_OPTS_FOR_JDK_11="-Dlog4j2.formatMsgNoLookups=true -Xmx8192m -XX:+UseG1GC -Xlog:gc*:${LOG_DIR}/fe.gc.log.$DATE:time"

sys_log_level = INFO

http_port = 8030
rpc_port = 9020
query_port = 9030
edit_log_port = 9010
mysql_service_nio_enabled = true

# 突出显示-开始
run_mode=shared_data
aws_s3_path=starrocks
aws_s3_endpoint=minio:9000
aws_s3_use_instance_profile=false
cloud_native_storage_type=S3
aws_s3_use_aws_sdk_default_behavior=true
# 突出显示-结束
```

:::注意
此配置文件包含默认条目和共享数据的添加项。突出显示共享数据的条目。
:::

非默认 FE 配置设置：

:::注意
许多配置参数都以 为前缀`s3_`。此前缀用于所有与 Amazon S3 兼容的存储类型（例如：S3、GCS 和 MinIO）。使用 Azure Blob 存储时，前缀为 `azure_`.
:::

#### `run_mode=shared_data`

这样就可以使用共享数据。

#### `aws_s3_path=starrocks`

存储桶名称。

#### `aws_s3_endpoint=minio:9000`

MinIO 终结点，包括端口号。

#### `aws_s3_use_instance_profile=false`

使用 MinIO 时，会使用访问密钥，因此实例配置文件不会与 MinIO 一起使用。

#### `cloud_native_storage_type=S3`

这指定是使用 S3 兼容存储还是 Azure Blob 存储。对于 MinIO，这始终是 S3。

#### `aws_s3_use_aws_sdk_default_behavior=true`

使用 MinIO 时，此参数始终设置为 true。

---

## 总结

在本教程中，您：

- 在 Docker 中部署了 StarRocks 和 Minio
- 创建了 MinIO 访问密钥
- 配置了使用 MinIO 的 StarRocks 存储卷
- 加载了纽约市提供的碰撞数据和 NOAA 提供的天气数据
- 使用 SQL JOIN 分析数据，发现在能见度低或结冰的街道上行驶是个坏主意


还有更多内容需要了解；我们故意在流加载过程中忽略了数据转换的细节。有关此内容的详细信息，请参阅下面有关 curl 命令的注释。

## 关于 curl 命令的注释

<Curl />

## 更多信息

[StarRocks 表设计](../table_design/StarRocks_table_design.md)

[物化视图](../cover_pages/mv_use_cases.mdx)

[流加载](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)

[机动车辆碰撞 - 撞车](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95) 数据集由纽约市提供，受到这些[使用条款](https://www.nyc.gov/home/terms-of-use.page)和[隐私政策](https://www.nyc.gov/home/privacy-policy.page)约束。

[当地气候数据](https://www.ncdc.noaa.gov/cdo-web/datatools/lcd)（LCD）由美国国家海洋和大气管理局提供，并附有此[免责声明](https://www.noaa.gov/disclaimer)和此[隐私政策](https://www.noaa.gov/protecting-your-privacy)。