---
description: StarRocks in Docker：使用 JOIN 查询真实数据
displayed_sidebar: English
sidebar_position: 1
---
从'../assets/quick-start/_DDL.mdx'导入DDL
从'../assets/quick-start/_clientsAllin1.mdx'导入Clients
从'../assets/quick-start/_SQL.mdx'导入SQL
从'../assets/quick-start/_curl.mdx'导入Curl

# StarRocks 基础

本教程包括：

- 在单个 Docker 容器中运行 StarRocks
- 加载两个公共数据集，包括对数据的基本转换
- 使用 SELECT 和 JOIN 分析数据
- 基本数据转换（ETL中的**T**）

使用的数据由 NYC OpenData 和国家环境信息中心提供。

这两个数据集都非常大，由于本教程的目的是帮助您熟悉如何使用 StarRocks，我们不会加载过去120年的数据。您可以在分配了4GB RAM给Docker的机器上运行Docker镜像并加载这些数据。对于需要更大的容错能力和可扩展性的部署，我们有其他的文档，并将在稍后提供。

本文档包含大量信息，内容的安排是先从逐步的介绍开始，然后在文末提供技术细节。这样的安排是为了按照以下顺序服务于这些目的：

1. 允许用户在 StarRocks 中加载数据并分析这些数据。
2. 解释在加载期间数据转换的基础。

---

## 先决条件

### Docker

- [Docker](https://docs.docker.com/engine/install/)
- 分配给 Docker 的 4 GB RAM
- 分配给 Docker 的 10 GB 免费磁盘空间

### SQL 客户端

您可以使用 Docker 环境中提供的 SQL 客户端，也可以使用您系统上的客户端。许多兼容 MySQL 的客户端都可以工作，本指南涵盖了 DBeaver 和 MySQL Workbench 的配置。

### Curl

`curl` 用于向 StarRocks 发起数据加载作业，并下载数据集。通过在操作系统提示符下运行 `curl` 或 `curl.exe` 来检查您是否已安装它。如果未安装 curl，[请在此处获取 curl](https://curl.se/dlwiz/?type=bin)。

---
## 术语

### 铁
前端节点负责元数据管理、客户端连接管理、查询规划和查询调度。每个FE都在其内存中存储并维护一份完整的元数据副本，这保证了FE之间提供无差别的服务。

### 是
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

将这两个数据集下载到您的计算机上。您可以将它们下载到运行Docker的宿主机上，不需要下载到容器内。

### 纽约市交通事故数据

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/NYPD_Crash_Data.csv
```

### 天气数据

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/72505394728.csv
```

---

### 使用 SQL 客户端连接到 StarRocks

:::tip

如果您使用的是 mysql CLI 以外的客户端，请现在打开它。
:::

此命令将在 Docker 容器中运行 `mysql` 命令：

```sql
docker exec -it quickstart \
mysql -P 9030 -h 127.0.0.1 -u root --prompt="StarRocks > "
```

---

## 创建一些表格

<DDL />

---

## 加载两个数据集
有许多方法可以将数据加载到 StarRocks 中。对于本教程，最简单的方式是使用 curl 和 StarRocks Stream Load。

:::tip
打开一个新的 shell，因为这些 curl 命令是在操作系统提示符下运行的，而不是在 `mysql` 客户端中运行的。这些命令引用您下载的数据集，因此请从您下载文件的目录中运行它们。

系统将提示您输入密码。您可能还没有为 MySQL 的 `root` 用户设置密码，所以只需按回车键即可。
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

这是前一个命令的输出结果。第一个高亮部分展示了您应该期望看到的内容（显示OK并且除了一行之外的所有行都被插入了）。有一行因为列数不正确而被过滤掉了。

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

如果出现错误，输出会提供一个 URL 以查看错误信息。在浏览器中打开此链接以了解发生了什么。展开详情以查看错误消息：

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

## 回答一些问题

<SQL />

---

## 总结

在本教程中，你将：

- 在 Docker 中部署 StarRocks
- 由纽约市提供的事故数据和由NOAA提供的天气数据
- 使用 SQL JOINs 分析数据，发现在能见度低或有冰的街道上驾驶是个坏主意

还有更多东西需要学习；我们故意忽略了在流加载期间完成的数据转换。关于这方面的详细信息，请参阅下面关于 curl 命令的注释。

---

## 关于 curl 命令的笔记

<Curl />

---

## 更多信息

[StarRocks 表设计](../table_design/StarRocks_table_design.md)

[物化视图](../cover_pages/mv_use_cases.mdx)

[流式加载](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)

[机动车辆碰撞 - 事故](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95) 数据集由纽约市提供，使用须遵守这些[使用条款](https://www.nyc.gov/home/terms-of-use.page)和[隐私政策](https://www.nyc.gov/home/privacy-policy.page)。

[本地气候数据](https://www.ncdc.noaa.gov/cdo-web/datatools/lcd)（LCD）由NOAA提供，附带此[免责声明](https://www.noaa.gov/disclaimer)和此[隐私政策](https://www.noaa.gov/protecting-your-privacy)。