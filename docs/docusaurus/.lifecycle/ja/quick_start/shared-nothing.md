---
displayed_sidebar: "日本語"
sidebar_position: 1
description: "StarRocks in Docker: JOINを使用して実際のデータをクエリ"
---
import DDL from '../assets/quick-start/_DDL.mdx'
import Clients from '../assets/quick-start/_clientsAllin1.mdx'
import SQL from '../assets/quick-start/_SQL.mdx'
import Curl from '../assets/quick-start/_curl.mdx'

# StarRocks 基礎

このチュートリアルでは以下の内容をカバーします：

- 単一のDockerコンテナでStarRocksを実行
- 基本的なデータのロード
- SELECTとJOINを使用したデータの解析
- 基本的なデータ変換（ETLの**T**）

使用されるデータはNYC OpenDataとNational Centers for Environmental Informationによって提供されています。

これらのデータセットは非常に大きいため、このチュートリアルでは過去120年間のデータをロードすることはしません。Dockerに割り当てられた4GBのRAMを持つマシンでDockerイメージを実行し、このデータをロードすることができます。耐障害性とスケーラブルな展開については、後で別の文書を提供します。

この文書には多くの情報が含まれており、最初にステップバイステップのコンテンツと最後に技術的な詳細を提示しています。これは次の目的を達成するためにこの順序で行われています：

1. StarRocksでデータをロードし、そのデータを解析するための読者への許可
2. データのロード中の基本的なデータ変換を説明

---

## 前提条件

### Docker

- [Docker](https://docs.docker.com/engine/install/)
- Dockerに割り当てられた4GBのRAM
- Dockerに割り当てられた10GBの空きディスク領域

### SQLクライアント

Docker環境で提供されているSQLクライアントを使用するか、システム上で使用しても構いません。多くのMySQL互換クライアントが動作し、このガイドではDBeaverとMySQL WorkBenchの構成について説明します。

### curl

`curl`はデータロードジョブをStarRocksに発行したり、データセットをダウンロードするために使用されます。OSプロンプトで`curl`または`curl.exe`を実行してインストールされているかどうかを確認してください。curlがインストールされていない場合は、[こちらからcurlを入手してください](https://curl.se/dlwiz/?type=bin)。

---
## 用語

### FE
フロントエンドノードは、メタデータ管理、クライアント接続管理、クエリの計画、およびクエリのスケジューリングに責任があります。各FEはそのメモリにメタデータの完全なコピーを保存および維持し、これによりFE間での無差別なサービスが保証されます。

### BE
バックエンドノードはデータの保存とクエリプランの実行に責任があります。

---

## StarRocksを起動

```bash
docker run -p 9030:9030 -p 8030:8030 -p 8040:8040 -itd \
--name quickstart starrocks/allin1-ubuntu
```
---
## SQLクライアント

<Clients />

---

## データをダウンロード

これらの2つのデータセットをマシンにダウンロードしてください。Dockerを実行しているホストマシンにダウンロードしても問題ありません。コンテナ内にダウンロードする必要はありません。

### ニューヨーク市のクラッシュデータ

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/NYPD_Crash_Data.csv
```

### 天気データ

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/72505394728.csv
```

---

### SQLクライアントでStarRocksに接続

:::tip

mysql CLI以外のクライアントを使用している場合は、そのクライアントを開いてください。
:::

このコマンドはDockerコンテナ内で`mysql`コマンドを実行します：

```sql
docker exec -it quickstart \
mysql -P 9030 -h 127.0.0.1 -u root --prompt="StarRocks > "
```

---

## いくつかのテーブルを作成

<DDL />

---

## 2つのデータセットをロード

StarRocksにデータをロードする方法はいくつかあります。このチュートリアルでは、最も簡単な方法としてcurlとStarRocks Stream Loadを使用します。

:::tip
これらのcurlコマンドは`mysql`クライアントではなく、オペレーティングシステムのプロンプトで実行されるため、新しいシェルを開いてください。これらのコマンドはダウンロードしたディレクトリから実行するため、ダウンロードしたファイルのディレクトリで実行してください。

パスワードが求められます。おそらくMySQL `root`ユーザにパスワードを割り当てていないため、ただEnterキーを押してください。
:::

これらの`curl`コマンドは複雑に見えますが、この後のチュートリアルで詳しく説明します。今のところは、コマンドを実行し、データを解析するためのいくつかのSQLを実行し、最後にデータのロードの詳細を読むことを推奨します。

### ニューヨーク市の衝突データ - クラッシュ

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

直前のコマンドの出力は次の通りです。最初のハイライトされたセクションは、予想される出力です（OKとほぼ一行が挿入されました）。行が正しい列数を持たないため、1行がフィルタリングされました。

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

エラーがある場合、出力にエラーメッセージを確認するためのURLが提供されます。ブラウザでこれを開き、何が起こったかを確認してください。エラーメッセージを表示するには詳細を展開してください。

<details>

<summary>ブラウザでのエラーメッセージの読み取り</summary>

```bash
Error: Value count does not match column count. Expect 29, but got 32.

Column delimiter: 44,Row delimiter: 10.. Row: 09/06/2015,14:15,,,40.6722269,-74.0110059,"(40.6722269, -74.0110059)",,,"R/O 1 BEARD ST. ( IKEA'S 
09/14/2015,5:30,BRONX,10473,40.814551,-73.8490955,"(40.814551, -73.8490955)",TORRY AVENUE                    ,NORTON AVENUE                   ,,0,0,0,0,0,0,0,0,Driver Inattention/Distraction,Unspecified,,,,3297457,PASSENGER VEHICLE,PASSENGER VEHICLE,,,
```

</details>

### 天気データ

クラッシュデータをロードしたのと同じ方法で天気データをロードします。

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