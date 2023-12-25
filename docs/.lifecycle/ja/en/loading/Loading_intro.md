---
displayed_sidebar: English
---

# データロードの概要

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

データロードは、ビジネス要件に基づいてさまざまなデータソースからの生データをクレンジングし変換し、その結果のデータをStarRocksにロードして、超高速データ分析を容易にするプロセスです。

StarRocksにデータをロードするには、ロードジョブを実行します。各ロードジョブには、ユーザーが指定した一意のラベル、またはStarRocksによって自動生成される一意のラベルがあります。各ラベルは、1つのロードジョブにのみ使用できます。ロードジョブが完了した後、そのラベルを他のロードジョブで再利用することはできません。失敗したロードジョブのラベルのみが再利用可能です。このメカニズムにより、特定のラベルに関連付けられたデータは一度だけロードされることが保証され、At-Most-Onceセマンティクスが実装されます。

StarRocksが提供するすべてのロード方法は、原子性を保証します。原子性とは、ロードジョブ内の適格データが全て成功してロードされるか、全くロードされないことを意味します。適格データの一部がロードされて他のデータがロードされないということはありません。適格データには、データ型変換エラーなどの品質問題によりフィルタリングされたデータは含まれません。

StarRocksは、ロードジョブを送信するために使用できる2つの通信プロトコル、MySQLとHTTPをサポートしています。各ロード方法によってサポートされるプロトコルの詳細については、[ロード方法](../loading/Loading_intro.md#loading-methods)セクションを参照してください。

<InsertPrivNote />

## サポートされるデータ型

StarRocksは、すべてのデータ型のデータロードをサポートしています。特定のデータ型のロードに関する制限に注意する必要があります。詳細については、[データ型](../sql-reference/sql-statements/data-types/BIGINT.md)を参照してください。

## ロードモード

StarRocksは、同期ロードモードと非同期ロードモードの2つのロードモードをサポートしています。

> **注記**
>
> 外部プログラムを使用してデータをロードする場合、ビジネス要件に最適なロードモードを選択した後、選択するロード方法を決定する必要があります。

### 同期ロード

同期ロードモードでは、ロードジョブを送信した後、StarRocksはそのジョブを同期的に実行してデータをロードし、ジョブが完了した後に結果を返します。ジョブ結果に基づいて、ジョブが成功したかどうかを確認できます。

StarRocksは、同期ロードをサポートする2つのロード方法を提供しています：[Stream Load](../loading/StreamLoad.md)と[INSERT](../loading/InsertInto.md)。

同期ロードのプロセスは以下の通りです：

1. ロードジョブを作成します。

2. StarRocksから返されるジョブ結果を確認します。

3. ジョブ結果に基づいて、ジョブが成功したかどうかを確認します。ジョブ結果がロード失敗を示している場合、ジョブを再試行できます。

### 非同期ロード

非同期ロードモードでは、ロードジョブを送信すると、StarRocksは直ちにジョブ作成結果を返します。

- 結果がジョブ作成成功を示している場合、StarRocksはジョブを非同期に実行します。しかし、それはデータが正常にロードされたことを意味するものではありません。ステートメントやコマンドを使用してジョブの状態を確認し、ジョブの状態に基づいてデータが正常にロードされたかどうかを判断する必要があります。

- 結果がジョブ作成失敗を示している場合、失敗情報に基づいてジョブを再試行するかどうかを判断できます。

StarRocksは、非同期ロードをサポートする3つのロード方法を提供しています：[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、および[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)。

非同期ロードのプロセスは以下の通りです：

1. ロードジョブを作成します。

2. StarRocksから返されるジョブ作成結果を確認し、ジョブが正常に作成されたかどうかを判断します。
   a. ジョブ作成が成功した場合、ステップ3に進みます。
   b. ジョブ作成が失敗した場合、ステップ1に戻ります。

3. ステートメントやコマンドを使用して、ジョブの状態が**FINISHED**または**CANCELLED**になるまでジョブの状態を確認します。

Broker LoadまたはSpark Loadジョブのワークフローは、以下の図に示されている5つのステージで構成されています。

![Broker LoadまたはSpark Loadのワークフロー](../assets/4.1-1.png)

ワークフローは以下のように説明されます：

1. **PENDING**

   ジョブはFEによってスケジュールされるのを待っているキューに入っています。

2. **ETL**

   FEはデータを前処理します。これにはクレンジング、パーティション分割、ソート、集約などが含まれます。
   > **注記**
   >
   > ジョブがBroker Loadジョブの場合、このステージは直ちに完了します。

3. **LOADING**

   FEはデータをクレンジングし変換し、その後BEにデータを送信します。すべてのデータがロードされた後、データは有効になるのを待っているキューに入ります。この時点で、ジョブの状態は**LOADING**のままです。

4. **FINISHED**

   すべてのデータが有効になったとき、ジョブの状態は**FINISHED**になります。この時点で、データはクエリ可能です。**FINISHED**はジョブの最終状態です。

5. **CANCELLED**

   ジョブの状態が**FINISHED**になる前に、いつでもジョブをキャンセルすることができます。また、StarRocksはロードエラーが発生した場合に自動的にジョブをキャンセルすることができます。ジョブがキャンセルされた後、ジョブの状態は**CANCELLED**になります。**CANCELLED**もジョブの最終状態です。

Routine Loadジョブのワークフローは以下のように説明されます：

1. ジョブはMySQLクライアントからFEに送信されます。

2. FEはジョブを複数のタスクに分割します。各タスクは、複数のパーティションからデータをロードするように設計されています。

3. FEは指定されたBEにタスクを配布します。

4. BEはタスクを実行し、完了後にFEに報告します。

5. FEはBEからの報告に基づいて、後続のタスクを生成したり、失敗したタスクを再試行したり、タスクのスケジューリングを中断したりします。

## ロード方法

StarRocksは、さまざまなビジネスシナリオでデータをロードするための5つのロード方法を提供しています：[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)、および[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

| 読み込み方式     | データソース                                        | ビジネスシナリオ                                            | ロードジョブあたりのデータ量                                     | データファイル形式                                | 読み込みモード | プロトコル |
| ------------------ | -------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ----------------------------------------------- | ------------ | -------- |
| Stream Load        |  <ul><li>ローカルファイル</li><li>データストリーム</li></ul>| ローカルファイルシステムからデータファイルをロードするか、プログラムを使用してデータストリームをロードします。 | 10 GB以下                             |<ul><li>CSV</li><li>JSON</li></ul>               | 同期  | HTTP     |
| Broker Load        | <ul><li>HDFS</li><li>Amazon S3</li><li>Google GCS</li><li>Microsoft Azure Storage</li><li>Alibaba Cloud OSS</li><li>Tencent Cloud COS</li><li>Huawei Cloud OBS</li><li>その他のS3互換ストレージシステム（例：MinIO）</li></ul>| HDFSまたはクラウドストレージからデータをロードします。                        | 数十GBから数百GB                               | <ul><li>CSV</li><li>Parquet</li><li>ORC</li></ul>| 非同期 | MySQL    |
| Routine Load       | Apache Kafka®                                       | Kafkaからリアルタイムでデータをロードします。                   | MBからGBのデータをミニバッチとして                           |<ul><li>CSV</li><li>JSON</li><li>Avro (v3.0.1以降でサポート)</li></ul>          | 非同期 | MySQL    |
| Spark Load         | <ul><li>HDFS</li><li>Hive</li></ul>     |<ul><li>Apache Spark™クラスターを使用して、HDFSまたはHiveから大量のデータを移行します。</li><li>グローバルデータディクショナリを使用してデータ重複排除しながらロードします。</li></ul>| 数十GBからTB                                         |<ul><li>CSV</li><li>ORC (v2.0以降でサポート)</li><li>Parquet (v2.0以降でサポート)</li></ul>       | 非同期 | MySQL    |
| INSERT INTO SELECT | <ul><li>StarRocksテーブル</li><li>外部テーブル</li><li>AWS S3</li></ul>**注意**<br />AWS S3からデータをロードする場合、Parquet形式またはORC形式のファイルのみがサポートされます。     |<ul><li>外部テーブルからデータをロードします。</li><li>StarRocksテーブル間でデータをロードします。</li></ul>| 固定されていない（データ量はメモリサイズに基づいて変動します。） | StarRocksテーブル      | 同期  | MySQL    |
| INSERT INTO VALUES | <ul><li>プログラム</li><li>ETLツール</li></ul>    |<ul><li>少量のデータを個別レコードとして挿入します。</li><li>JDBCなどのAPIを使用してデータをロードします。</li></ul>| 少量                                          | SQL                   | 同期  | MySQL    |

ビジネスシナリオ、データ量、データソース、データファイル形式、および読み込み頻度に基づいて、適切な読み込み方法を選択できます。また、読み込み方法を選択する際には、以下の点に注意してください。

- Kafkaからデータをロードする場合は、[Routine Load](../loading/RoutineLoad.md)の使用を推奨します。しかし、データに複数テーブルの結合やETL操作が必要な場合は、Apache Flink®を使用してKafkaからデータを読み取り、前処理を行った後、[flink-connector-starrocks](../loading/Flink-connector-starrocks.md)を使用してStarRocksにデータをロードできます。

- Hive、Iceberg、Hudi、またはDelta Lakeからデータをロードする場合は、[Hiveカタログ](../data_source/catalog/hive_catalog.md)、[Icebergカタログ](../data_source/catalog/iceberg_catalog.md)、[Hudiカタログ](../data_source/catalog/hudi_catalog.md)、または[Delta Lakeカタログ](../data_source/catalog/deltalake_catalog.md)を作成し、[INSERT](../loading/InsertInto.md)を使用してデータをロードすることを推奨します。

- 別のStarRocksクラスターやElasticsearchクラスターからデータをロードする場合は、[StarRocks外部テーブル](../data_source/External_table.md#starrocks-external-table)または[Elasticsearch外部テーブル](../data_source/External_table.md#deprecated-elasticsearch-external-table)を作成し、[INSERT](../loading/InsertInto.md)を使用してデータをロードすることを推奨します。

  > **注意**
  >
  > StarRocksの外部テーブルはデータの書き込みのみをサポートし、データの読み取りはサポートしていません。

- MySQLデータベースからデータをロードする場合は、[MySQL外部テーブル](../data_source/External_table.md#deprecated-mysql-external-table)を作成し、[INSERT](../loading/InsertInto.md)を使用してデータをロードすることを推奨します。リアルタイムでデータをロードしたい場合は、[MySQLからのリアルタイム同期](../loading/Flink_cdc_load.md)の指示に従ってデータをロードすることを推奨します。

- Oracle、PostgreSQL、SQL Serverなどの他のデータソースからデータをロードする場合は、[JDBC外部テーブル](../data_source/External_table.md#external-table-for-a-jdbc-compatible-database)を作成し、[INSERT](../loading/InsertInto.md)を使用してデータをロードすることを推奨します。

次の図は、StarRocksがサポートするさまざまなデータソースの概要と、これらのデータソースからデータをロードするために使用できるロード方法を示しています。

![データロードソース](../assets/4.1-3.png)

## メモリ制限

StarRocksでは、各ロードジョブのメモリ使用量を制限するパラメータを提供しており、特に高い同時実行シナリオでのメモリ消費を削減することができます。ただし、過度に低いメモリ使用量の制限を設定しないでください。メモリ使用量の制限が低すぎると、ロードジョブのメモリ使用量が指定した制限に達したため、データが頻繁にメモリからディスクにフラッシュされる可能性があります。ビジネスシナリオに基づいて適切なメモリ使用量の制限を設定することを推奨します。

メモリ使用量を制限するために使用されるパラメータは、ロード方式ごとに異なります。詳細については、[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)、および[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。ロードジョブは通常、複数のBEで実行されるため、パラメータは関与するすべてのBEでのロードジョブの合計メモリ使用量ではなく、各BEでの各ロードジョブのメモリ使用量を制限します。

StarRocksはまた、各個別BEで実行されるすべてのロードジョブの合計メモリ使用量を制限するパラメータも提供しています。詳細については、このトピックの「[システム設定](../loading/Loading_intro.md#system-configurations)」セクションを参照してください。

## 使用上の注意点

### ロード中に宛先カラムを自動的に埋める

データをロードする際、データファイルの特定のフィールドからデータをロードしない選択が可能です。

- StarRocksテーブル作成時にソースフィールドに対応する宛先カラムに`DEFAULT`キーワードを指定している場合、StarRocksは指定されたデフォルト値を宛先カラムに自動的に埋めます。

  [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、および [INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md) は `DEFAULT current_timestamp`、`DEFAULT <default_value>`、および `DEFAULT (<expression>)` をサポートします。[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) は `DEFAULT current_timestamp` と `DEFAULT <default_value>` のみをサポートします。

  > **注記**
  >
  > `DEFAULT (<expression>)` は関数 `uuid()` と `uuid_numeric()` のみをサポートします。

- StarRocks テーブルを作成する際に、ソースフィールドにマッピングされる宛先 StarRocks テーブルの列に `DEFAULT` キーワードを指定しなかった場合、StarRocks は宛先列に `NULL` を自動的に挿入します。

  > **注記**
  >
  > 宛先列が `NOT NULL` として定義されている場合、ロードは失敗します。

  [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、および [Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) では、列マッピングを指定するために使用されるパラメータを使用して、宛先列に入力したい値を指定することもできます。

`NOT NULL` と `DEFAULT` の使用方法については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。

### データロードのための書き込みクォーラムの設定

StarRocks クラスタに複数のデータレプリカがある場合、テーブルごとに異なる書き込みクォーラムを設定できます。つまり、StarRocks がロードタスクが成功したと判断する前にロード成功を返す必要があるレプリカの数です。書き込みクォーラムは、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を実行する際に `write_quorum` プロパティを追加するか、[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)を使用して既存のテーブルにこのプロパティを追加することで指定できます。このプロパティは v2.5 からサポートされています。

## システム設定

このセクションでは、StarRocks が提供するすべてのロード方法に適用可能ないくつかのパラメータ設定について説明します。

### FE 設定

各 FE の設定ファイル **fe.conf** で以下のパラメータを設定できます：

- `max_load_timeout_second` と `min_load_timeout_second`
  
  これらのパラメータは、各ロードジョブの最大タイムアウト期間と最小タイムアウト期間を指定します。タイムアウト期間は秒単位で測定されます。デフォルトの最大タイムアウト期間は3日間、デフォルトの最小タイムアウト期間は1秒です。指定する最大タイムアウト期間と最小タイムアウト期間は、1秒から3日の範囲内である必要があります。これらのパラメータは、同期ロードジョブと非同期ロードジョブの両方に有効です。

- `desired_max_waiting_jobs`
  
  このパラメータは、キューで待機できるロードジョブの最大数を指定します。デフォルト値は **1024** です（v2.4 以前では100、v2.5 以降では1024）。FE の **PENDING** 状態のロードジョブの数が指定した最大数に達すると、FE は新しいロードリクエストを拒否します。このパラメータは非同期ロードジョブのみに有効です。

- `max_running_txn_num_per_db`
  
  このパラメータは、StarRocks クラスタの各データベースで許可される進行中のロードトランザクションの最大数を指定します。ロードジョブは1つ以上のトランザクションを含むことができます。デフォルト値は **100** です。データベースで実行中のロードトランザクションの数が指定した最大数に達すると、提出された後続のロードジョブはスケジュールされません。この状況で同期ロードジョブを提出すると、ジョブは拒否されます。非同期ロードジョブを提出すると、ジョブはキューで待機状態になります。

  > **注記**
  >
  > StarRocks はすべてのロードジョブを一緒にカウントし、同期ロードジョブと非同期ロードジョブを区別しません。

- `label_keep_max_second`
  
  このパラメータは、完了して **FINISHED** または **CANCELLED** 状態のロードジョブの履歴レコードの保持期間を指定します。デフォルトの保持期間は3日間です。このパラメータは同期ロードジョブと非同期ロードジョブの両方に有効です。

### BE 設定

各 BE の設定ファイル **be.conf** で以下のパラメータを設定できます：

- `write_buffer_size`
  
  このパラメータは最大メモリブロックサイズを指定します。デフォルトサイズは100MBです。ロードされたデータは最初にBEのメモリブロックに書き込まれます。ロードされるデータ量が指定した最大メモリブロックサイズに達すると、データはディスクにフラッシュされます。ビジネスシナリオに基づいて適切な最大メモリブロックサイズを指定する必要があります。

  - 最大メモリブロックサイズが極端に小さい場合、BE上に多数の小さなファイルが生成される可能性があります。この場合、クエリパフォーマンスが低下します。生成されるファイルの数を減らすために最大メモリブロックサイズを増やすことができます。
  - 最大メモリブロックサイズが極端に大きい場合、リモートプロシージャコール（RPC）がタイムアウトする可能性があります。この場合、ビジネスニーズに基づいてこのパラメータの値を調整することができます。

- `streaming_load_rpc_max_alive_time_sec`
  
  各 Writer プロセスの待機タイムアウト期間です。デフォルト値は600秒です。データロードプロセス中、StarRocks は Writer プロセスを開始して、各タブレットからデータを受信し、各タブレットにデータを書き込みます。指定した待機タイムアウト期間内に Writer プロセスがデータを受信しない場合、StarRocks は Writer プロセスを停止します。StarRocks クラスタが低速でデータを処理する場合、Writer プロセスが長時間データの次のバッチを受信しないため、「TabletWriter add batch with unknown id」エラーが報告されることがあります。この場合、このパラメータの値を増やすことができます。

- `load_process_max_memory_limit_bytes` と `load_process_max_memory_limit_percent`
  
  これらのパラメータは、各 BE 上のすべてのロードジョブで消費できるメモリの最大量を指定します。StarRocks は、2つのパラメータの値のうち、メモリ消費量が小さい方を許容される最終的なメモリ消費量として識別します。

  - `load_process_max_memory_limit_bytes`：最大メモリサイズを指定します。デフォルトの最大メモリサイズは100GBです。
  - `load_process_max_memory_limit_percent`：最大メモリ使用率を指定します。デフォルト値は30%です。このパラメータは `mem_limit` パラメータとは異なります。`mem_limit` パラメータは、StarRocks クラスタの全体的な最大メモリ使用量を指定し、デフォルト値は90% x 90%です。

    BE が配置されているマシンのメモリ容量が M の場合、ロードジョブで消費できる最大メモリ量は次のように計算されます：`M x 90% x 90% x 30%`。

### システム変数の設定

以下の[システム変数](../reference/System_variable.md)を設定できます：

- `query_timeout`

  クエリのタイムアウト時間。単位：秒。値の範囲：`1` から `259200`。デフォルト値：`300`。この変数は、現在の接続における全てのクエリ文及びINSERT文に適用されます。

## トラブルシューティング

データロードに関するよくある質問については、[FAQ about data loading](../faq/loading/Loading_faq.md)をご覧ください。
