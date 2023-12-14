---
displayed_sidebar: "Chinese"
sidebar_position: 1
description: "在Docker中使用StarRocks：使用JOIN查询实际数据"
---
import DDL from '../assets/quick-start/_DDL.mdx'
import Clients from '../assets/quick-start/_clientsAllin1.mdx'
import SQL from '../assets/quick-start/_SQL.mdx'
import Curl from '../assets/quick-start/_curl.mdx'

# StarRocks基础知识

此教程涵盖：

- 在单个Docker容器中运行StarRocks
- 加载两个公共数据集，包括对数据的基本转换
- 使用SELECT和JOIN分析数据
- 基本数据转换（ETL中的T）

所使用的数据由纽约市开放数据和国家环境信息中心提供。

这两个数据集都非常庞大，由于本教程旨在帮助您熟悉使用StarRocks，我们不会加载过去120年的数据。您可以在分配了4GB RAM的Docker机器上运行Docker镜像并加载这些数据。对于更大的容错和可伸缩部署，我们有其他文档，并将在后续提供。

本文档包含大量信息，首先按照逐步内容，最后是技术细节呈现。这样做的目的是按照以下顺序为这些目的服务：

1. 允许读者加载StarRocks中的数据并分析该数据。
2. 解释加载过程中的数据转换基础知识。

---

## 先决条件

### Docker

- [Docker](https://docs.docker.com/engine/install/)
- 为Docker分配4GB RAM
- 为Docker分配10GB免费磁盘空间

### SQL客户端

您可以使用Docker环境中提供的SQL客户端，也可以使用系统上的任何一个。许多兼容MySQL的客户端都可以工作，本指南涵盖了DBeaver和MySQL WorkBench的配置。

### curl

`curl` 用于向StarRocks发出数据加载作业以及下载数据集。通过在操作系统提示符处运行 `curl` 或 `curl.exe` 来检查是否已安装curl。如果未安装curl，[在此获取curl](https://curl.se/dlwiz/?type=bin)。

---
## 术语

### FE
前端节点负责元数据管理、客户端连接管理、查询计划和查询调度。每个FE在其内存中存储和维护完整的元数据副本，从而保证在FEs之间提供无差别的服务。

### BE
后端节点负责数据存储和执行查询计划。

---

## 启动StarRocks

```bash
docker run -p 9030:9030 -p 8030:8030 -p 8040:8040 -itd \
--name quickstart starrocks/allin1-ubuntu
```
---
## SQL客户端

<Clients />

---

## 下载数据

将这两个数据集下载到您的计算机。您可以将它们下载到运行Docker的主机计算机上，它们不需要在容器内部下载。

### 纽约市撞车数据

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/NYPD_Crash_Data.csv
```

### 天气数据

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/72505394728.csv
```

---

### 使用SQL客户端连接到StarRocks

:::提示

如果您使用的不是mysql CLI客户端，请打开它。
:::

此命令会在Docker容器中运行 `mysql` 命令：

```sql
docker exec -it quickstart \
mysql -P 9030 -h 127.0.0.1 -u root --prompt="StarRocks > "
```

---

## 创建一些表

<DDL />

---

## 加载两个数据集
有许多方法可以将数据加载到StarRocks中。对于本教程，最简单的方法是使用curl和StarRocks Stream Load。

:::提示
打开一个新的shell，因为这些curl命令是在操作系统提示符下运行的，而不是在 `mysql` 客户端中。命令引用您下载文件的目录，因此请从下载文件的目录中运行这些命令。

您将被提示输入密码。您可能还没有为MySQL `root` 用户分配密码，所以只需按Enter键。
:::

`curl` 命令看起来复杂，但它们在本教程的最后有详细说明。现在，我们建议运行这些命令并运行一些SQL来分析数据，然后在最后阅读有关数据加载细节的内容。

### 纽约市碰撞数据 - 撞车

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

以下是前述命令的输出。首先突出显示的部分显示了您应该期望看到的内容（OK和除一个行以外的所有行已插入）。有一行被过滤掉，因为它包含了不正确数量的列。

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
    "ErrorURL": "http://127.0.0.1:8040/api/_load_error_log?file=error_log_da41dd88276a7bfc_739087c94262ae9f"
    # highlight-end
}%
```

如果出现错误，输出中将提供一个URL以查看错误消息。在浏览器中打开此URL以查找出现的问题。展开详细信息以查看错误消息：

<details>

<summary>在浏览器中阅读错误消息</summary>

```bash
Error: Value count does not match column count. Expect 29, but got 32.

Column delimiter: 44,Row delimiter: 10.. Row: 09/06/2015,14:15,,,40.6722269,-74.0110059,"(40.6722269, -74.0110059)",,,"R/O 1 BEARD ST. ( IKEA'S 
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


---

## 回答一些问题

<SQL />

---

## 摘要

在本教程中：

- 部署了Docker中的StarRocks
- 加载了由纽约市提供的碰撞数据以及由NOAA提供的天气数据
- 使用SQL JOIN分析数据，以找出在能见度低或结冰街道上行驶是个坏主意

还有更多要学习的；我们故意忽略了流式加载期间所做的数据转换。有关详情，请参阅以下curl命令的说明。

---

## Curl命令的附加说明

<Curl />

---

## 更多信息

[StarRocks表设计](../table_design/StarRocks_table_design.md)

[物化视图](../cover_pages/mv_use_cases.mdx)

[流式加载](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)

[机动车辆碰撞-碰撞](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95) 数据集由纽约市提供，受到这些[使用条款](https://www.nyc.gov/home/terms-of-use.page)和[隐私政策](https://www.nyc.gov/home/privacy-policy.page)的约束。

局部气候数据(LCD)由NOAA提供，受到这个[免责声明](https://www.noaa.gov/disclaimer)和这个[隐私政策](https://www.noaa.gov/protecting-your-privacy)的约束。