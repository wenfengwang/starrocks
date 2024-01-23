---
description: 分离计算和存储
displayed_sidebar: English
sidebar_position: 2
---

# 分离存储和计算
从'../assets/quick-start/_DDL.mdx'导入DDL
从'../assets/quick-start/_clientsCompose.mdx'导入Clients
从'../assets/quick-start/_SQL.mdx'导入SQL
从'../assets/quick-start/_curl.mdx'导入Curl

在将存储与计算分离的系统中，数据被存储在低成本、可靠的远程存储系统中，如Amazon S3、Google Cloud Storage、Azure Blob Storage以及其他兼容S3的存储，例如MinIO。热数据被本地缓存，当缓存命中时，查询性能可与存储-计算耦合架构相媲美。计算节点（CN）可以根据需求在几秒钟内增加或移除。这种架构降低了存储成本，确保了更好的资源隔离，并提供了弹性和可扩展性。

本教程包括：

- 在 Docker 容器中运行 StarRocks
- 使用 MinIO 作为对象存储
- 配置 StarRocks 用于共享数据
- 加载两个公开数据集
- 使用 SELECT 和 JOIN 分析数据
- 基本数据转换（ETL中的**T**）

使用的数据由NYC OpenData和美国国家海洋和大气管理局的国家环境信息中心提供。

这两个数据集都非常大，由于本教程旨在帮助您熟悉如何使用 StarRocks，我们不打算加载过去120年的数据。您可以运行 Docker 镜像，并在分配给 Docker 的 4 GB RAM 的机器上加载这些数据。对于更大型的容错性和可扩展性部署，我们有其他的文档，将在稍后提供。

本文档包含大量信息，内容的安排是：开头部分按步骤展开，末尾部分则提供技术细节。这样的安排是为了按照以下顺序服务于这些目的：

1. 允许用户在共享数据部署中加载数据并分析这些数据。
2. 提供共享数据部署的配置详情。
3. 解释在加载期间数据转换的基础。

---

## 先决条件

### Docker

