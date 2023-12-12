---
displayed_sidebar: "Japanese"
sidebar_position: 2
description: コンピュートとストレージを分離
---

# ストレージとコンピュートを分離する
import DDL from '../assets/quick-start/_DDL.mdx'
import Clients from '../assets/quick-start/_clientsCompose.mdx'
import SQL from '../assets/quick-start/_SQL.mdx'
import Curl from '../assets/quick-start/_curl.mdx'


ストレージをコンピュートから分離するシステムでは、データは低コストで信頼性の高いリモートストレージシステムに保存されます。Amazon S3、Google Cloud Storage、Azure Blob Storage、およびMinIOなどの他のS3互換ストレージのような場所です。ホットデータはローカルにキャッシュされ、キャッシュがヒットすると、クエリのパフォーマンスはストレージとコンピュートが結合したアーキテクチャと比較しても同等になります。コンピュートノード（CN）は、数秒で追加または削除できます。このアーキテクチャはストレージコストを削減し、より良いリソース分離を保証し、弾力性とスケーラビリティを提供します。

このチュートリアルでは以下の項目をカバーしています:

- DockerコンテナでStarRocksを実行する
- オブジェクトストレージにMinIOを使用する
- 共有データのためにStarRocksを構成する
- 2つのパブリックデータセットをロードする
- SELECTとJOINでデータを分析する
- ベーシックなデータ変換（ETLのT）

使用するデータは、NYC OpenDataおよびNOAAのNational Centers for Environmental Informationによって提供されています。

これらのデータセットは非常に大きいため、このチュートリアルでは過去120年間のデータをロードしないようにしています。4 GBのRAMがDockerに割り当てられたマシンでDockerイメージを実行し、このデータをロードできます。より大規模で耐障害性とスケーラブルな展開については、後で別のドキュメンテーションを提供します。

このドキュメントには多くの情報が含まれており、最初にステップバイステップの内容、最後に技術的な詳細が表示されています。これは次の順序でそれぞれの目的に対応するために行われています:

1. 読者に共有データの展開でデータをロードし、そのデータを分析することを許可する
2. 共有データの展開の構成詳細を提供する
3. ロード中の基本的なデータ変換を説明する

---

## 前提条件

### Docker

