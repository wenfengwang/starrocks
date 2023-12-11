---
displayed_sidebar: "Japanese"
sidebar_position: 2
description: コンピューティングとストレージを分離
---

# ストレージとコンピューティングを分離する
import DDL from '../assets/quick-start/_DDL.mdx'
import Clients from '../assets/quick-start/_clientsCompose.mdx'
import SQL from '../assets/quick-start/_SQL.mdx'
import Curl from '../assets/quick-start/_curl.mdx'


コンピューティングからストレージを分離するシステムでは、データはAmazon S3、Google Cloud Storage、Azure Blob Storageなどの低コストで信頼性の高いリモートストレージシステムやMinIOなどの他のS3互換ストレージに保存されます。ホットデータはローカルにキャッシュされます。キャッシュがヒットすると、クエリのパフォーマンスはストレージとコンピューティングが結合されたアーキテクチャと比較して同等です。コンピューティングノード（CN）は、数秒で追加または削除できます。このアーキテクチャは、ストレージコストを削減し、より良いリソースの分離を確認し、弾力性とスケーラビリティを提供します。

このチュートリアルでは、以下の内容がカバーされています。

- DockerコンテナーでStarRocksを実行する
- オブジェクトストレージとしてMinIOを使用する
- 共有データのStarRocksの構成
- 2つの公開データセットをロード
- SELECTおよびJOINでデータを分析
- データの基本的な変換（ETLの「T」）

使用されているデータは、NYC OpenDataおよびNOAAのNational Centers for Environmental Informationによって提供されています。

これらのデータセットは非常に大きいため、このチュートリアルはStarRocksと連携する作業に慣れるのを手助けすることを目的としているため、過去120年間のデータをロードすることはありません。Dockerイメージを実行し、このデータをDockerに割り当てられた4 GBのRAMのマシンにロードできます。より大規模な耐障害性とスケーラブルな展開については、他のドキュメントがあり、後で提供されます。

この文書には多くの情報が含まれており、最初にステップバイステップの内容で提示され、最後に技術的な詳細が示されています。これは、次の順序でこれらの目的を果たすために行われています。

1. 読者に共有データの展開でデータをロードし、そのデータを分析する機会を提供する
2. 共有データ展開の構成詳細を提供する
3. ロード中のデータ変換の基本を説明する

---

## 前提条件

### Docker

