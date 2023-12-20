---
description: 'StarRocks in Docker: Query real data with JOINs'
displayed_sidebar: English
sidebar_position: 1
---

从 '../assets/quick-start/_DDL.mdx' 导入DDL，从 '../assets/quick-start/_clientsAllin1.mdx' 导入客户端，从 '../assets/quick-start/_SQL.mdx' 导入SQL，从 '../assets/quick-start/_curl.mdx' 导入Curl

# StarRocks基础

本教程包括以下内容：

- 在单个Docker容器中运行StarRocks
- 加载两个公共数据集，包括对数据进行基本转换
- 通过SELECT和JOIN分析数据
- 基础数据转换（ETL中的**T**）

所使用的数据由纽约市开放数据和国家环境信息中心提供。

由于这两个数据集的体积非常大，而本教程的目的是帮助您熟悉StarRocks的操作，我们不会加载过去120年的数据。您可以在为Docker分配了4GB RAM的机器上运行Docker镜像并加载这些数据。对于更大规模、具有容错性和可扩展性的部署，我们有其他文档，稍后会提供。

本文档包含大量信息，前面部分按步骤介绍内容，后面部分提供技术细节。这样安排是为了实现以下目的：

1. 使读者能够在StarRocks中加载数据并分析这些数据。
2. 解释在加载过程中进行数据转换的基础知识。


## 先决条件

### Docker

- [Docker](https://docs.docker.com/engine/install/)
- 分配给Docker的4GB RAM
- 分配给Docker的10GB可用磁盘空间

### SQL客户端

您可以使用Docker环境中提供的SQL客户端，也可以使用您系统上的客户端。许多兼容MySQL的客户端都可以使用，本指南包括配置DBeaver和MySQL Workbench的内容。

### curl

`curl`用于向StarRocks提交数据加载作业，并下载数据集。通过在操作系统提示符下运行`curl`或`curl.exe`来检查是否已安装它。如果尚未安装curl，请在[此处获取curl](https://curl.se/dlwiz/?type=bin)。

## 术语

### FE
前端节点负责元数据管理、客户端连接管理、查询计划和查询调度。每个FE在其内存中存储并维护一份元数据的完整副本，确保FE之间提供一致的服务。

### BE
后端节点负责数据存储和执行查询计划。


## 启动StarRocks

```bash
docker run -p 9030:9030 -p 8030:8030 -p 8040:8040 -itd \
--name quickstart starrocks/allin1-ubuntu
```

## SQL客户端

<Clients />



## 下载数据

将这两个数据集下载到您的机器上。您可以将它们下载到运行Docker的宿主机，无需下载到容器内。

### 纽约市碰撞数据

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/NYPD_Crash_Data.csv
```

### 天气数据

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/72505394728.csv
```


### 使用SQL客户端连接到StarRocks

:::提示

如果您使用的是除mysql命令行界面之外的客户端，请现在打开它。:::

以下命令将在Docker容器中运行mysql命令：

```sql
docker exec -it quickstart \
mysql -P 9030 -h 127.0.0.1 -u root --prompt="StarRocks > "
```


## 创建一些表

<DDL />



## 加载两个数据集
有许多方法可以将数据加载到StarRocks中。对于本教程，最简单的方法是使用curl和StarRocks的Stream Load功能。

:::提示在新的shell中打开，因为这些curl命令是在操作系统提示符下运行的，而不是在mysql客户端中运行的。命令引用了您下载的数据集，因此请在下载文件的目录中运行它们。

系统会提示您输入密码。您可能还没有为MySQL的root用户设置密码，所以直接按Enter键即可。:::

curl命令看起来可能很复杂，但它们在教程末尾有详细解释。现在，我们建议您运行这些命令并执行一些SQL来分析数据，然后在最后阅读有关数据加载的详细信息。

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

下面是前一个命令的输出结果。第一部分突出显示了您应该看到的内容（OK，除了一行外的所有行都已插入）。有一行因为列数不正确而被过滤掉了。

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

如果出现错误，输出会提供一个URL以查看错误信息。在浏览器中打开它以了解发生了什么。展开详细信息以查看错误信息：

<details>


<summary>Reading error messages in the browser</summary>


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
```


## 回答一些问题

<SQL />



## 总结

在本教程中，您：

- 在Docker中部署了StarRocks
- 加载了纽约市提供的碰撞数据和NOAA提供的天气数据
- 使用SQL JOIN分析数据，了解到在能见度低或路面结冰的情况下驾车是不明智的

还有更多内容需要学习；我们有意未详细介绍Stream Load过程中完成的数据转换。关于curl命令的详细说明，请参见下方的注释。


## curl命令的注释

<Curl />



## 更多信息

[StarRocks表设计](../table_design/StarRocks_table_design.md)

[物化视图](../cover_pages/mv_use_cases.mdx)

[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)

“[机动车辆碰撞 - 碰撞](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95)”数据集由纽约市提供，使用受这些[使用条款](https://www.nyc.gov/home/terms-of-use.page)和[隐私政策](https://www.nyc.gov/home/privacy-policy.page)的约束。

“[本地气候数据](https://www.ncdc.noaa.gov/cdo-web/datatools/lcd)”(LCD)由NOAA提供，适用这[免责声明](https://www.noaa.gov/disclaimer)和这[隐私政策](https://www.noaa.gov/protecting-your-privacy)。
