---
description: 'StarRocks in Docker: Query real data with JOINs'
displayed_sidebar: English
sidebar_position: 1
---
import DDL from '../assets/quick-start/_DDL.mdx'
import Clients from '../assets/quick-start/_clientsAllin1.mdx'
import SQL from '../assets/quick-start/_SQL.mdx'
import Curl from '../assets/quick-start/_curl.mdx'

# StarRocksの基本

このチュートリアルでは、次の内容について説明します：

- 単一のDockerコンテナでStarRocksを実行する方法
- 基本的なデータ変換を含む2つの公開データセットのロード
- SELECTとJOINを使用したデータ分析
- 基本的なデータ変換（ETLの**T**）

使用されるデータは、NYC OpenDataとNational Centers for Environmental Informationによって提供されます。

これらのデータセットは非常に大きいため、このチュートリアルではStarRocksの使用に慣れることを目的としており、過去120年分のデータをロードすることはしません。Dockerに割り当てられた4GBのRAMを搭載したマシンでDockerイメージを実行し、このデータをロードできます。より大規模でフォールトトレラントなスケーラブルなデプロイメントには、他のドキュメントがあり、それは後ほど提供されます。

このドキュメントには多くの情報が含まれており、最初にステップバイステップの内容が、最後に技術的な詳細が提示されます。これは、以下の目的をこの順序で達成するためです：

1. 読者がStarRocksにデータをロードし、そのデータを分析することを可能にする。
2. データロード中の基本的なデータ変換について説明する。

---

## 前提条件

### Docker

- [Docker](https://docs.docker.com/engine/install/)
- Dockerに割り当てられた4GBのRAM
- Dockerに割り当てられた10GBの空きディスクスペース

### SQLクライアント

Docker環境内で提供されるSQLクライアントを使用することも、システム上で別のものを使用することもできます。多くのMySQL互換クライアントが動作しますが、このガイドではDBeaverとMySQL Workbenchの設定について説明します。

### curl

`curl`はStarRocksへのデータロードジョブの発行とデータセットのダウンロードに使用されます。OSプロンプトで`curl`または`curl.exe`を実行してインストールされているかどうかを確認してください。curlがインストールされていない場合は、[こちらからcurlを取得してください](https://curl.se/dlwiz/?type=bin)。

---
## 用語

### FE
フロントエンドノードはメタデータ管理、クライアント接続管理、クエリ計画、およびクエリスケジューリングを担当します。各FEはメタデータの完全なコピーをメモリ内に保持し、FE間での均等なサービスを保証します。

### BE
バックエンドノードはデータストレージとクエリプランの実行を担当します。

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

## データをダウンロードする

これら2つのデータセットをマシンにダウンロードします。Dockerを実行しているホストマシンにダウンロードすることも、コンテナ内にダウンロードする必要はありません。

### ニューヨーク市のクラッシュデータ

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/NYPD_Crash_Data.csv
```

### 気象データ

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/72505394728.csv
```

---

### SQLクライアントでStarRocksに接続

:::tip

mysql CLI以外のクライアントを使用している場合は、それを今開いてください。
:::

このコマンドはDockerコンテナ内で`mysql`コマンドを実行します：

```sql
docker exec -it quickstart \
mysql -P 9030 -h 127.0.0.1 -u root --prompt="StarRocks > "
```

---

## テーブルを作成する

<DDL />

---

## 2つのデータセットをロードする
StarRocksにデータをロードする方法は多数ありますが、このチュートリアルではcurlとStarRocks Stream Loadを使用するのが最も簡単です。

:::tip
これらのcurlコマンドはOSのプロンプトで実行されるため、`mysql`クライアントではなく新しいシェルを開いてください。コマンドはダウンロードしたデータセットを参照しているため、ファイルをダウンロードしたディレクトリで実行してください。

パスワードを求められた場合、MySQLの`root`ユーザーにパスワードを設定していない可能性が高いので、エンターキーを押してください。
:::

`curl`コマンドは複雑に見えるかもしれませんが、チュートリアルの最後で詳細に説明されます。今は、コマンドを実行してデータを分析するSQLを実行し、その後でデータロードの詳細について読むことをお勧めします。

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

上記のコマンドの出力は以下の通りです。最初にハイライトされたセクションが、期待される内容（OKとほぼ全ての行が挿入された状態）を示しています。正しい数の列を含まないため、1行がフィルタリングされました。

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

エラーが発生した場合、エラーメッセージを確認するためのURLが出力に含まれます。このURLをブラウザで開いて、何が起こったかを確認してください。詳細を展開してエラーメッセージを確認します。

<details>

<summary>ブラウザでのエラーメッセージの読み方</summary>

```bash
Error: Value count does not match column count. Expected 29, but got 32.

Column delimiter: 44,Row delimiter: 10.. Row: 09/06/2015,14:15,,,40.6722269,-74.0110059,"(40.6722269, -74.0110059)",,,"R/O 1 BEARD ST. (IKEA'S 
09/14/2015,5:30,BRONX,10473,40.814551,-73.8490955,"(40.814551, -73.8490955)",TORRY AVENUE,NORTON AVENUE,,0,0,0,0,0,0,0,0,Driver Inattention/Distraction,Unspecified,,,,3297457,PASSENGER VEHICLE,PASSENGER VEHICLE,,,
```

</details>

### 気象データ

気象データセットを、事故データをロードしたのと同じ方法でロードします。

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

## いくつかの質問に答える

<SQL />

---

## 概要

このチュートリアルでは、以下のことを行いました。

- DockerにStarRocksをデプロイしました
- ニューヨーク市が提供する事故データとNOAAが提供する気象データをロードしました
- SQL JOINを使用してデータを分析し、視界が低い状態や凍結した道路での運転は危険であることを発見しました

まだ学ぶべきことはたくさんあります。ストリームロード中に行われるデータ変換については意図的に詳細を省略しました。それについての詳細は、以下のcurlコマンドに関する注記で説明しています。

---

## curlコマンドに関する注記

<Curl />

---

## 詳細情報

[StarRocksテーブルデザイン](../table_design/StarRocks_table_design.md)

[マテリアライズド・ビュー](../cover_pages/mv_use_cases.mdx)

[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)

[Motor Vehicle Collisions - Crashes](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95) データセットは、これらの[利用規約](https://www.nyc.gov/home/terms-of-use.page)および[プライバシーポリシー](https://www.nyc.gov/home/privacy-policy.page)に基づいてニューヨーク市から提供されています。

[Local Climatological Data](https://www.ncdc.noaa.gov/cdo-web/datatools/lcd)(LCD)は、この[免責事項](https://www.noaa.gov/disclaimer)および[プライバシーポリシー](https://www.noaa.gov/protecting-your-privacy)をもってNOAAから提供されています。
