---
displayed_sidebar: "中文"
sidebar_position: 2
description: 分离计算和存储
---

# 分开存储和计算
import DDL from '../assets/quick-start/_DDL.mdx'
import Clients from '../assets/quick-start/_clientsCompose.mdx'
import SQL from '../assets/quick-start/_SQL.mdx'
import Curl from '../assets/quick-start/_curl.mdx'


在将存储与计算分离的系统中，数据存储在低成本的可靠远程存储系统中，例如Amazon S3，Google Cloud Storage，Azure Blob Storage以及其他S3兼容存储，例如MinIO。热数据被缓存在本地，当命中缓存时，查询性能与存储-计算耦合架构相当。计算节点（CN）可以在几秒钟内根据需要添加或移除。此架构降低了存储成本，确保更好的资源隔离，并提供了弹性和可扩展性。

本教程涵盖以下内容：

- 在Docker容器中运行StarRocks
- 使用MinIO进行对象存储
- 配置StarRocks以进行共享数据
- 加载两个公共数据集
- 使用SELECT和JOIN分析数据
- 基本数据转换（ETL中的T）

使用的数据由NYC OpenData和NOAA的National Centers for Environmental Information提供。

这两个数据集都非常庞大，由于本教程旨在帮助您了解如何使用StarRocks，因此我们不会加载过去120年的数据。您可以在分配了4 GB RAM的机器上运行Docker映像并加载这些数据。对于更大的容错和可扩展部署，我们有其他文档，并将在以后提供。

本文档中包含大量信息，最初是逐步内容，最后是技术细节。这样做是为了按照这个顺序服务这些目的：

1. 允许读者在共享数据部署中加载数据并分析该数据。
2. 提供共享数据部署的配置详情。
3. 在加载期间解释数据转换的基础知识。

---

## 先决条件

### Docker