- [Docker](https://docs.docker.com/engine/install/)
- Dockerに割り当てられた4 GBのRAM
- Dockerに割り当てられた10 GBの空きディスク領域

### SQLクライアント

Docker環境で提供されているSQLクライアントを使用するか、独自のシステム上のものを使用できます。多くのMySQL互換クライアントが動作するため、このガイドはDBeaverとMySQL WorkBenchの構成に関して説明します。

### curl

`curl`はStarRocksにデータロードジョブを発行し、データセットをダウンロードするために使用されます。OSプロンプトで`curl`または`curl.exe`を実行してインストールされているかどうかを確認してください。curlがインストールされていない場合は、[ここからcurlを入手してください](https://curl.se/dlwiz/?type=bin)。

---

## 用語

### FE
フロントエンドノードは、メタデータ管理、クライアント接続管理、クエリプランニング、およびクエリスケジューリングを担当しています。各FEはメモリ内のメタデータの完全なコピーを格納し、FE間で均一なサービスを保証します。

### CN
コンピュートノードは、共有データ展開でのクエリプランの実行を担当しています。

### BE
バックエンドノードは、共有なしデータ展開でのデータの格納とクエリプランの実行を担当しています。

:::note
このガイドではBEは使用していませんが、BEとCNの違いを理解していただくためにこの情報を含めています。
:::

---

## StarRocksを起動する

Object Storageを使用して共有データでStarRocksを実行するには、次のものが必要です:

- フロントエンドエンジン（FE）
- コンピュートノード（CN）
- オブジェクトストレージ

このガイドでは、GNU Affero General Public Licenseの下で提供されているS3互換のオブジェクトストレージであるMinIOを使用します。

これら3つの必要なコンテナを提供するために、StarRocksはDockerコンポーズファイルを提供しています。

```bash
mkdir quickstart
cd quickstart
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/docker-compose.yml
```

```bash
docker compose up -d
```

サービスの進行状況を確認します。FEとCNのhealthyになるまで約30秒かかります。MinIOコンテナはヘルスインジケータは表示されませんが、MinIOウェブUIを使用して健全性を確認できます。

FEとCNが`healthy`の状態になるまで`docker compose ps`を実行します:

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

## MinIOの認証情報を生成する

StarRocksでMinIOを使用するためには、アクセスキーを生成する必要があります。

### MinIOウェブUIを開く

http://localhost:9001/access-keys に移動します。ユーザー名とパスワードはDockerコンポーズファイルで指定され、`minioadmin`と`minioadmin`です。まだアクセスキーがないことを確認できます。**Create access key +** をクリックします。

MinIOはキーを生成し、**Create** をクリックしてキーをダウンロードします。

![ユーザーがCreateをクリックすることを確認](../assets/quick-start/MinIO-create.png)

:::note
アクセスキーはクリックして**Create**をクリックするまで保存されません。キーをコピーしてページを移動しないでください。
:::

---

## SQLクライアント

<Clients />

---

## データをダウンロードする

これら2つのデータセットをFEコンテナにダウンロードします。

### FEコンテナでシェルを開く

シェルを開き、ダウンロードしたファイル用のディレクトリを作成します:

```bash
docker compose exec starrocks-fe bash
```

```bash
mkdir quickstart
cd quickstart
```

### ニューヨーク市のクラッシュデータ

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/NYPD_Crash_Data.csv
```

### 天候データ

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/72505394728.csv
```

---

## 共有データのためにStarRocksを構成する

この時点で、StarRocksとMinIOが実行中である状態にあります。MinIOのアクセスキーはStarRocksと接続するために使用されます。

### SQLクライアントでStarRocksに接続する

::tip

`docker-compose.yml`ファイルが含まれているディレクトリからこのコマンドを実行します。

mysql CLI以外のクライアントを使用している場合は、そのクライアントを開いてください。
:::

```sql
docker compose exec starrocks-fe \
mysql -P9030 -h127.0.0.1 -uroot --prompt="StarRocks > "
```

### ストレージボリュームを作成する

以下に表示される構成の詳細:

- MinIOサーバーはURL `http://minio:9000` で利用可能です
- 作成したバケットは `starrocks` と呼ばれます
- このボリュームに書き込まれるデータは、バケット `starrocks` 内の `shared` というフォルダに保存されます
:::tip
初めてデータがボリュームに書き込まれると、`shared` フォルダが作成されます
:::
- MinIOサーバーはSSLを使用していません
- MinIOのキーとシークレットは `aws.s3.access_key` と `aws.s3.secret_key` として入力されます。以前にMinIOのウェブUIで作成したアクセスキーを使用してください。
- ボリューム `shared` がデフォルトのボリュームです
:::tip
コマンドを実行する前に、強調表示されているアクセスキー情報をMinIOで作成したアクセスキーとシークレットに置き換えてください。
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
このドキュメントや、StarRocksドキュメントの多くのSQLは、`;\G` として表示されます。`\G` はmysql CLIがクエリ結果を縦に表示するようにするために使用されます。ほとんどのSQLクライアントは縦方向のフォーマット出力を解釈しないため、`\G` を `;` に置き換える必要があります。
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
「shared」フォルダには、バケットにデータが書き込まれるまで、MinIOオブジェクトリストに表示されません。
:::

---

## いくつかのテーブルを作成します

<DDL />

---

## 2つのデータセットを読み込む

StarRocksにデータを読み込む方法はいくつかあります。このチュートリアルでは、最も単純な方法として、curlとStarRocks Stream Loadを使用します。

:::tip

これらのcurlコマンドを、データセットをダウンロードしたディレクトリで、FEシェルから実行してください。

パスワードが要求されます。おそらくMySQL「root」ユーザにパスワードを割り当てていないでしょうから、単にEnterキーを押してください。

:::

`curl`コマンドは複雑に見えますが、このチュートリアルの終わりに詳細を説明しています。今のところは、コマンドを実行し、データを分析するためのいくつかのSQLを実行し、その後データの読み込みの詳細を読むことをお勧めします。

### ニューヨーク市の衝突データ - クラッシュ

```bash
curl --location-trusted -u root             \
    -T ./NYPD_Crash_Data.csv                \
    -H "label:crashdata-0"                  \
    -H "column_separator:,"                 \
    -H "skip_header:1"                      \
    -H "enclose:\""                         \
    -H "max_filter_ratio:1"                 \
    -H "columns:tmp_CRASH_DATE, tmp_CRASH_TIME, CRASH_DATE=str_to_date(concat_ws(' ', tmp_CRASH_DATE, tmp_CRASH_TIME), '%m/%d/%Y %H:%i'),BOROUGH,ZIP_CODE,LATITUDE,LONGITUDE,LOCATION,ON_STREET_NAME,CROSS_STREET_NAME,OFF_STREET_NAME,NUMBER_OF_PERSONS_INJURED,NUMBER_OF_PERSONS_KILLED,NUMBER_OF_PEDESTRIANS_INJURED,NUMBER_OF_PERSONS_KILLED,NUMBER_OF_CYCLIST_INJURED,NUMBER_OF_CYCLIST_KILLED,NUMBER_OF_MOTORIST_INJURED,NUMBER_OF_MOTORIST_KILLED,CONTRIBUTING_FACTOR_VEHICLE_1,CONTRIBUTING_FACTOR_VEHICLE_2,CONTRIBUTING_FACTOR_VEHICLE_3,CONTRIBUTING_FACTOR_VEHICLE_4,CONTRIBUTING_FACTOR_VEHICLE_5,COLLISION_ID,VEHICLE_TYPE_CODE_1,VEHICLE_TYPE_CODE_2,VEHICLE_TYPE_CODE_3,VEHICLE_TYPE_CODE_4,VEHICLE_TYPE_CODE_5" \
    -XPUT http://localhost:8030/api/quickstart/crashdata/_stream_load
```

上記のコマンドの出力は次のとおりです。最初の強調表示されたセクションは、期待される出力（OK、および行を1行除いてすべて挿入）を示しています。1行は適切な列数を含んでいないためにフィルタリングされました。

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

エラーがある場合、出力にはエラーメッセージを表示するURLが提供されます。コンテナにプライベートIPアドレスがあるため、コンテナからcurlを実行して表示する必要があります。

```bash
curl http://10.5.0.3:8040/api/_load_error_log<エラーURLからの詳細>
```

このチュートリアルの作成中に見た内容の要約を展開します。

<details>

<summary>ブラウザでエラーメッセージを表示する</summary>

```bash
Error: Value count does not match column count. Expect 29, but got 32.

Column delimiter: 44,Row delimiter: 10.. Row: 09/06/2015,14:15,,,40.6722269,-74.0110059,"(40.6722269, -74.0110059)",,,"R/O 1 BEARD ST. ( IKEA'S 
09/14/2015,5:30,BRONX,10473,40.814551,-73.8490955,"(40.814551, -73.8490955)",TORRY AVENUE                    ,NORTON AVENUE                   ,,0,0,0,0,0,0,0,0,Driver Inattention/Distraction,Unspecified,,,,3297457,PASSENGER VEHICLE,PASSENGER VEHICLE,,,
```

</details>

### 天候データ

クラッシュデータと同じ方法で天候データセットを読み込んでください。

```bash
curl --location-trusted -u root             \
    -T ./72505394728.csv                    \
    -H "label:weather-0"                    \
    -H "column_separator:,"                 \
    -H "skip_header:1"                      \
    -H "enclose:\""                         \
    -H "max_filter_ratio:1"                 \
    -H "columns: STATION, DATE, LATITUDE, LONGITUDE, ELEVATION, NAME, REPORT_TYPE, SOURCE, HourlyAltimeterSetting, HourlyDewPointTemperature, HourlyDryBulbTemperature, HourlyPrecipitation, HourlyPresentWeatherType, HourlyPressureChange, HourlyPressureTendency, HourlyRelativeHumidity, HourlySkyConditions, HourlySeaLevelPressure, HourlyStationPressure, HourlyVisibility, HourlyWetBulbTemperature, HourlyWindDirection, HourlyWindGustSpeed, HourlyWindSpeed, Sunrise, Sunset, DailyAverageDewPointTemperature, DailyAverageDryBulbTemperature, DailyAverageRelativeHumidity, DailyAverageSeaLevelPressure, DailyAverageStationPressure, DailyAverageWetBulbTemperature, DailyAverageWindSpeed, DailyCoolingDegreeDays, DailyDepartureFromNormalAverageTemperature, DailyHeatingDegreeDays, DailyMaximumDryBulbTemperature, DailyMinimumDryBulbTemperature, DailyPeakWindDirection, DailyPeakWindSpeed, DailyPrecipitation, DailySnowDepth, DailySnowfall, DailySustainedWindDirection, DailySustainedWindSpeed, DailyWeather, MonthlyAverageRH, MonthlyDaysWithGT001Precip, MonthlyDaysWithGT010Precip, MonthlyDaysWithGT32Temp, MonthlyDaysWithGT90Temp, MonthlyDaysWithLT0Temp, MonthlyDaysWithLT32Temp, MonthlyDepartureFromNormalAverageTemperature, MonthlyDepartureFromNormalCoolingDegreeDays, MonthlyDepartureFromNormalHeatingDegreeDays, MonthlyDepartureFromNormalMaximumTemperature, MonthlyDepartureFromNormalMinimumTemperature, MonthlyDepartureFromNormalPrecipitation, MonthlyDewpointTemperature, MonthlyGreatestPrecip, MonthlyGreatestPrecipDate, MonthlyGreatestSnowDepth, MonthlyGreatestSnowDepthDate, MonthlyGreatestSnowfall, MonthlyGreatestSnowfallDate, MonthlyMaxSeaLevelPressureValue, MonthlyMaxSeaLevelPressureValueDate, MonthlyMaxSeaLevelPressureValueTime, MonthlyMaximumTemperature, MonthlyMeanTemperature, MonthlyMinSeaLevelPressureValue, MonthlyMinSeaLevelPressureValueDate, MonthlyMinSeaLevelPressureValueTime, MonthlyMinimumTemperature, MonthlySeaLevelPressure, MonthlyStationPressure, MonthlyTotalLiquidPrecipitation, MonthlyTotalSnowfall, MonthlyWetBulb, AWND, CDSD, CLDD, DSNW, HDSD, HTDD, NormalsCoolingDegreeDay, NormalsHeatingDegreeDay, ShortDurationEndDate005, ShortDurationEndDate010, ShortDurationEndDate015, ShortDurationEndDate020, ShortDurationEndDate030, ShortDurationEndDate045, ShortDurationEndDate060, ShortDurationEndDate080, ShortDurationEndDate100, ShortDurationEndDate120, ShortDurationEndDate150, ShortDurationEndDate180, ShortDurationPrecipitationValue005, ShortDurationPrecipitationValue010, ShortDurationPrecipitationValue015, ShortDurationPrecipitationValue020, ShortDurationPrecipitationValue030, ShortDurationPrecipitationValue045, ShortDurationPrecipitationValue060, ShortDurationPrecipitationValue080, ShortDurationPrecipitationValue100, ShortDurationPrecipitationValue120, ShortDurationPrecipitationValue150, ShortDurationPrecipitationValue180, REM, BackupDirection, BackupDistance, BackupDistanceUnit, BackupElements, BackupElevation, BackupEquipment, BackupLatitude, BackupLongitude, BackupName, WindEquipmentChangeDate" \
```bash
-XPUT http://localhost:8030/api/quickstart/weatherdata/_stream_load
```

---

## MinIOにデータが保存されていることを確認する

MinIO [http://localhost:9001/browser/starrocks/](http://localhost:9001/browser/starrocks/) を開き、`starrocks/shared/` ディレクトリ以下の各ディレクトリに `data`、`metadata`、`schema` のエントリがあることを確認してください。

:::tip
`starrocks/shared/` 以下のフォルダ名はデータをロードする際に生成されます。`shared` の下に1つのディレクトリがあり、その下に2つのディレクトリがあります。それぞれのディレクトリの中にはデータ、メタデータ、スキーマのエントリがあります。

![MinIO object browser](../assets/quick-start/MinIO-data.png)
:::

---

## いくつかの質問に答える

<SQL />

---

## 共有データ用にStarRocksを構成する

共有データを使用したStarRocksの使用方法を体験したので、構成を理解することが重要です。

### CN構成

ここで使用されているCN構成はデフォルトです。CNは共有データの使用を想定して設計されています。デフォルトの構成は以下の通りです。変更する必要はありません。

```bash
sys_log_level = INFO

be_port = 9060
be_http_port = 8040
heartbeat_service_port = 9050
brpc_port = 8060
```

### FE構成

FE構成は、デフォルトとは少し異なります。なぜなら、FEはデータがBEノード上のローカルディスクではなくオブジェクトストレージに保存されることを想定して構成される必要があるからです。

`docker-compose.yml` ファイルはFE構成を `command` で生成します。

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

これにより、この構成ファイルが生成されます。

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

# ハイライト開始
run_mode=shared_data
aws_s3_path=starrocks
aws_s3_endpoint=minio:9000
aws_s3_use_instance_profile=false
cloud_native_storage_type=S3
aws_s3_use_aws_sdk_default_behavior=true
# ハイライト終了
```

:::note
この構成ファイルにはデフォルトのエントリと共有データの追加が含まれています。共有データのエントリはハイライトされています。
:::

デフォルトでないFE構成設定:

:::note
多くの構成パラメータは `s3_` で始まります。この接頭辞はAmazon S3互換のストレージタイプ（例：S3、GCS、MinIO）すべてで使用されます。Azure Blob Storageを使用する場合、この接頭辞は `azure_` になります。
:::

#### `run_mode=shared_data`

これにより、共有データの使用が有効になります。

#### `aws_s3_path=starrocks`

バケット名です。

#### `aws_s3_endpoint=minio:9000`

MinIOのエンドポイント（ポート番号を含む）です。

#### `aws_s3_use_instance_profile=false`

MinIOを使用する場合、アクセスキーが使用されるため、インスタンスプロファイルは使用されません。

#### `cloud_native_storage_type=S3`

これは、S3互換のストレージまたはAzure Blob Storageが使用されているかを指定します。MinIOの場合、常にS3になります。

#### `aws_s3_use_aws_sdk_default_behavior=true`

MinIOを使用する場合、このパラメータは常にtrueに設定されます。

---

## まとめ

このチュートリアルでは、以下を行いました：

- DockerでStarRocksとMinioを展開しました
- MinIOアクセスキーを作成しました
- MinIOを使用するStarRocksストレージボリュームを構成しました
- ニューヨーク市の提供するクラッシュデータとNOAAの提供する天候データをロードしました
- 低い視界や凍結した道路で運転することが悪いことを見つけるためにSQL JOINを使用してデータを解析しました

さらに学ぶことがあります。私たちは意図的に、ストリームロード中に行われるデータ変換については省略しました。その詳細は以下のcurlコマンドのノートにあります。

## curlコマンドのノート

<Curl />

## さらなる情報

[StarRocksテーブル設計](../table_design/StarRocks_table_design.md)

[マテリアライズドビュー](../cover_pages/mv_use_cases.mdx)

[ストリームロード](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)

[モータービークルコリジョン - クラッシュ](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95) データセットは、これらの[利用条件](https://www.nyc.gov/home/terms-of-use.page)および[プライバシーポリシー](https://www.nyc.gov/home/privacy-policy.page)に従ってニューヨーク市が提供しています。

[ローカル気象データ](https://www.ncdc.noaa.gov/cdo-web/datatools/lcd)（LCD）は、NOAAがこの[免責事項](https://www.noaa.gov/disclaimer)とこの[プライバシーポリシー](https://www.noaa.gov/protecting-your-privacy)に従って提供しています。
```