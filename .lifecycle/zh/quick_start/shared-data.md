---
description: Separate compute and storage
displayed_sidebar: English
sidebar_position: 2
---

# 存储与计算分离
从'../assets/quick-start/_DDL.mdx'导入DDL，从'../assets/quick-start/_clientsCompose.mdx'导入客户端，从'../assets/quick-start/_SQL.mdx'导入SQL，从'../assets/quick-start/_curl.mdx'导入Curl。

在分离存储与计算的系统中，数据存储在成本低廉且可靠的远程存储系统中，例如Amazon S3、Google Cloud Storage、Azure Blob Storage，以及其他兼容S3的存储如MinIO。热数据被本地缓存，当缓存命中时，查询性能可与存储计算耦合架构媲美。计算节点（CN）可以在几秒内按需增加或移除。这种架构降低了存储成本，确保了更好的资源隔离，并提供了弹性和可扩展性。

本教程包括：

- 在Docker容器中运行StarRocks
- 使用MinIO作为对象存储
- 为共享数据配置StarRocks
- 加载两个公开数据集
- 使用SELECT和JOIN分析数据
- 基础数据转换（ETL中的**T**）

所使用的数据由NYC OpenData和NOAA国家环境信息中心提供。

这两个数据集都非常庞大，鉴于本教程旨在帮助您熟悉StarRocks的操作，我们不会加载过去120年的数据。您可以在为Docker分配了4GB RAM的机器上运行Docker镜像并加载这些数据。对于更大规模的容错和可扩展部署，我们有其他文档，稍后将提供。

本文档包含大量信息，内容分为开头的分步指南和结尾的技术细节。这样安排是为了按此顺序实现以下目的：

1. 允许读者在共享数据部署中加载数据并分析这些数据。
2. 提供共享数据部署的配置细节。
3. 解释在加载过程中进行的数据转换基础。


## 先决条件

### Docker