- [Docker](https://docs.docker.com/engine/install/)
- Dockerに割り当てられた4 GB RAM
- Dockerに割り当てられた10 GBの空きディスクスペース

### SQLクライアント

Docker環境で提供されているSQLクライアントを使用するか、システム上のSQLクライアントを使用できます。多くのMySQL互換のクライアントが動作し、このガイドではDBeaverとMySQL WorkBenchの構成をカバーしています。

### curl

`curl`は、データロードジョブをStarRocksに発行したり、データセットをダウンロードしたりするために使用されます。OSプロンプトで`curl`または`curl.exe`を実行してインストールされているかどうかを確認してください。curlがインストールされていない場合は、[ここからcurlを取得してください](https://curl.se/dlwiz/?type=bin)。

---

## 用語

### FE
フロントエンドノードは、メタデータ管理、クライアント接続管理、クエリプランニング、クエリスケジューリングを担当しています。各FEは、そのメモリにメタデータの完全なコピーを保存および維持し、FE間のサービスの均一性を保証します。

### CN
コンピューティングノードは、共有データ展開でクエリプランを実行する責任を持っています。

### BE
バックエンドノードは、共有ナッシング展開でデータストレージとクエリプランの実行を担当しています。

:::note
このガイドではBEは使用しません。BEとCNの違いを理解するためにこの情報がここに含まれています。
:::

---

## StarRocksを起動する

オブジェクトストレージを使用して共有データでStarRocksを実行するには、次のものが必要です。

- フロントエンドエンジン（FE）
- コンピューティングノード（CN）
- オブジェクトストレージ

このガイドでは、GNU Affero General Public Licenseの下で提供されているS3互換のオブジェクトストレージであるMinIOを使用しています。

これら3つの必要なコンテナを備えた環境を提供するために、StarRocksはDockerコンポーズファイルを提供しています。

```bash
mkdir quickstart
cd quickstart
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/docker-compose.yml
```

```bash
docker compose up -d
```

サービスの進捗状況を確認します。FEとCNが健全になるまでに約30秒かかります。MinIOコンテナにはヘルスインジケータが表示されませんが、MinIOウェブUIを使用してその健全性を確認します。

FEとCNがステータスを `healthy` に表示するまで、`docker compose ps`を実行します。

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

StarRocksでMinIOを使用するには、**アクセスキー**を生成する必要があります。

### MinIOウェブUIを開く

http://localhost:9001/access-keys にアクセスします。ユーザー名とパスワードは、Dockerコンポーズファイルで指定されており、`minioadmin`と`minioadmin`です。まだアクセスキーがないことを確認できるはずです。**Create access key +**をクリックします。

MinIOはキーを生成し、**Create**をクリックしてキーをダウンロードします。

![Make sure to click create](../assets/quick-start/MinIO-create.png)

:::note
アクセスキーは、**Create**をクリックするまで保存されません。キーをコピーしてページから移動しないでください
:::

---

## SQLクライアント

<Clients />

---

## データをダウンロードする

これら2つのデータセットをFEコンテナにダウンロードします。

### FEコンテナでシェルを開く

シェルを開き、ダウンロードされたファイルのためのディレクトリを作成します。

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

### 天気データ

```bash
curl -O https://raw.githubusercontent.com/StarRocks/demo/master/documentation-samples/quickstart/datasets/72505394728.csv
```

---

## 共有データのStarRocksの構成

この時点で、StarRocksが実行されており、MinIOが実行されています。MinIOのアクセスキーは、StarRocksとMinioを接続するために使用されます。

### SQLクライアントでStarRocksに接続

:::tip

`docker-compose.yml` ファイルが含まれているディレクトリからこのコマンドを実行してください。

mysql CLI以外のクライアントを使用している場合は、今すぐ開いてください。
:::

```sql
docker compose exec starrocks-fe \
mysql -P9030 -h127.0.0.1 -uroot --prompt="StarRocks > "
```

### ストレージボリュームを作成する

以下に示す構成の詳細:

- MinIOサーバーはURL `http://minio:9000` で利用可能
- 作成したバケットは `starrocks` と呼ばれます
- このボリュームに書き込まれるデータは、バケット `starrocks` 内の `shared` という名前のフォルダに保存されます
:::tip
`shared` フォルダは、このボリュームに初めてデータを書き込むときに作成されます
:::
- MinIOサーバーはSSLを使用していません
- MinIO鍵とシークレットは、`aws.s3.access_key` および `aws.s3.secret_key` として入力されます。前述のMinIOウェブUIで作成したアクセスキーを使用します。
- `shared` ボリュームがデフォルトボリュームです

:::tip
コマンドを実行する前に、ハイライトされたアクセスキー情報を、MinIOで作成したアクセスキーとシークレットに置き換えてください。
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
この文書のSQLの一部、およびStarRocksドキュメントの多くの他の文書は、 `;`の代わりに `\G`を使用しています。` `\G` はmysql CLIにクエリ結果を縦に描画するように指示します。

多くのSQLクライアントは、垂直書式の出力を解釈しないため、`\G` を `;` に置き換える必要があります。
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
`shared`フォルダにデータが書き込まれるまで、MinIOオブジェクトリストで `shared` フォルダは表示されません。
:::

---

## いくつかのテーブルを作成する

<DDL />

---

## データセットを2つロードする

StarRocksにデータをロードする方法はたくさんあります。このチュートリアルでは、最も簡単な方法として、curlとStarRocks Stream Loadを使用します。

:::tip

これらのcurlコマンドは、データセットをダウンロードしたディレクトリ内のFEシェルから実行します。

パスワードを入力します。おそらくMySQLの `root` ユーザーにパスワードを割り当てていない場合は、Enter キーを押します。

:::

`curl` コマンドは複雑に見えますが、このチュートリアルの最後で詳しく説明します。今のところ、コマンドを実行してデータを解析し、その後データロードの詳細について読むことをお勧めします。

### ニューヨーク市の衝突データ - 事故

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

上記のコマンドの出力は以下の通りです。最初の強調表示されたセクションは、期待される表示（OK および行を挿入した結果を除くすべてのもの）を示しています。1行は正しい列数を持っていないためにフィルタリングされました。

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

エラーが発生した場合、出力にはエラーメッセージを確認するためのURLが示されます。コンテナにプライベートIPアドレスがあるため、コンテナからcurlを実行して表示する必要があります。

```bash
curl http://10.5.0.3:8040/api/_load_error_log<details from ErrorURL>
```

このチュートリアルを開発中に見たコンテンツのサマリーを展開します：

<details>

<summary>ブラウザでエラーメッセージを読む</summary>

```bash
Error: Value count does not match column count. Expect 29, but got 32.

Column delimiter: 44,Row delimiter: 10.. Row: 09/06/2015,14:15,,,40.6722269,-74.0110059,"(40.6722269, -74.0110059)",,,"R/O 1 BEARD ST. ( IKEA'S 
09/14/2015,5:30,BRONX,10473,40.814551,-73.8490955,"(40.814551, -73.8490955)",TORRY AVENUE                    ,NORTON AVENUE                   ,,0,0,0,0,0,0,0,0,Driver Inattention/Distraction,Unspecified,,,,3297457,PASSENGER VEHICLE,PASSENGER VEHICLE,,,
```

</details>

### 天気データ

衝突データと同じ方法で天気データセットをロードします。

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

MinIO [http://localhost:9001/browser/starrocks/](http://localhost:9001/browser/starrocks/) を開き、`starrocks/shared/` の各ディレクトリに`data`、`metadata`、`schema`のエントリがあることを確認してください。

:::tip
`starrocks/shared/` の下のフォルダ名はデータをロードすると生成されます。`shared`の下に1つのディレクトリがあり、その下に2つのディレクトリがさらにあります。それぞれのディレクトリの中にはデータ、メタデータ、およびスキーマのエントリがあります。

![MinIO object browser](../assets/quick-start/MinIO-data.png)
:::

---

## いくつかの質問に回答する

<SQL />

---

## 共有データのStarRocksの構成

共有データを使用したStarRocksの経験を得たので、構成を理解することが重要です。

### CN構成

ここで使用されているCN構成はデフォルトのものであり、CNは共有データの使用を目的として設計されています。デフォルトの構成は以下の通りです。変更する必要はありません。

```bash
sys_log_level = INFO

be_port = 9060
be_http_port = 8040
heartbeat_service_port = 9050
brpc_port = 8060
```

### FE構成

FE構成は、デフォルトとは少し異なります。なぜなら、FEはBEノードのローカルディスクではなく、オブジェクトストレージにデータが保存されることを想定して構成されている必要があるからです。

`docker-compose.yml`ファイルはFE構成を`command`に生成します。

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
この構成ファイルには、デフォルトのエントリと共有データ用の追加エントリが含まれています。共有データのエントリはハイライト表示されています。
:::

非デフォルトのFE構成設定:

:::note
多くの構成パラメータは`s3_`で始まります。この接頭辞は、すべてのAmazon S3互換のストレージタイプ（例: S3、GCS、MinIO）に使用されます。Azure Blob Storageを使用する場合、接頭辞は`azure_`です。
:::

#### `run_mode=shared_data`

これにより、共有データの使用が有効になります。

#### `aws_s3_path=starrocks`

バケット名です。

#### `aws_s3_endpoint=minio:9000`

ポート番号を含むMinIOのエンドポイントです。

#### `aws_s3_use_instance_profile=false`

MinIOを使用する際にはアクセスキーを使用するため、インスタンスプロファイルは使用しません。

#### `cloud_native_storage_type=S3`

これは、S3互換のストレージまたはAzure Blob Storageが使用されているかどうかを指定します。MinIOの場合、これは常にS3です。

#### `aws_s3_use_aws_sdk_default_behavior=true`

MinIOを使用する場合、このパラメータは常にtrueに設定されます。

---

## 要約

このチュートリアルでは以下のことを行いました：

- StarRocksとMinioをDockerでデプロイしました
- MinIOのアクセスキーを作成しました
- MinIOを使用するStarRocksストレージボリュームを構成しました
- ニューヨーク市が提供するクラッシュデータおよびNOAAが提供する気象データをロードしました
- 低視界または凍結した通りでの運転は良くないことを発見するためにSQL JOINを使用してデータを分析しました

これから学ぶことは他にもあります；わざとStream Load中に行われるデータ変換については意図的に省略しました。これに関する詳細は以下のcurlコマンドに関する注釈にあります。

## curlコマンドに関する注釈

<Curl />

## さらなる情報

[StarRocksテーブル設計](../table_design/StarRocks_table_design.md)

[マテリアライズドビュー](../cover_pages/mv_use_cases.mdx)

[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)

[モータービークルコリジョン-クラッシュ](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95)データセットは、これらの[使用条件](https://www.nyc.gov/home/terms-of-use.page)および[プライバシーポリシー](https://www.nyc.gov/home/privacy-policy.page)に従ってニューヨーク市が提供しています。

[Local Climatological Data](https://www.ncdc.noaa.gov/cdo-web/datatools/lcd)(LCD)は、NOAAによって[this disclaimer](https://www.noaa.gov/disclaimer)および[this privacy policy](https://www.noaa.gov/protecting-your-privacy)と共に提供されています。
```