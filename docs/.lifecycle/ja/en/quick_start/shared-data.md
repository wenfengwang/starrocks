---
description: Separate compute and storage
displayed_sidebar: English
sidebar_position: 2
---

# ストレージとコンピュートの分離
import DDL from '../assets/quick-start/_DDL.mdx'
import Clients from '../assets/quick-start/_clientsCompose.mdx'
import SQL from '../assets/quick-start/_SQL.mdx'
import Curl from '../assets/quick-start/_curl.mdx'


ストレージとコンピュートを分離するシステムでは、データはAmazon S3、Google Cloud Storage、Azure Blob Storageなどの低コストで信頼性の高いリモートストレージシステム、またはMinIOのような他のS3互換ストレージに格納されます。ホットデータはローカルにキャッシュされ、キャッシュがヒットすると、クエリのパフォーマンスはストレージとコンピュートが結合されたアーキテクチャに匹敵します。コンピュートノード（CN）は、オンデマンドで数秒以内に追加または削除できます。このアーキテクチャにより、ストレージコストが削減され、リソースの分離が向上し、弾力性とスケーラビリティが提供されます。

このチュートリアルでは、以下の内容について説明します：

- DockerコンテナでのStarRocksの実行
- オブジェクトストレージとしてMinIOの使用
- 共有データ用のStarRocksの設定
- 2つの公開データセットのロード
- SELECTとJOINを使用したデータ分析
- 基本的なデータ変換（ETLの**T**）

使用されるデータはNYC OpenDataとNOAAのNational Centers for Environmental Informationによって提供されています。

これらのデータセットは非常に大きいため、このチュートリアルはStarRocksの操作に慣れることを目的としており、過去120年分のデータをロードすることはありません。Dockerに4GBのRAMが割り当てられたマシンでDockerイメージを実行し、このデータをロードすることができます。より大規模でフォールトトレラントなスケーラブルなデプロイメントについては、他のドキュメントがあり、それについては後ほど説明します。

このドキュメントには多くの情報が含まれており、最初にステップバイステップの内容が、最後に技術的な詳細が提示されています。これは、以下の目的をこの順序で達成するためです：

1. 読者が共有データデプロイメントでデータをロードし、そのデータを分析できるようにする。
2. 共有データデプロイメントの設定の詳細を提供する。
3. データロード中の基本的なデータ変換について説明する。

---

## 前提条件

### Docker