- [Docker](https://docs.docker.com/engine/install/)
- 分配给 Docker 的 4 GB RAM
- 分配给 Docker 的 10 GB 免费磁盘空间

### SQL 客户端

您可以使用 Docker 环境中提供的 SQL 客户端，或者在您的系统上使用一个。许多兼容 MySQL 的客户端都可以工作，本指南涵盖了 DBeaver 和 MySQL Workbench 的配置。

### curl

`curl` 用于向 StarRocks 发起数据加载作业，并下载数据集。您可以通过在操作系统提示符下运行 `curl` 或 `curl.exe` 来检查是否已安装它。如果您尚未安装 curl，[请在此处下载 curl](https://curl.se/dlwiz/?type=bin)。

---

## 术语

### 铁
前端节点负责元数据管理、客户端连接管理、查询规划和查询调度。每个FE都在其内存中存储并维护一份完整的元数据副本，这保证了FEs之间提供无差别的服务。

### 快递 之 家
计算节点负责在共享数据部署中执行查询计划。

### 是
后端节点负责数据存储和在无共享部署中执行查询计划。

:::note
本指南不使用 BEs，此信息包含在这里是为了帮助您理解 BEs 和 CNs 之间的区别。
:::

---

## 启动 StarRocks

要使用对象存储运行具有共享数据的 StarRocks，我们需要：

- 一个前端引擎（FE）
- 计算节点（CN）
- 对象存储

本指南使用 MinIO，这是一种兼容 S3 的对象存储，根据 GNU Affero 通用公共许可证提供。

为了提供一个包含三个必要容器的环境，StarRocks 提供了一个 Docker compose 文件。

```bash
mkdir quickstart
cd quickstart
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/docker-compose.yml
```

```bash
docker compose up -d
```

检查服务的进度。前端（FE）和内容节点（CN）大约需要30秒时间才能变为健康状态。MinIO容器不会显示健康指示器，但你将使用MinIO的Web界面，这将验证其健康状况。

运行 `docker compose ps` 直到 FE 和 CN 显示状态为 `healthy`：

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

## 生成 MinIO 凭证

为了在 StarRocks 中使用 MinIO 进行对象存储，您需要生成一个**访问密钥**。

### 打开 MinIO 网络用户界面

浏览到 [http://localhost:9001/access-keys](http://localhost:9001/access-keys) 用户名和密码在 Docker compose 文件中指定，分别为 `minioadmin` 和 `minioadmin`。您应该会看到目前还没有访问密钥。点击 **创建访问密钥 +**。

MinIO 将生成一个密钥，点击**创建**并下载密钥。

![确保点击“创建”](../assets/quick-start/MinIO-create.png)

:::note
在您点击**创建**之前，访问密钥不会被保存，请不要仅仅复制密钥然后离开页面
:::

---

## SQL 客户端

<Clients />


---

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

### 纽约市交通事故数据

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/NYPD_Crash_Data.csv
```

### 天气数据

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/72505394728.csv
```

---

## 为共享数据配置 StarRocks

此时，您已经成功运行了 StarRocks 和 MinIO。MinIO 的访问密钥用于连接 StarRocks 和 MinIO。

### 使用 SQL 客户端连接到 StarRocks

:::提示

从包含 `docker-compose.yml` 文件的目录中运行此命令。

如果您使用的是除mysql CLI之外的其他客户端，请现在打开它。
:::

```sql
docker compose exec starrocks-fe \
mysql -P9030 -h127.0.0.1 -uroot --prompt="StarRocks > "
```

### 创建存储卷

下面显示的配置详情：

- MinIO 服务器可通过此 URL 访问：`http://minio:9000`
- 上面创建的存储桶命名为 `starrocks`
- 写入此卷的数据将被存储在名为 `shared` 的文件夹中，该文件夹位于 `starrocks` 存储桶内
::::::
- MinIO 服务器没有使用 SSL
- MinIO 密钥和密钥分别以 `aws.s3.access_key` 和 `aws.s3.secret_key` 的形式输入。使用您之前在 MinIO Web UI 中创建的访问密钥。
- 卷 `shared` 是默认卷

:::tip
在您运行命令之前，请编辑该命令，并将突出显示的访问密钥信息替换成您在MinIO中创建的访问密钥和密钥。
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
本文档以及 StarRocks 文档中的许多其他文档中的一些 SQL 语句，使用的是 `\G` 而不是分号。`\\(G)` 会使 mysql 命令行界面（CLI）垂直显示查询结果。

许多 SQL 客户端不支持垂直格式的输出，因此你应该用分号 `;` 来替换 `\\(G)`。
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
在数据写入存储桶之前，`shared` 文件夹在 MinIO 对象列表中不会显示。
:::

---

## 创建一些表格

<DDL />


---

## 加载两个数据集

有许多方法可以将数据加载到 StarRocks 中。对于本教程来说，最简单的方式是使用 curl 和 StarRocks Stream Load。

:::提示

在您下载数据集的目录中，从 FE shell 运行这些 curl 命令。

系统会提示您输入密码。您可能还没有为 MySQL 的 `root` 用户设置密码，所以直接按回车键即可。
:::

这些 `curl` 命令看起来很复杂，但在教程的最后会有详细的解释。目前，我们建议先运行这些命令并执行一些 SQL 来分析数据，然后再阅读教程最后部分关于数据加载的详细信息。

### 纽约市碰撞数据 - 碰撞事故

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

这是上述命令的输出结果。第一个高亮部分显示了您应该期望看到的内容（显示OK并且除了一行之外的所有行都被插入）。有一行因为不包含正确的列数而被过滤掉了。

```bash
Enter host password for user 'root':
{
    "TxnId": 2,
    "Label": "crashdata-0",
    "Status": "Success",
    # highlight-start
    "Message": "OK",
    "NumberTotalRows": 423726,
    "NumberLoadedRows": 423725,
    # highlight-end
    "NumberFilteredRows": 1,
    "NumberUnselectedRows": 0,
    "LoadBytes": 96227746,
    "LoadTimeMs": 1013,
    "BeginTxnTimeMs": 21,
    "StreamLoadPlanTimeMs": 63,
    "ReadDataTimeMs": 563,
    "WriteDataTimeMs": 870,
    "CommitAndPublishTimeMs": 57,
    # highlight-start
    "ErrorURL": "http://10.5.0.3:8040/api/_load_error_log?file=error_log_da41dd88276a7bfc_739087c94262ae9f"
    # highlight-end
}%
```

如果出现错误，输出会提供一个 URL 以查看错误信息。因为容器有一个私有 IP 地址，你需要通过在容器内运行 curl 命令来查看。

```bash
curl http://10.5.0.3:8040/api/_load_error_log<details from ErrorURL>
```

展开本教程开发过程中看到的内容摘要：

<details>


<summary>在浏览器中读取错误消息</summary>


```bash
Error: Value count does not match column count. Expect 29, but got 32.

Column delimiter: 44,Row delimiter: 10.. Row: 09/06/2015,14:15,,,40.6722269,-74.0110059,"(40.6722269, -74.0110059)",,,"R/O 1 BEARD ST. ( IKEA'S 
09/14/2015,5:30,BRONX,10473,40.814551,-73.8490955,"(40.814551, -73.8490955)",TORRY AVENUE                    ,NORTON AVENUE                   ,,0,0,0,0,0,0,0,0,Driver Inattention/Distraction,Unspecified,,,,3297457,PASSENGER VEHICLE,PASSENGER VEHICLE,,,
```

</details>


### 天气数据

以与加载崩溃数据相同的方式加载天气数据集。

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

打开 MinIO [http://localhost:9001/browser/starrocks/](http://localhost:9001/browser/starrocks/)，并验证在 `starrocks/shared/` 下的每个目录中都有 `data`、`metadata` 和 `schema` 条目

:::tip
在加载数据后，`starrocks/shared/` 下面的文件夹名称会被生成。您应该能看到`shared`下面的一个单独目录，然后在那个目录下面再有两个目录。在这些目录中，您将会找到数据、元数据和模式条目。

![MinIO 对象浏览器](../assets/quick-start/MinIO-data.png)
:::

---

## 回答一些问题

<SQL />


---

## 配置 StarRocks 用于共享数据

现在您已经体验过使用StarRocks与共享数据，理解其配置是非常重要的。

### CN 配置

此处使用的 CN 配置是默认配置，因为 CN 是为共享数据使用而设计的。默认配置如下所示。您无需进行任何更改。

```bash
sys_log_level = INFO

be_port = 9060
be_http_port = 8040
heartbeat_service_port = 9050
brpc_port = 8060
```

### FE 配置

FE 配置与默认配置略有不同，因为必须将 FE 配置为期望数据存储在对象存储中，而不是 BE 节点的本地磁盘上。

`docker-compose.yml` 文件在 `command` 中生成 FE 配置。

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

这将生成以下配置文件：

```bash
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

# highlight-start
run_mode=shared_data
aws_s3_path=starrocks
aws_s3_endpoint=minio:9000
aws_s3_use_instance_profile=false
cloud_native_storage_type=S3
aws_s3_use_aws_sdk_default_behavior=true
# highlight-end
```

:::note
此配置文件包含默认条目和共享数据的附加条目。共享数据的条目已被突出显示。
:::

非默认的 FE 配置设置：

:::note
许多配置参数都以`s3_`为前缀。此前缀用于所有与Amazon S3兼容的存储类型（例如：S3、GCS和MinIO）。使用Azure Blob存储时，前缀为`azure_`。
:::

#### `run_mode=shared_data`

这样就可以使用共享数据。

#### `aws_s3_path=starrocks`

存储桶名称。

#### `aws_s3_endpoint=minio:9000`

MinIO 端点，包括端口号。

#### `aws_s3_use_instance_profile=false`

当使用 MinIO 时，需要使用访问密钥，因此不会使用实例配置文件。

#### `cloud_native_storage_type=S3`

这指定是使用 S3 兼容存储还是 Azure Blob 存储。对于 MinIO，这始终是 S3。

#### `aws_s3_use_aws_sdk_default_behavior=true`

使用 MinIO 时，此参数始终设置为 true。

---

## 总结

在本教程中，你将：

- 在 Docker 中部署了 StarRocks 和 Minio
- 已创建 MinIO 访问密钥
- 配置了一个使用 MinIO 的 StarRocks 存储卷
- 由纽约市提供的交通事故数据和由NOAA提供的天气数据
- 使用 SQL JOIN 分析数据，找出在能见度低或有冰的街道上驾驶是个坏主意

还有更多内容需要学习；我们有意略过了在流加载期间进行的数据转换。关于这方面的详细信息，请参阅下面关于 curl 命令的注释。

## 关于 curl 命令的笔记

<Curl />


## 更多信息

[StarRocks 表设计](../table_design/StarRocks_table_design.md)

[物化视图](../cover_pages/mv_use_cases.mdx)

[流式加载](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)

[机动车辆碰撞 - 事故](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95) 数据集由纽约市提供，使用须遵守这些[使用条款](https://www.nyc.gov/home/terms-of-use.page)和[隐私政策](https://www.nyc.gov/home/privacy-policy.page)。

[本地气象数据](https://www.ncdc.noaa.gov/cdo-web/datatools/lcd)（LCD）由NOAA提供，附带此[免责声明](https://www.noaa.gov/disclaimer)和此[隐私政策](https://www.noaa.gov/protecting-your-privacy)。
