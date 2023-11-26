---
displayed_sidebar: "Japanese"
---

# データローディングの概要

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

データローディングは、ビジネス要件に基づいてさまざまなデータソースからの生データをクレンジングおよび変換し、結果データをStarRocksにロードして高速なデータ分析を実現するプロセスです。

ロードジョブを実行することで、データをStarRocksにロードすることができます。各ロードジョブには、ユーザーによって指定されるか、StarRocksによって自動的に生成される一意のラベルがあります。各ラベルは、1つのロードジョブにのみ使用できます。ロードジョブが完了した後、そのラベルは他のロードジョブで再利用することはできません。失敗したロードジョブのラベルのみが再利用できます。このメカニズムにより、特定のラベルに関連付けられたデータは1回だけロードされることが保証され、At-Most-Onceセマンティクスが実装されます。

StarRocksが提供するすべてのローディングメソッドは、アトミック性を保証できます。アトミック性とは、ロードジョブ内の有資格データがすべて正常にロードされるか、有資格データのいずれかが正常にロードされないことを意味します。有資格データの一部がロードされ、他のデータがロードされないことはありません。なお、有資格データには、データ型変換エラーなどの品質問題によってフィルタリングされたデータは含まれません。

StarRocksは、ロードジョブを送信するために使用できる2つの通信プロトコル、MySQLとHTTPをサポートしています。各ローディングメソッドがサポートするプロトコルについての詳細は、このトピックの[ローディングメソッド](../loading/Loading_intro.md#loading-methods)セクションを参照してください。

<InsertPrivNote />

## サポートされるデータ型

StarRocksは、すべてのデータ型のデータをロードすることができます。ただし、一部の特定のデータ型のローディングに制限があることに注意してください。詳細については、[データ型](../sql-reference/sql-statements/data-types/BIGINT.md)を参照してください。

## ローディングモード

StarRocksは、同期ローディングモードと非同期ローディングモードの2つのローディングモードをサポートしています。

> **注意**
>
> 外部プログラムを使用してデータをロードする場合は、ローディングモードを選択してから、選択したローディングメソッドを決定する前に、ビジネス要件に最適なローディングモードを選択する必要があります。

### 同期ローディング

同期ローディングモードでは、ロードジョブを送信した後、StarRocksはジョブを同期的に実行してデータをロードし、ジョブが完了した後にジョブの結果を返します。ジョブの結果に基づいてジョブが成功したかどうかを確認することができます。

StarRocksは、同期ローディングをサポートする2つのローディングメソッドを提供しています：[Stream Load](../loading/StreamLoad.md)および[INSERT](../loading/InsertInto.md)。

同期ローディングのプロセスは次のとおりです。

1. ロードジョブを作成します。

2. StarRocksが返すジョブの結果を表示します。

3. ジョブの結果に基づいてジョブが成功したかどうかを確認します。ジョブの結果がロードの失敗を示す場合、ジョブを再試行することができます。

### 非同期ローディング

非同期ローディングモードでは、ロードジョブを送信した後、StarRocksはジョブの作成結果を即座に返します。

- 結果がジョブの作成成功を示す場合、StarRocksはジョブを非同期に実行します。ただし、これはデータが正常にロードされたことを意味するわけではありません。ジョブのステータスに基づいてデータが正常にロードされたかどうかを確認するために、ステートメントやコマンドを使用する必要があります。

- 結果がジョブの作成失敗を示す場合、失敗情報に基づいてジョブを再試行する必要があるかどうかを判断することができます。

StarRocksは、非同期ローディングをサポートする3つのローディングメソッドを提供しています：[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、および[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)。

非同期ローディングのプロセスは次のとおりです。

1. ロードジョブを作成します。

2. StarRocksが返すジョブの作成結果を表示し、ジョブが正常に作成されたかどうかを判断します。
   a. ジョブの作成が成功した場合、ステップ3に進みます。
   b. ジョブの作成に失敗した場合、ステップ1に戻ります。

3. ジョブのステータスが**FINISHED**または**CANCELLED**を示すまで、ステートメントやコマンドを使用してジョブのステータスを確認します。

Broker LoadまたはSpark Loadジョブのワークフローは、次の図に示すように、5つのステージで構成されています。

![Broker LoadまたはSpark Loadのワークフロー](../assets/4.1-1.png)

ワークフローは次のように説明されます。

1. **PENDING**

   ジョブは、FEによってスケジュールされるのを待つキューにあります。

2. **ETL**

   FEは、クレンジング、パーティショニング、ソート、集計など、データの事前処理を行います。
   > **注意**
   >
   > ジョブがBroker Loadジョブの場合、このステージは直接完了します。

3. **LOADING**

   FEはデータをクレンジングおよび変換し、データをBEに送信します。すべてのデータがロードされた後、データはキューに入って有効になるのを待ちます。この時点で、ジョブのステータスは**LOADING**のままです。

4. **FINISHED**

   すべてのデータが有効になると、ジョブのステータスは**FINISHED**になります。この時点で、データをクエリできます。**FINISHED**は最終的なジョブの状態です。

5. **CANCELLED**

   ジョブのステータスが**FINISHED**になる前に、いつでもジョブをキャンセルすることができます。また、ロードエラーが発生した場合、StarRocksはジョブを自動的にキャンセルすることもできます。ジョブがキャンセルされると、ジョブのステータスは**CANCELLED**になります。**CANCELLED**も最終的なジョブの状態です。

Routineジョブのワークフローは次のように説明されます。

1. MySQLクライアントからFEにジョブが送信されます。

2. FEはジョブを複数のタスクに分割します。各タスクは、複数のパーティションからデータをロードするために設計されています。

3. FEはタスクを指定されたBEに配布します。

4. BEはタスクを実行し、タスクの実行が完了した後にFEに報告します。

5. FEは、BEからのレポートに基づいて、次のタスクを生成し、失敗したタスクを再試行し、またはタスクのスケジューリングを一時停止します。

## ローディングメソッド

StarRocksは、さまざまなビジネスシナリオでデータをロードするための5つのローディングメソッドを提供しています：[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)、および[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

| ローディングメソッド | データソース | ビジネスシナリオ | ロードジョブごとのデータ量 | データファイル形式 | ローディングモード | プロトコル |
| ------------------ | -------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ----------------------------------------------- | ------------ | -------- |
| Stream Load        |  <ul><li>ローカルファイル</li><li>データストリーム</li></ul>| ローカルファイルシステムからデータファイルをロードするか、プログラムを使用してデータストリームをロードします。 | 10 GB以下                             |<ul><li>CSV</li><li>JSON</li></ul>               | 同期  | HTTP     |
| Broker Load        | <ul><li>HDFS</li><li>Amazon S3</li><li>Google GCS</li><li>Microsoft Azure Storage</li><li>Alibaba Cloud OSS</li><li>Tencent Cloud COS</li><li>Huawei Cloud OBS</li><li>その他のS3互換ストレージシステム（MinIOなど）</li></ul>| HDFSまたはクラウドストレージからデータをロードします。                        | 数十GBから数百GBまで                               | <ul><li>CSV</li><li>Parquet</li><li>ORC</li></ul>| 非同期 | MySQL    |
| Routine Load       | Apache Kafka®                                       | Kafkaからリアルタイムにデータをロードします。                   | MBからGBのデータ（ミニバッチ）                           |<ul><li>CSV</li><li>JSON</li><li>Avro（v3.0.1以降でサポート）</li></ul>          | 非同期 | MySQL    |
| Spark Load         | <ul><li>HDFS</li><li>Hive</li></ul>     |<ul><li>HDFSまたはHiveから大量のデータをApache Spark™クラスタを使用して移行します。</li><li>重複排除のためにグローバルデータディクショナリを使用しながらデータをロードします。</li></ul>| 数十GBから数TBまで                                         |<ul><li>CSV</li><li>ORC（v2.0以降でサポート）</li><li>Parquet（v2.0以降でサポート）</li></ul>       | 非同期 | MySQL    |
| INSERT INTO SELECT | <ul><li>StarRocksテーブル</li><li>外部テーブル</li><li>AWS S3</li></ul>**注意**<br />AWS S3からデータをロードする場合、Parquet形式またはORC形式のファイルのみサポートされます。     |<ul><li>外部テーブルからデータをロードします。</li><li>StarRocksテーブル間でデータをロードします。</li></ul>| 固定されていません（データ量はメモリサイズに基づいて変動します。） | StarRocksテーブル      | 同期  | MySQL    |
| INSERT INTO VALUES | <ul><li>プログラム</li><li>ETLツール</li></ul>    |<ul><li>個々のレコードとして少量のデータを挿入します。</li><li>JDBCなどのAPIを使用してデータをロードします。</li></ul>| 少量のデータを挿入します。                                          | SQL                   | 同期  | MySQL    |

ビジネスシナリオ、データ量、データソース、データファイル形式、およびローディング頻度に基づいて、選択したローディングメソッドを決定することができます。また、ローディングメソッドを選択する際には、次の点に注意してください。

- Kafkaからデータをロードする場合は、[Routine Load](../loading/RoutineLoad.md)を使用することをおすすめします。ただし、データが複数のテーブルの結合および抽出、変換、ロード（ETL）操作を必要とする場合は、Apache Flink®を使用してKafkaからデータを読み取り、前処理し、[flink-connector-starrocks](../loading/Flink-connector-starrocks.md)を使用してデータをStarRocksにロードすることができます。

- Hive、Iceberg、Hudi、またはDelta Lakeからデータをロードする場合は、[Hiveカタログ](../data_source/catalog/hive_catalog.md)、[Icebergカタログ](../data_source/catalog/iceberg_catalog.md)、[Hudiカタログ](../data_source/catalog/hudi_catalog.md)、または[Delta Lakeカタログ](../data_source/catalog/deltalake_catalog.md)を作成し、[INSERT](../loading/InsertInto.md)を使用してデータをロードすることをおすすめします。

- 別のStarRocksクラスタまたはElasticsearchクラスタからデータをロードする場合は、[StarRocks外部テーブル](../data_source/External_table.md#starrocks-external-table)または[Elasticsearch外部テーブル](../data_source/External_table.md#deprecated-elasticsearch-external-table)を作成し、[INSERT](../loading/InsertInto.md)を使用してデータをロードすることをおすすめします。

  > **注意**
  >
  > StarRocks外部テーブルはデータの書き込みのみをサポートしており、データの読み取りはサポートされていません。

- MySQLデータベースからデータをロードする場合は、[MySQL外部テーブル](../data_source/External_table.md#deprecated-mysql-external-table)を作成し、[INSERT](../loading/InsertInto.md)を使用してデータをロードすることをおすすめします。リアルタイムでデータをロードする場合は、[MySQLからのリアルタイム同期](../loading/Flink_cdc_load.md)の手順に従ってデータをロードすることをおすすめします。

- Oracle、PostgreSQL、SQL Serverなどの他のデータソースからデータをロードする場合は、[JDBC外部テーブル](../data_source/External_table.md#external-table-for-a-jdbc-compatible-database)を作成し、[INSERT](../loading/InsertInto.md)を使用してデータをロードすることをおすすめします。

次の図は、StarRocksがサポートするさまざまなデータソースと、これらのデータソースからデータをロードするために使用できるローディングメソッドの概要を示しています。

![データローディングソース](../assets/4.1-3.png)

## メモリ制限

StarRocksは、各ロードジョブのメモリ使用量を制限するためのパラメータを提供しており、特に高並行性のシナリオではメモリ消費を削減することができます。ただし、メモリ使用量の制限を過度に低く指定しないでください。メモリ使用量の制限が過度に低い場合、ロードジョブのメモリ使用量が指定された制限に達するたびにデータが頻繁にメモリからディスクにフラッシュされる可能性があります。ビジネスシナリオに基づいて適切なメモリ使用量制限を指定することをおすすめします。

メモリ使用量を制限するために使用されるパラメータは、各ローディングメソッドごとに異なります。詳細については、[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)、および[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。なお、ロードジョブは通常、複数のBEで実行されます。したがって、パラメータは、関与する各BEの各ロードジョブのメモリ使用量を制限するものであり、すべての関与するBEでのロードジョブの総メモリ使用量を制限するものではありません。

StarRocksは、各個別のBEで実行されるすべてのロードジョブの総メモリ使用量を制限するためのパラメータも提供しています。詳細については、このトピックの"[システム設定](../loading/Loading_intro.md#system-configurations)"セクションを参照してください。

## 使用上の注意

### ロード時に宛先の列を自動的に補完する

データをロードする際、データファイルの特定のフィールドからデータをロードしないように選択することができます。

- StarRocksテーブルの宛先列にソースフィールドをマッピングする際に、`DEFAULT`キーワードを指定した場合、StarRocksは指定されたデフォルト値を宛先列に自動的に補完します。

  [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、および[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)は、`DEFAULT current_timestamp`、`DEFAULT <default_value>`、および`DEFAULT (<expression>)`をサポートしています。[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)は、`DEFAULT current_timestamp`と`DEFAULT <default_value>`のみをサポートしています。

  > **注意**
  >
  > `DEFAULT (<expression>)`は、`uuid()`および`uuid_numeric()`の関数のみをサポートしています。

- StarRocksテーブルの宛先列にソースフィールドをマッピングする際に、`DEFAULT`キーワードを指定しなかった場合、StarRocksは宛先列に`NULL`を自動的に補完します。

  > **注意**
  >
  > 宛先列が`NOT NULL`として定義されている場合、ロードは失敗します。

  [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、および[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)では、宛先列に補完する値をパラメータを使用して指定することもできます。

`NOT NULL`および`DEFAULT`の使用方法についての詳細は、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。

### データローディングのための書き込みクォーラムの設定

StarRocksクラスタに複数のデータレプリカがある場合、テーブルごとに異なる書き込みクォーラムを設定できます。つまり、StarRocksがローディングタスクが成功したと判断するために必要なレプリカの数を指定できます。テーブルを作成する際に、プロパティ`write_quorum`を追加するか、[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)を使用して既存のテーブルにこのプロパティを追加することで、書き込みクォーラムを指定できます。このプロパティはv2.5以降でサポートされています。

## システム設定

このセクションでは、StarRocksが提供するすべてのローディングメソッドに適用されるいくつかのパラメータ設定について説明します。

### FEの設定

各FEの設定ファイル**fe.conf**で、次のパラメータを設定できます。

- `max_load_timeout_second`および`min_load_timeout_second`
  
  これらのパラメータは、各ロードジョブの最大タイムアウト期間と最小タイムアウト期間を指定します。タイムアウト期間は秒単位で測定されます。デフォルトの最大タイムアウト期間は3日間で、デフォルトの最小タイムアウト期間は1秒です。指定する最大タイムアウト期間と最小タイムアウト期間は、1秒から3日の範囲内にある必要があります。これらのパラメータは、同期ロードジョブと非同期ロードジョブの両方に対して有効です。

- `desired_max_waiting_jobs`
  
  このパラメータは、キューで保持できる最大のロードジョブ数を指定します。デフォルト値は**1024**（v2.4以前は100、v2.5以降は1024）です。FE上の**PENDING**状態のロードジョブの数が指定した最大数に達すると、FEは新しいロードリクエストを拒否します。このパラメータは非同期ロードジョブのみに有効です。

- `max_running_txn_num_per_db`
  
  このパラメータは、StarRocksクラスタの各データベースで許可される実行中のロードトランザクションの最大数を指定します。ロードジョブには1つ以上のトランザクションが含まれる場合があります。デフォルト値は**100**です。データベースで実行されているロードトランザクションの数が指定した最大数に達すると、送信する後続のロードジョブはスケジュールされません。この状況では、同期ロードジョブを送信すると、ジョブが拒否されます。非同期ロードジョブを送信すると、ジョブはキューで待機します。

  > **注意**
  >
  > StarRocksは、すべてのロードジョブをカウントし、同期ロードジョブと非同期ロードジョブを区別しません。

- `label_keep_max_second`
  
  このパラメータは、完了し**FINISHED**または**CANCELLED**状態にあるロードジョブの履歴レコードの保持期間を指定します。デフォルトの保持期間は3日間です。このパラメータは、同期ロードジョブと非同期ロードジョブの両方に対して有効です。

### BEの設定

各BEの設定ファイル**be.conf**で、次のパラメータを設定できます。

- `write_buffer_size`
  
  このパラメータは、最大メモリブロックサイズを指定します。デフォルトのサイズは100MBです。ロードされたデータは、まずBE上のメモリブロックに書き込まれます。ロードされたデータの量が指定した最大メモリブロックサイズに達すると、データはディスクにフラッシュされます。ビジネスシナリオに基づいて適切な最大メモリブロックサイズを指定する必要があります。

  - 最大メモリブロックサイズが極端に小さい場合、BE上に多数の小さなファイルが生成される可能性があります。この場合、クエリのパフォーマンスが低下します。生成されるファイルの数を減らすために、最大メモリブロックサイズを増やすことができます。
  - 最大メモリブロックサイズが極端に大きい場合、リモートプロシージャコール（RPC）がタイムアウトする可能性があります。この場合、ビジネスニーズに基づいてこのパラメータの値を調整することができます。

- `streaming_load_rpc_max_alive_time_sec`
  
  各Writerプロセスの待機タイムアウト期間です。デフォルト値は600秒です。データローディングプロセス中、StarRocksは各タブレットからデータを受信し、データを書き込むために各タブレットに対してWriterプロセスを起動します。指定した待機タイムアウト期間内にWriterプロセスがデータを受信しない場合、StarRocksはWriterプロセスを停止します。StarRocksクラスタが低速でデータを処理する場合、Writerプロセスは長時間次のデータバッチを受信せず、「TabletWriter add batch with unknown id」エラーを報告する場合があります。この場合、このパラメータの値を増やすことができます。

- `load_process_max_memory_limit_bytes`および`load_process_max_memory_limit_percent`
  
  これらのパラメータは、各個別のBE上で実行されるすべてのロードジョブのメモリ使用量の最大値を指定します。StarRocksは、2つのパラメータの値のうち小さい方を最終的な許容メモリ使用量として識別します。

  - `load_process_max_memory_limit_bytes`：最大メモリサイズを指定します。デフォルトの最大メモリサイズは100GBです。
  - `load_process_max_memory_limit_percent`：最大メモリ使用量を指定します。デフォルト値は30%です。このパラメータは`mem_limit`パラメータとは異なります。`mem_limit`パラメータはStarRocksクラスタの総最大メモリ使用量を指定し、デフォルト値は90% x 90%です。

    BEが搭載されているマシンのメモリ容量をMとすると、ロードジョブのメモリ使用量の最大値は次のように計算されます：`M x 90% x 90% x 30%`。

### システム変数の設定

次の[システム変数](../reference/System_variable.md)を設定できます。

- `query_timeout`

  クエリのタイムアウト期間です。単位：秒。値の範囲：`1`から`259200`。デフォルト値：`300`。この変数は、現在の接続のすべてのクエリステートメント、およびINSERTステートメントに対して有効になります。

## トラブルシューティング

詳細については、[データローディングに関するFAQ](../faq/loading/Loading_faq.md)を参照してください。