- [Docker](https://docs.docker.com/engine/install/)
- Dockerに割り当てられた4GBのRAM
- Dockerに割り当てられた10GBの空きディスクスペース

### SQLクライアント

Docker環境に提供されているSQLクライアントを使用することも、システム上で使用することもできます。多くのMySQL互換クライアントが動作し、このガイドではDBeaverとMySQL Workbenchの設定について説明します。

### curl

`curl`は、StarRocksへのデータロードジョブを発行し、データセットをダウンロードするために使用されます。OSプロンプトで`curl`または`curl.exe`を実行して、インストールされているかどうかを確認します。curlがインストールされていない場合は、[ここからcurlを入手してください](https://curl.se/dlwiz/?type=bin)。

---

## 用語集

### FE
フロントエンドノードは、メタデータ管理、クライアント接続管理、クエリプランニング、およびクエリスケジューリングを担当します。各FEはメモリ内にメタデータの完全なコピーを保存および維持し、FE間でサービスが不変であることを保証します。

### CN
コンピュートノードは、共有データデプロイメントでクエリプランの実行を担当します。

### BE
バックエンドノードは、共有なしデプロイメントでデータストレージとクエリプランの実行の両方を担当します。

:::note
このガイドではBEは使用しませんが、BEとCNの違いを理解するためにこの情報が含まれています。
:::

---

## StarRocksの起動

オブジェクトストレージを使用して共有データでStarRocksを実行するためには、以下が必要です：

- フロントエンドエンジン（FE）
- コンピュートノード（CN）
- オブジェクトストレージ

このガイドでは、GNU Affero General Public Licenseの下で提供されるS3互換のオブジェクトストレージであるMinIOを使用します。

必要な3つのコンテナを提供するために、StarRocksはDocker Composeファイルを提供します。

```bash
mkdir quickstart
cd quickstart
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/docker-compose.yml
```

```bash
docker compose up -d
```

サービスの進行状況を確認します。FEとCNが`healthy`のステータスを表示するまで約30秒かかります。MinIOコンテナは健全性指標を表示しませんが、MinIOのWeb UIを使用してその健全性を確認できます。

FEとCNが`healthy`のステータスを表示するまで`docker compose ps`を実行します。

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

## MinIOの資格情報を生成する

StarRocksでオブジェクトストレージとしてMinIOを使用するには、**アクセスキー**を生成する必要があります。

### MinIOのWeb UIを開く

http://localhost:9001/access-keys にアクセスします。ユーザー名とパスワードはDocker Composeファイルで指定されており、`minioadmin`と`minioadmin`です。まだアクセスキーがないことがわかります。**アクセスキーを作成+**をクリックします。

MinIOはキーを生成します。**作成**をクリックしてキーをダウンロードします。

![「作成」をクリックしてください](../assets/quick-start/MinIO-create.png)

:::note
アクセスキーは、**作成**をクリックするまで保存されません。キーをコピーしてページから離れないでください。
:::

---

## SQLクライアント

<Clients />

---

## データをダウンロードする

これらの2つのデータセットをFEコンテナにダウンロードします。

### FEコンテナでシェルを開く

シェルを開き、ダウンロードしたファイル用のディレクトリを作成します。

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

### 気象データ

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/72505394728.csv
```

---

## 共有データ用にStarRocksを設定する

この時点でStarRocksが実行されており、MinIOも実行されています。MinIOのアクセスキーはStarRocksとMinIOを接続するために使用されます。

### SQLクライアントでStarRocksに接続する

:::tip


`docker-compose.yml` ファイルが含まれるディレクトリからこのコマンドを実行してください。

mysql CLI以外のクライアントを使用している場合は、それを今開いてください。
:::

```sql
docker compose exec starrocks-fe \
mysql -P9030 -h127.0.0.1 -uroot --prompt="StarRocks > "
```

### ストレージボリュームの作成

以下に示す設定の詳細です：

- MinIOサーバーはURL `http://minio:9000` で利用可能です
- 上で作成したバケットは `starrocks` という名前です
- このボリュームに書き込まれるデータは、バケット `starrocks` 内の `shared` というフォルダに保存されます
:::tip
フォルダ `shared` は、初めてデータがボリュームに書き込まれる際に作成されます
:::
- MinIOサーバーはSSLを使用していません
- MinIOのキーとシークレットは `aws.s3.access_key` と `aws.s3.secret_key` として入力されます。以前にMinIO Web UIで作成したアクセスキーを使用してください。
- ボリューム `shared` はデフォルトのボリュームです

:::tip
実行する前にコマンドを編集し、強調表示されたアクセスキー情報をMinIOで作成したアクセスキーとシークレットに置き換えてください。
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
このドキュメントのSQL、およびStarRocksドキュメントの他の多くのドキュメントでは、セミコロンの代わりに `\G` が使用されています。`\G` はmysql CLIにクエリ結果を縦に表示させる効果があります。

多くのSQLクライアントは縦のフォーマット出力を解釈しないので、`\G` を `;` に置き換える必要があります。
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
フォルダ `shared` は、データがバケットに書き込まれるまでMinIOオブジェクトリストには表示されません。
:::

---

## テーブルの作成

<DDL />

---

## 2つのデータセットのロード

StarRocksにデータをロードする方法は多数ありますが、このチュートリアルではcurlとStarRocks Stream Loadを使用するのが最もシンプルです。

:::tip

これらのcurlコマンドは、データセットをダウンロードしたディレクトリにあるFEシェルから実行してください。

パスワードを求められますが、MySQLの `root` ユーザーにパスワードを設定していない場合は、ただEnterキーを押してください。

:::

`curl` コマンドは複雑に見えるかもしれませんが、チュートリアルの最後に詳細が説明されています。今は、コマンドを実行してデータを分析するためのSQLを実行し、その後でデータロードの詳細について読むことをお勧めします。

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

上記のコマンドの出力は以下の通りです。最初に強調表示されたセクションは、期待される結果（OKと1行を除く全ての行が挿入された）を示しています。列の数が正しくないため、1行がフィルタリングされました。

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

エラーが発生した場合、出力にはエラーメッセージを確認するためのURLが提供されます。コンテナがプライベートIPアドレスを持っているため、コンテナ内からcurlを実行して確認する必要があります。

```bash
curl http://10.5.0.3:8040/api/_load_error_log<ErrorURLからの詳細>
```

このチュートリアルの開発中に表示された内容の要約を展開します：

<details>

<summary>ブラウザでのエラーメッセージの読み方</summary>

```bash
Error: Value count does not match column count. Expect 29, but got 32.

Column delimiter: 44, Row delimiter: 10.. Row: 09/06/2015,14:15,,,40.6722269,-74.0110059,"(40.6722269, -74.0110059)",,,"R/O 1 BEARD ST. (IKEA'S 
09/14/2015,5:30,BRONX,10473,40.814551,-73.8490955,"(40.814551, -73.8490955)",TORRY AVENUE, NORTON AVENUE,,0,0,0,0,0,0,0,0,Driver Inattention/Distraction,Unspecified,,,,3297457,PASSENGER VEHICLE,PASSENGER VEHICLE,,,
```

</details>

### 気象データ

気象データセットを、事故データと同じ方法でロードします。

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

## MinIOにデータが格納されていることを確認

MinIO [http://localhost:9001/browser/starrocks/](http://localhost:9001/browser/starrocks/) を開いて、`starrocks/shared/`の下のディレクトリに`data`、`metadata`、`schema`の各エントリがあることを確認してください。

:::tip
`starrocks/shared/`以下のフォルダ名は、データをロードする際に生成されます。`shared`の下には1つのディレクトリがあり、その下にさらに2つのディレクトリがあります。それぞれのディレクトリ内には、データ、メタデータ、スキーマのエントリが見つかります。

![MinIOオブジェクトブラウザ](../assets/quick-start/MinIO-data.png)
:::

---

## いくつかの質問に答える

<SQL />

---

## 共有データ用にStarRocksを設定する

共有データを使用してStarRocksを体験したので、設定について理解することが重要です。

### CNの設定

ここで使用されるCNの設定はデフォルトです。CNは共有データの使用に適しているため、変更する必要はありません。デフォルトの設定は以下の通りです。

```bash
sys_log_level = INFO

be_port = 9060
be_http_port = 8040
heartbeat_service_port = 9050
brpc_port = 8060
```

### FEの設定

FEの設定はデフォルトとは少し異なります。FEは、データがBEノードのローカルディスクではなくオブジェクトストレージに保存されることを想定して設定する必要があります。

`docker-compose.yml`ファイルは、`command`でFEの設定を生成します。

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

これにより、以下の設定ファイルが生成されます。

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

:::note
この設定ファイルには、デフォルトのエントリと共有データ用の追加が含まれています。共有データ用のエントリは強調表示されています。
:::

デフォルト以外のFE設定:

:::note
多くの設定パラメータには`s3_`のプレフィックスが付いています。このプレフィックスは、Amazon S3互換のすべてのストレージタイプ（例：S3、GCS、MinIOなど）に使用されます。Azure Blob Storageを使用する場合、プレフィックスは`azure_`です。
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

これは、S3互換ストレージまたはAzure Blob Storageのどちらを使用するかを指定します。MinIOの場合、これは常にS3です。

#### `aws_s3_use_aws_sdk_default_behavior=true`

MinIOを使用する場合、このパラメータは常にtrueに設定されます。

---

## まとめ

このチュートリアルでは、以下のことを行いました：

- DockerにStarRocksとMinioをデプロイしました
- MinIOアクセスキーを作成しました
- MinIOを使用するStarRocksストレージボリュームを設定しました
- ニューヨーク市から提供されたクラッシュデータとNOAAから提供された気象データをロードしました
- SQL JOINを使用してデータを分析し、視界が低いか氷の道で運転することは良くないという結果を得ました

まだ学ぶべきことはたくさんあります。ストリームロード中に行われたデータ変換については意図的に説明を省略しました。その詳細は、以下のcurlコマンドに関する注記で説明しています。

## curlコマンドに関する注記

<Curl />

## 詳細情報

[StarRocks テーブル設計](../table_design/StarRocks_table_design.md)

[マテリアライズド・ビュー](../cover_pages/mv_use_cases.mdx)

[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)

[Motor Vehicle Collisions - Crashes](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95) データセットは、ニューヨーク市からこれらの[利用規約](https://www.nyc.gov/home/terms-of-use.page)および[プライバシーポリシー](https://www.nyc.gov/home/privacy-policy.page)に基づいて提供されています。

[Local Climatological Data](https://www.ncdc.noaa.gov/cdo-web/datatools/lcd)(LCD)は、NOAAからこの[免責事項](https://www.noaa.gov/disclaimer)および[プライバシーポリシー](https://www.noaa.gov/protecting-your-privacy)をもって提供されています。