- [Docker](https://docs.docker.com/engine/install/)
- 为Docker分配4 GB RAM
- 为Docker分配10 GB的可用磁盘空间

### SQL客户端

您可以使用Docker环境中提供的SQL客户端，也可以使用系统上的客户端。许多与MySQL兼容的客户端都可以使用，此指南涵盖了DBeaver和MySQL WorkBench的配置。

### curl

`curl`用于向StarRocks发出数据加载作业以及下载数据集。通过在操作系统提示符下运行`curl`或`curl.exe`来检查是否已安装curl。如果未安装curl，[在此处获取curl](https://curl.se/dlwiz/?type=bin)。

---

## 术语

### FE

前端节点负责元数据管理，客户端连接管理，查询计划和查询调度。每个FE在其内存中存储并维护元数据的完整副本，以保证各个FE之间的服务不加区别。

### CN

计算节点负责在共享数据部署中执行查询计划。

### BE

后端节点负责数据存储和在共享无内容的部署中执行查询计划。

:::note
本指南不使用BE，此信息包含在此处以便您了解BE和CN之间的区别。
:::

---

## 启动StarRocks

要使用对象存储运行StarRocks的共享数据，我们需要：

- 一个前端引擎（FE）
- 一个计算节点（CN）
- 对象存储

本指南使用MinIO，这是根据GNU Affero通用公共许可证提供的与S3兼容的对象存储。

为了提供一个带有三个必要容器的环境，StarRocks提供了一个Docker compose文件。

```bash
mkdir quickstart
cd quickstart
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/docker-compose.yml
```

```bash
docker compose up -d
```

检查服务的进展。FE和CN应该需要大约30秒才能变为健康状态。MinIO容器将不会显示健康指示器，但您将使用MinIO Web UI并验证其健康状态。

运行`docker compose ps`，直到FE和CN显示`健康`状态：

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

## 生成MinIO凭证

为了将MinIO用于对象存储与StarRocks连接，您需要生成访问密钥。

### 打开MinIO Web UI

浏览到 http://localhost:9001/access-keys 用户名和密码在Docker compose文件中指定，为`minioadmin`和`minioadmin`。您应该看到尚未有访问密钥。点击**创建访问密钥+**。

MinIO将生成一个密钥，点击**创建**并下载该密钥。

![确保点击创建](../assets/quick-start/MinIO-create.png)

:::note
只有当点击**创建**后，访问密钥才会被保存，不要只复制密钥并转离该页面
:::

---

## SQL客户端

<Clients />

---

## 下载数据

将这两个数据集下载到您的FE容器。

### 在FE容器上打开一个shell

打开一个shell并创建一个用于下载文件的目录：

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

## 配置StarRocks以进行共享数据

此时，您已经运行了StarRocks，并且已经运行了MinIO。MinIO访问密钥用于连接StarRocks和Minio。

### 使用SQL客户端连接到StarRocks

:::tip

从包含`docker-compose.yml`文件的目录中运行此命令。

如果您使用的客户端不是mysql CLI，则现在打开该客户端。
:::

```sql
docker compose exec starrocks-fe \
mysql -P9030 -h127.0.0.1 -uroot --prompt="StarRocks > "
```

### 创建存储卷

下面显示的配置的详细信息：

- MinIO服务器的URL为 `http://minio:9000`
- 上面创建的桶的名称为 `starrocks`
- 写入该卷的数据将存储在名为`shared`的文件夹中的`starrocks`桶内
:::tip
第一次写入数据到卷时将会创建`shared`文件夹
:::
- MinIO服务器未使用SSL
- 将MinIO密钥和密码输入为`aws.s3.access_key`和`aws.s3.secret_key`。 使用您在MinIO Web UI中创建的访问密钥。

:::tip
编辑命令之前并用MinIO中创建的访问密钥和密码替换突出显示的访问密钥信息后运行它。
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
本文档中的某些SQL以及StarRocks文档中的许多其他文档用`\G`而不是分号。 `\G`会导致mysql CLI以纵向方式呈现查询结果。

许多SQL客户端不解释纵向格式输出，因此您应该将`\G`替换为`；`。
:::

```plaintext
*************************** 1. row ***************************
     Name: shared
     Type: S3
IsDefault: true
# highlight-start
位置：s3://starrocks/shared/
   参数：{"aws.s3.access_key":"******","aws.s3.secret_key":"******","aws.s3.endpoint":"http://minio:9000","aws.s3.region":"us-east-1","aws.s3.use_instance_profile":"false","aws.s3.use_aws_sdk_default_behavior":"false"}
# highlight-end
  启用: true
  注释:
1 row in set (0.03 sec)
```

:::note
在向存储桶写入数据之前，在MinIO对象列表中将看不到`shared`文件夹。
:::

---

## 创建一些表

<DDL />

---

## 载入两个数据集

有许多方法可以将数据加载到StarRocks中。对于本教程，最简单的方式是使用curl和StarRocks Stream Load。

:::tip

从您下载数据集的目录中的FE shell运行这些curl命令。

系统将提示您输入密码。您可能尚未为MySQL的`root`用户分配密码，因此只需按回车键。

:::

`curl`命令看起来很复杂，但在本教程的最后有详细说明。现在，我们建议您运行命令和运行一些SQL来分析数据，然后在最后阅读有关数据加载详细信息。

### 纽约市碰撞数据 - 碰撞

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

以下是上述命令的输出。第一部分显示了您应该期望看到的内容（OK以及除一行外的所有行都已插入）。有一行被过滤掉，因为它的列数不正确。

```bash
输入root 用户密码:
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

如果有错误，则输出提供了一个URL来查看错误消息。因为容器具有私有IP地址，所以您必须通过从容器运行curl来查看。

```bash
curl http://10.5.0.3:8040/api/_load_error_log<details from ErrorURL>
```

展开总结以查看在开发本教程时看到的内容：

<details>

<summary>在浏览器中阅读错误消息</summary>

```bash
错误：值计数与列计数不匹配。预期为29，但获得32。

列分隔符:44，换行符:10.. 行:09/06/2015,14:15,,,40.6722269,-74.0110059,"(40.6722269, -74.0110059)",,,"R/O 1 BEARD ST. ( IKEA'S 
09/14/2015,5:30,BRONX,10473,40.814551,-73.8490955,"(40.814551, -73.8490955)",TORRY AVENUE ,NORTON AVENUE ,,0,0,0,0,0,0,0,0,Driver Inattention/Distraction,Unspecified,,,,3297457,PASSENGER VEHICLE,PASSENGER VEHICLE,,,
```

</details>

### 天气数据

以与您加载碰撞数据相同的方式加载天气数据集。

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

## 验证数据是否存储在MinIO

打开MinIO [http://localhost:9001/browser/starrocks/](http://localhost:9001/browser/starrocks/) 并验证在`starrocks/shared/`下的每个目录中是否有`data`、`metadata`和`schema`条目。

:::提示
当你加载数据时，`starrocks/shared/`下的文件夹名称是动态生成的。你应该在`shared`下面看到一个单独的目录，然后在其中再看到两个目录。在每个目录中，你将找到数据、元数据和模式条目。

![MinIO对象浏览器](../assets/quick-start/MinIO-data.png)
:::

---

## 回答一些问题

<SQL />

---

## 配置StarRocks用于共享数据

现在你已经体验了使用共享数据的StarRocks，重要的是要了解配置。

### CN 配置

这里使用的CN配置是默认的，因为CN被设计为用于共享数据。默认配置如下所示。你无需进行任何更改。

```bash
sys_log_level = INFO

be_port = 9060
be_http_port = 8040
heartbeat_service_port = 9050
brpc_port = 8060
```

### FE 配置

FE配置与默认配置略有不同，因为FE必须配置为期望数据存储在对象存储而不是BE节点上的本地磁盘上。

`docker-compose.yml` 文件在`command`中生成了FE配置。

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

这导致生成了这个配置文件：

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

# highlight-start
run_mode=shared_data
aws_s3_path=starrocks
aws_s3_endpoint=minio:9000
aws_s3_use_instance_profile=false
cloud_native_storage_type=S3
aws_s3_use_aws_sdk_default_behavior=true
# highlight-end
```

:::注意
这个配置文件包含默认条目和共享数据的附加条目。共享数据的条目被标出。
:::

非默认的FE配置设置：

:::注意
许多配置参数以`s3_`为前缀。这个前缀适用于所有的Amazon S3兼容存储类型（例如：S3、GCS和MinIO）。当使用Azure Blob Storage时，这个前缀为`azure_`。
:::

#### `run_mode=shared_data`

这启用了共享数据的使用。

#### `aws_s3_path=starrocks`

存储桶名称。

#### `aws_s3_endpoint=minio:9000`

MinIO端点，包括端口号。

#### `aws_s3_use_instance_profile=false`

在使用MinIO时会使用访问密钥，因此不会使用实例配置文件。

#### `cloud_native_storage_type=S3`

这指定了是使用S3兼容存储还是Azure Blob Storage。对于MinIO，这总是S3。

#### `aws_s3_use_aws_sdk_default_behavior=true`

在使用MinIO时，这个参数始终设置为true。

---

## 总结

在本教程中，你：

- 部署了StarRocks和Minio在Docker中
- 创建了一个MinIO访问密钥
- 配置了一个使用MinIO的StarRocks存储卷
- 加载了纽约市提供的事故数据和美国国家海洋和大气管理局提供的天气数据
- 使用SQL JOIN分析数据，以找出在能见度低或道路结冰时开车是一个坏主意

还有更多需要学习；我们有意地忽略了流加载期间进行的数据转换。关于这个方面的详细信息，可以参考下面的curl命令注释。

## curl命令注释

<Curl />

## 更多信息

[StarRocks表设计](../table_design/StarRocks_table_design.md)

[物化视图](../cover_pages/mv_use_cases.mdx)

[流加载](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)

[机动车辆碰撞 - 撞车](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95) 数据集由纽约市提供，受到这些[使用条款](https://www.nyc.gov/home/terms-of-use.page)和[隐私政策](https://www.nyc.gov/home/privacy-policy.page)的约束。

[当地气象数据](https://www.ncdc.noaa.gov/cdo-web/datatools/lcd)(LCD)由美国国家海洋和大气管理局提供，带有这个[免责声明](https://www.noaa.gov/disclaimer)和这个[隐私政策](https://www.noaa.gov/protecting-your-privacy)。