---
displayed_sidebar: "Japanese"
sidebar_position: 1
description: "DockerでStarRocks: JOINを使用してリアルデータをクエリする"
---
import DDL from '../assets/quick-start/_DDL.mdx'
import Clients from '../assets/quick-start/_clientsAllin1.mdx'
import SQL from '../assets/quick-start/_SQL.mdx'
import Curl from '../assets/quick-start/_curl.mdx'

# StarRocksの基本

このチュートリアルでは以下をカバーしています：

- 単一のDockerコンテナでStarRocksを実行する方法
- インプットデータを含む2つの公開データセットをロードする方法、データの基本的な変換
- SELECTとJOINを使用してデータを解析する方法
- 基本的なデータ変換 (ETLの'T')

使用するデータは、NYC OpenDataとNational Centers for Environmental Informationが提供しています。

これらのデータセットは非常に大きいため、このチュートリアルでは過去120年間のデータをロードすることは行いません。Dockerイメージを実行し、このデータを4 GBのRAMが割り当てられたマシンにロードできます。より大規模で信頼性のある展開については、後で他の文書を提供します。

この文書には多くの情報が含まれており、最初にステップバイステップのコンテンツで、最後に技術的な詳細が提示されています。これは以下の目的をこの順番で達成するために行われています：

1. 読者にStarRocksでデータをロードし、そのデータを解析することを許可する。
2. ロード中のデータ変換の基本を説明する。

---

## 前提条件

### Docker

- [Docker](https://docs.docker.com/engine/install/)
- Dockerに割り当てられた4 GBのRAM
- Dockerに割り当てられた10 GBの空きディスク容量

### SQLクライアント

Docker環境で提供されているSQLクライアントを使用するか、システム上のクライアントを使用できます。多くのMySQL互換のクライアントが動作するため、このガイドではDBeaverとMySQL WorkBenchの構成について説明します。

### curl

`curl`は、StarRocksにデータロードジョブを発行し、データセットをダウンロードするために使用されます。OSプロンプトで`curl`または`curl.exe`を実行してインストールされているかどうかを確認してください。curlがインストールされていない場合は、[こちらからcurlを取得してください](https://curl.se/dlwiz/?type=bin)。

---
## 用語

### FE
フロントエンドノードは、メタデータ管理、クライアント接続管理、クエリプランニング、およびクエリスケジューリングを担当しています。各FEは、そのメモリにメタデータの完全なコピーを保存および維持するため、FE間で均一なサービスを保証します。

### BE
バックエンドノードは、データの保存およびクエリプランの実行を担当しています。

---

## StarRocksを起動する

```bash
docker run -p 9030:9030 -p 8030:8030 -p 8040:8040 -itd \
--name quickstart starrocks/allin1-ubuntu
```
---
## SQLクライアント

<Clients />

---

## データをダウンロードする

これらの2つのデータセットを自分のマシンにダウンロードします。Dockerを実行しているホストマシンにダウンロードすることができます。コンテナ内でダウンロードする必要はありません。

### ニューヨーク市のクラッシュデータ

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/NYPD_Crash_Data.csv
```

### 天候データ

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/72505394728.csv
```

---

### SQLクライアントを使用してStarRocksに接続する

:::tip

mysql CLI以外のクライアントを使用している場合は、そのクライアントを開いてください。
:::

このコマンドはDockerコンテナで`mysql`コマンドを実行します：

```sql
docker exec -it quickstart \
mysql -P 9030 -h 127.0.0.1 -u root --prompt="StarRocks > "
```

---

## テーブルを作成する

<DDL />

---

## 2つのデータセットをロードする
StarRocksにデータをロードする方法はいくつかあります。このチュートリアルでは、最も簡単な方法はcurlとStarRocks Stream Loadを使用する方法です。

:::tip
これらのcurlコマンドは`mysql`クライアントではなく、オペレーティングシステムプロンプトで実行されるため、新しいシェルを開いてください。コマンドはダウンロードした場所から実行するように指定されているため、ファイルをダウンロードしたディレクトリから実行してください。

パスワードが求められます。おそらくMySQLの`root`ユーザにパスワードを割り当てていないので、単にEnterキーを押してください。
:::

これらの`curl`コマンドは複雑に見えますが、チュートリアルの最後で詳しく説明されています。現時点では、コマンドを実行し、データを解析するためにいくつかのSQLを実行し、最後にデータロードの詳細を読むことをお勧めします。

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

前のコマンドの出力は以下です。最初のハイライトされたセクションは、期待される出力 (OKおよび1行を除きすべての行が挿入された) を示しています。1行が正しい列数を含んでいないため、1行がフィルタリングされました。

```bash
ユーザ 'root' のホストパスワード:
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

エラーがある場合、出力はエラーメッセージを見るためのURLを提供します。ブラウザでこれを開いて何が起こったかを確認してください。詳細を見ると、エラーメッセージが表示されます。

<details>

<summary>ブラウザでエラーメッセージを見る</summary>

```bash
エラー: 値の数が列の数と一致しません。29を予期していましたが、32が渡されました。

列区切り文字: 44, 行区切り文字: 10.. 行: 09/06/2015,14:15,,,40.6722269,-74.0110059,"(40.6722269, -74.0110059)",,,"R/O 1 BEARD ST. ( IKEA'S 
09/14/2015,5:30,BRONX,10473,40.814551,-73.8490955,"(40.814551, -73.8490955)",TORRY AVENUE                    ,NORTON AVENUE                   ,,0,0,0,0,0,0,0,0,Driver Inattention/Distraction,Unspecified,,,,3297457,PASSENGER VEHICLE,PASSENGER VEHICLE,,,
```

</details>

### 天候データ

クラッシュデータと同じ方法で天候データをロードします。

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