- [Docker](https://docs.docker.com/engine/install/)
- 为Docker分配的4GB RAM
- 为Docker分配的10GB空闲磁盘空间

### SQL客户端

您可以使用Docker环境中提供的SQL客户端，或者在您的系统上使用SQL客户端。许多兼容MySQL的客户端都可以工作，本指南涵盖了DBeaver和MySQL Workbench的配置。

### curl

`curl`用于向StarRocks提交数据加载作业，并下载数据集。通过在操作系统提示符下运行`curl`或`curl.exe`检查是否已安装curl。如果curl未安装，请在这里获取[curl](https://curl.se/dlwiz/?type=bin)。


## 术语

### FE
前端节点负责元数据管理、客户端连接管理、查询规划和查询调度。每个FE都在其内存中存储并维护一份元数据的完整副本，这保证了FE之间的服务无差别。

### CN
计算节点负责执行共享数据部署中的查询计划。

### BE
后端节点负责无共享部署中的数据存储和执行查询计划。

:::注意本指南不使用BE节点，此信息仅用于帮助您了解BE和CN之间的区别。:::


## 启动StarRocks

要运行使用对象存储的共享数据StarRocks，我们需要：

- 一个前端引擎（FE）
- 一个计算节点（CN）
- 对象存储

本指南使用的是MinIO，这是一种兼容S3的对象存储，基于GNU Affero通用公共许可证提供。

为了提供一个包含所需三个容器的环境，StarRocks提供了一个Docker Compose文件。

```bash
mkdir quickstart
cd quickstart
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/docker-compose.yml
```

```bash
docker compose up -d
```

检查服务的进度。FE和CN大约需要30秒时间才能变为健康状态。MinIO容器不会显示健康指示器，但您将使用MinIO的Web UI，这将验证其健康状态。

运行docker compose ps，直到FE和CN显示为健康状态：

```bash
docker compose ps
```

```plaintext
SERVICE        CREATED          STATUS                    PORTS
starrocks-cn   25 seconds ago   Up 24 seconds (healthy)   0.0.0.0:8040->8040/tcp
starrocks-fe   25 seconds ago   Up 24 seconds (healthy)   0.0.0.0:8030->8030/tcp, 0.0.0.0:9020->9020/tcp, 0.0.0.0:9030->9030/tcp
minio          25 seconds ago   Up 24 seconds             0.0.0.0:9000-9001->9000-9001/tcp
```


## 生成MinIO凭证

为了使用MinIO作为StarRocks的对象存储，您需要生成一个**访问密钥**。

### 打开MinIO的Web UI

浏览至[http://localhost:9001/access-keys](http://localhost:9001/access-keys)。用户名和密码在Docker Compose文件中指定，分别为`minioadmin`和`minioadmin`。您应该看到还没有访问密钥。点击**创建访问密钥 +**。

MinIO将生成一个密钥，点击 **创建** 并下载密钥。

![Make sure to click create](../assets/quick-start/MinIO-create.png)

:::注意访问密钥在您点击**创建**之前不会被保存，请不要仅仅复制密钥然后离开页面:::


## SQL客户端

<Clients />



## 下载数据

将这两个数据集下载到您的FE容器中。

### 在FE容器上打开一个终端

打开终端并为下载的文件创建一个目录：

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

### 气象数据

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/72505394728.csv
```


## 为共享数据配置StarRocks

此时，您已经运行了StarRocks和MinIO。MinIO访问密钥用于连接StarRocks和MinIO。

### 使用SQL客户端连接到StarRocks

:::提示

从包含docker-compose.yml文件的目录中运行此命令。

如果您使用的是MySQL CLI以外的客户端，请现在打开它。:::

```sql
docker compose exec starrocks-fe \
mysql -P9030 -h127.0.0.1 -uroot --prompt="StarRocks > "
```

### 创建一个存储卷

下面展示的是配置的详细信息：

- MinIO服务器可以通过URL http://minio:9000 访问
- 上面创建的存储桶命名为starrocks
- 写入此卷的数据将被存储在名为shared的文件夹中，位于存储桶starrocks内:::提示当第一次向该卷写入数据时，将创建名为shared的文件夹。:::
- MinIO服务器未使用SSL
- 输入MinIO的密钥和秘密，分别为aws.s3.access_key和aws.s3.secret_key。使用您之前在MinIO Web UI中创建的访问密钥。
- 共享卷是默认卷

:::提示在运行命令前，请编辑命令，并用您在MinIO中创建的访问密钥和秘密替换掉高亮显示的访问密钥信息。:::

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

:::提示本文档中的一些SQL，以及StarRocks文档中的许多其他文档，都使用\G代替了分号。 \G使得mysql CLI能够垂直渲染查询结果。

许多SQL客户端不解释垂直格式输出，因此您应该用分号替换\G。:::

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

:::注意在数据写入存储桶之前，名为shared的文件夹在MinIO对象列表中不可见。:::


## 创建一些表格

<DDL />



## 加载两个数据集

有多种方法可以将数据加载到StarRocks中。对于本教程，最简单的方法是使用curl和StarRocks Stream Load。

:::提示

在FE容器中下载数据集的目录里运行这些curl命令。

系统会提示您输入密码。您可能还没有为MySQL root用户设置密码，所以直接按回车键即可。

:::

curl命令看似复杂，但在教程的最后会有详细解释。现在，我们建议您运行这些命令并执行一些SQL语句来分析数据，然后再阅读教程末尾关于数据加载的详细信息。

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

以下是上述命令的输出。第一个高亮的部分显示了您应该期望看到的结果（OK，除了一行以外的所有行都插入了）。一行被过滤掉了，因为它的列数不正确。

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

如果有错误，输出会提供一个URL来查看错误信息。因为容器有一个私有IP地址，所以您需要在容器内运行curl来查看它。

```bash
curl http://10.5.0.3:8040/api/_load_error_log<details from ErrorURL>
```

展开摘要，查看开发本教程时所见的内容：

<details>


<summary>Reading error messages in the browser</summary>


```bash
Error: Value count does not match column count. Expect 29, but got 32.

Column delimiter: 44,Row delimiter: 10.. Row: 09/06/2015,14:15,,,40.6722269,-74.0110059,"(40.6722269, -74.0110059)",,,"R/O 1 BEARD ST. ( IKEA'S 
09/14/2015,5:30,BRONX,10473,40.814551,-73.8490955,"(40.814551, -73.8490955)",TORRY AVENUE                    ,NORTON AVENUE                   ,,0,0,0,0,0,0,0,0,Driver Inattention/Distraction,Unspecified,,,,3297457,PASSENGER VEHICLE,PASSENGER VEHICLE,,,
```

</details>


### 气象数据

以与加载碰撞数据相同的方式加载气象数据集。

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


## 验证数据是否已经存储在MinIO中

打开MinIO [http://localhost:9001/browser/starrocks/](http://localhost:9001/browser/starrocks/)并确认您在`starrocks/shared/`下的每个目录中都有`数据`、`元数据`和`模式`条目

:::提示starrocks/shared/下的文件夹名称在加载数据时生成。您应该在shared下面看到一个目录，然后在其下面看到另外两个目录。在每个目录中，您都会找到数据、元数据和模式条目。

![MinIO对象浏览器](../assets/quick-start/MinIO-data.png)
:::


## 回答一些问题

<SQL />



## 为共享数据配置StarRocks

现在您已经体验了使用StarRocks和共享数据，了解配置是很重要的。

### CN配置

这里使用的CN配置是默认的，因为CN是为共享数据使用而设计的。默认配置如下所示。您无需进行任何更改。

```bash
sys_log_level = INFO

be_port = 9060
be_http_port = 8040
heartbeat_service_port = 9050
brpc_port = 8060
```

### FE配置

FE配置与默认配置稍有不同，因为FE必须配置为期望数据存储在对象存储中，而不是存储在BE节点的本地磁盘上。

docker-compose.yml文件在命令中生成FE配置。

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

:::注意此配置文件包含默认条目和共享数据的附加内容。共享数据的条目已经突出显示。:::

非默认FE配置设置：

:::注意许多配置参数都以s3_为前缀。这个前缀用于所有Amazon S3兼容的存储类型（例如：S3、GCS和MinIO）。使用Azure Blob存储时，前缀为azure_.:::

#### run_mode=shared_data

这启用了共享数据的使用。

#### aws_s3_path=starrocks

存储桶的名称。

#### aws_s3_endpoint=minio:9000

包括端口号的MinIO端点。

#### aws_s3_use_instance_profile=false

使用MinIO时，会使用访问密钥，所以不使用实例配置文件。

#### cloud_native_storage_type=S3

这指定了是使用S3兼容存储还是Azure Blob存储。对于MinIO，这总是S3。

#### aws_s3_use_aws_sdk_default_behavior=true

使用MinIO时，此参数总是设置为true。


## 总结

在本教程中，您：

- 在Docker中部署了StarRocks和MinIO
- 创建了MinIO访问密钥
- 配置了使用MinIO的StarRocks存储卷
- 加载了由纽约市提供的碰撞数据和NOAA提供的气象数据
- 使用SQL JOIN分析数据，发现在能见度低或结冰的街道上驾驶是不明智的

还有更多内容需要学习；我们故意略过了流加载过程中完成的数据转换。有关详细信息，请参阅下面的curl命令说明。

## curl命令的说明

<Curl />


## 更多信息

[StarRocks表设计](../table_design/StarRocks_table_design.md)

[物化视图](../cover_pages/mv_use_cases.mdx)

[流加载](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)

这个[机动车辆碰撞-碰撞](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95)数据集是由纽约市提供的，受这些[使用条款](https://www.nyc.gov/home/terms-of-use.page)和[隐私政策](https://www.nyc.gov/home/privacy-policy.page)约束。

本地气候数据（[LCD](https://www.ncdc.noaa.gov/cdo-web/datatools/lcd)）由NOAA提供，并带有此[免责声明](https://www.noaa.gov/disclaimer)和此[隐私政策](https://www.noaa.gov/protecting-your-privacy)。
