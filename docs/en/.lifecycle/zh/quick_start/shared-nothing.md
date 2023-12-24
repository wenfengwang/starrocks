---
description: 'StarRocks in Docker: Query real data with JOINs'
displayed_sidebar: English
sidebar_position: 1
---
import DDL from '../assets/quick-start/_DDL.mdx'
import Clients from '../assets/quick-start/_clientsAllin1.mdx'
import SQL from '../assets/quick-start/_SQL.mdx'
import Curl from '../assets/quick-start/_curl.mdx'

# StarRocks 基础知识

本教程涵盖内容如下：

- 在单个 Docker 容器中运行 StarRocks
- 加载两个公共数据集，包括对数据的基本转换
- 使用 SELECT 和 JOIN 分析数据
- 数据的基本转换（ETL 中的 T）

所使用的数据由纽约市开放数据和国家环境信息中心提供。

这两个数据集都非常庞大，因为本教程旨在帮助您熟悉 StarRocks 的使用，所以我们不会加载过去 120 年的数据。您可以在分配了 4 GB RAM 的 Docker 容器上的计算机上运行 Docker 镜像并加载这些数据。对于更大的容错和可扩展部署，我们有其他文档，并将在稍后提供。

本文档包含大量信息，开头是逐步内容，结尾是技术细节。这样做是为了按以下顺序达到这些目的：

1. 允许读者在 StarRocks 中加载数据并分析该数据。
2. 解释加载期间的数据转换基础知识。

---

## 先决条件

### Docker

- [Docker](https://docs.docker.com/engine/install/)
- 为 Docker 分配 4 GB RAM
- 为 Docker 分配 10 GB 可用磁盘空间

### SQL 客户端

您可以使用 Docker 环境中提供的 SQL 客户端，也可以使用系统上的客户端。许多兼容 MySQL 的客户端都可以使用，本指南涵盖了 DBeaver 和 MySQL WorkBench 的配置。

### curl

`curl` 用于向 StarRocks 发送数据加载作业和下载数据集。通过在操作系统提示符下运行 `curl` 或 `curl.exe` 来检查是否已安装。如果未安装 curl，[请在此处获取 curl](https://curl.se/dlwiz/?type=bin)。

---
## 术语

### FE
前端节点负责元数据管理、客户端连接管理、查询规划和查询调度。每个 FE 在其内存中存储并维护完整的元数据副本，从而保证了 FE 之间的无差别服务。

### BE
后端节点负责数据存储和执行查询计划。

---

## 启动 StarRocks

```bash
docker run -p 9030:9030 -p 8030:8030 -p 8040:8040 -itd \
--name quickstart starrocks/allin1-ubuntu
```
---
## SQL 客户端

<Clients />

---

## 下载数据

将这两个数据集下载到您的计算机上。您可以将它们下载到运行 Docker 的主机上，它们不需要下载到容器内。

### 纽约市车祸数据

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/NYPD_Crash_Data.csv
```

### 天气数据

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/72505394728.csv
```

---

### 使用 SQL 客户端连接 StarRocks

:::提示

如果您使用的是 mysql CLI 之外的客户端，请立即打开该客户端。
:::

此命令将在 Docker 容器中运行 `mysql` 命令：

```sql
docker exec -it quickstart \
mysql -P 9030 -h 127.0.0.1 -u root --prompt="StarRocks > "
```

---

## 创建一些表

<DDL />

---

## 加载两个数据集

有许多种方法可以将数据加载到 StarRocks 中。在本教程中，最简单的方法是使用 curl 和 StarRocks Stream Load。

:::提示

打开一个新的 shell，因为这些 curl 命令是在操作系统提示符下运行的，而不是在 `mysql` 客户端中运行的。这些命令引用您下载的文件，因此请从下载文件的目录运行它们。

系统将提示您输入密码。您可能尚未为 MySQL 的 `root` 用户分配密码，因此只需按回车键即可。
:::

这些 `curl` 命令看起来很复杂，但在本教程的末尾进行了详细解释。现在，我们建议运行这些命令并运行一些 SQL 来分析数据，然后在最后阅读有关数据加载的详细信息。

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

以下是上述命令的输出。第一个突出显示的部分显示了您应该看到的内容（已确定并插入了除一行之外的所有内容）。有一行被筛选掉，因为它不包含正确的列数。

```bash

输入主机密码以供用户'root'使用：
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
    "ErrorURL": "http://127.0.0.1:8040/api/_load_error_log?file=error_log_da41dd88276a7bfc_739087c94262ae9f"
    # highlight-end
}%
```

如果出现错误，输出将提供一个 URL 以查看错误消息。在浏览器中打开此 URL 以了解发生了什么。展开详细信息以查看错误消息：

<details>

<summary>在浏览器中阅读错误消息</summary>

```bash
错误：值计数与列计数不匹配。期望 29，但实际得到 32。

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

## 回答一些问题

<SQL />

---

## 总结

在本教程中，您：

- 在 Docker 中部署了 StarRocks
- 加载了纽约市提供的碰撞数据和 NOAA 提供的天气数据
- 使用 SQL JOIN 进行数据分析，发现在能见度低或结冰的街道上驾驶是个坏主意

还有更多内容可以学习；我们有意地略过了流加载期间所做的数据转换。有关此内容的详细信息，请参阅下面有关 curl 命令的注释。

---

## curl 命令的注释

<Curl />

---

## 更多信息

[StarRocks 表设计](../table_design/StarRocks_table_design.md)

[物化视图](../cover_pages/mv_use_cases.mdx)

[流加载](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)

[机动车碰撞 - 碰撞](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95) 数据集由纽约市提供，受到这些[使用条款](https://www.nyc.gov/home/terms-of-use.page)和[隐私政策](https://www.nyc.gov/home/privacy-policy.page)的约束。

[当地气候数据](https://www.ncdc.noaa.gov/cdo-web/datatools/lcd)（LCD）由美国国家海洋和大气管理局（NOAA）提供，并附有此[免责声明](https://www.noaa.gov/disclaimer)和此[隐私政策](https://www.noaa.gov/protecting-your-privacy)。