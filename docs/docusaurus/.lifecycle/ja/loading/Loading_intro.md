---
displayed_sidebar: "Japanese"
---

# データロードの概要

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

データロードは、ビジネス要件に基づいて様々なデータソースからの生データをクレンジングおよび変換し、その結果をStarRocksにロードして、超高速のデータ分析を可能にするプロセスです。

データはロードジョブを実行することでStarRocksにロードすることができます。各ロードジョブには、ユーザーによって指定された一意のラベルがあります。また、StarRocksによって自動的に生成されることもあります。各ラベルは一つのロードジョブにのみ使用できます。ロードジョブが完了した後、そのラベルは他のロードジョブで再利用することはできません。失敗したロードジョブのラベルのみが再利用可能です。この仕組みにより、特定のラベルに関連付けられたデータは一度だけしかロードされないようになり、At-Most-Onceセマンティクスが実装されます。

StarRocksが提供するすべてのロード方法は原子性を保証します。原子性とは、ロードジョブ内の認定されたデータはすべて正常にロードされるか、または認定されたデータが一部ロードされ一部がロードされないことは絶対に起こらないことを意味します。なお、認定されたデータにはデータ型変換エラーなどの品質の問題によってフィルタリングされたデータは含まれません。

StarRocksは、ロードジョブを提出するために使用できる2つの通信プロトコル、MySQLおよびHTTPをサポートしています。各ロード方法でサポートされるプロトコルに関する詳細については、このトピックの[ロード方法](../loading/Loading_intro.md#loading-methods)セクションを参照してください。

<InsertPrivNote />

## サポートされるデータ型

StarRocksはすべてのデータ型のデータをロードすることができます。ただし、特定のデータ型のロードには制限があることに注意する必要があります。詳細については、[データ型](../sql-reference/sql-statements/data-types/BIGINT.md)を参照してください。 

## ロードモード

StarRocksは同期ロードモードと非同期ロードモードの2つのロードモードをサポートしています。

> **注意**
>
> 外部プログラムを使用してデータをロードする場合は、ロード方法を選択する前に、ビジネス要件に最適なロードモードを選択する必要があります。

### 同期ロード

同期ロードモードでは、ロードジョブを提出した後、StarRocksはジョブを同期的に実行してデータをロードし、ジョブが完了した後にジョブの結果を返します。ジョブの結果に基づいてジョブが成功したかどうかを確認することができます。

StarRocksは同期ロードをサポートする2つのロード方法、[Stream Load](../loading/StreamLoad.md)および[INSERT](../loading/InsertInto.md)を提供しています。

同期ロードのプロセスは以下の通りです。

1. ロードジョブを作成する。

2. StarRocksによって返されるジョブの結果を表示する。

3. ジョブの結果に基づいてジョブが成功したかどうかをチェックする。ジョブの結果がロードの失敗を示す場合、ジョブを再試行することができます。

### 非同期ロード

非同期ロードモードでは、ロードジョブを提出した後、StarRocksは即座にジョブの作成結果を返します。

- ジョブ作成が成功した場合、StarRocksは非同期的にジョブを実行します。ただし、これはデータが正常にロードされたことを意味するものではありません。ステートメントやコマンドを使用してジョブのステータスをチェックする必要があります。その後、ジョブのステータスに基づいてデータが正常にロードされたかどうかを決定することができます。

- ジョブ作成が失敗した場合、失敗情報に基づいてジョブの再試行が必要かどうかを決定することができます。

StarRocksは非同期ロードをサポートする3つのロード方法、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、および[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)を提供しています。

非同期ロードのプロセスは以下の通りです。

1. ロードジョブを作成する。

2. StarRocksによって返されるジョブの作成結果を表示し、ジョブが正常に作成されたかどうかを決定する。
   a. ジョブの作成が成功した場合、Step 3に進みます。
   b. ジョブの作成が失敗した場合、Step 1に戻ります。

3. ステートメントやコマンドを使用してジョブのステータスを確認し、ジョブのステータスが**FINISHED**または**CANCELLED**を示すまで状態を確認します。

Broker LoadまたはSpark Loadジョブのワークフローは、次の図に示す通り5つのステージで構成されています。

![ブローカーロードまたはスパークロードオーバーフロー](../assets/4.1-1.png)

ワークフローは以下のように説明されます。

1. **PENDING**

   ジョブはFEによってスケジュールされるのを待っているキューにあります。

2. **ETL**

   FEは、データの前処理（クレンジング、パーティショニング、ソーティング、集計など）を含む、前処理を行います。
   > **注意**
   >
   > ジョブがBroker Loadジョブの場合、このステージは直ちに完了します。

3. **LOADING**

   FEはデータのクレンジングと変換を行い、その後データをBEに送信します。すべてのデータがロードされた後、データは有効となるまでキューにあります。この時点で、ジョブのステータスは**LOADING**のままです。

4. **FINISHED**

   すべてのデータが有効になると、ジョブのステータスは**FINISHED**になります。この時点でデータをクエリできます。**FINISHED**は最終的なジョブの状態です。

5. **CANCELLED**

   ジョブのステータスが**FINISHED**になる前に、いつでもジョブをキャンセルすることができます。また、ロードエラーが発生した場合、StarRocksは自動的にジョブをキャンセルすることができます。ジョブがキャンセルされた後、ジョブのステータスは**CANCELLED**になります。**CANCELLED**もまた、最終的なジョブの状態です。

Routineジョブのワークフローは以下のように説明されます。

1. ジョブはMySQLクライアントからFEに提出されます。

2. FEはジョブを複数のタスクに分割します。各タスクは複数のパーティションからデータをロードするためのものです。

3. FEはタスクを指定されたBEに配布します。

4. BEはタスクを実行し、タスクの完了後にFEに報告します。

5. FEはBEからの報告に基づいて、後続のタスクを生成し、失敗したタスクがあれば再試行したり、タスクのスケジューリングを一時停止したりします。

## ロード方法

StarRocksは、さまざまなビジネスシナリオでデータをロードするための5つのロード方法、[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)、および[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を提供しています。

| ロード方法       | データソース                                    | ビジネスシナリオ                                     | 1つのロードジョブあたりのデータボリューム                             | データファイル形式                                   | ロードモード  | プロトコル |
| --------------- | ------------------------------------------ | ----------------------------------------------------| ----------------------------------------------------------------- | ----------------------------------------------- | ------------ | -------- |
| Stream Load     |  <ul><li>ローカルファイル</li><li>データストリーム</li></ul>| ローカルファイルシステムからデータファイルをロードしたり、プログラムを使用してデータストリームをロードしたりします。   | 10 GB以下            |<ul><li>CSV</li><li>JSON</li></ul>               | 同期         | HTTP     |
| Broker Load     | <ul><li>HDFS</li><li>Amazon S3</li><li>Google GCS</li><li>Microsoft Azure Storage</li><li>Alibaba Cloud OSS</li><li>Tencent Cloud COS</li><li>Huawei Cloud OBS</li><li>その他のS3互換ストレージシステム（MinIOなど）</li></ul>| HDFSやクラウドストレージからデータをロードします。                          | 数十GBから数百GB                                | <ul><li>CSV</li><li>Parquet</li><li>ORC</li></ul>| 非同期         | MySQL    |
| Routine Load    | Apache Kafka®                         | Kafkaからリアルタイムでデータをロードします。                            | MBからGB単位のデータ（ミニバッチ）                                   |<ul><li>CSV</li><li>JSON</li><li>Avro（v3.0.1以降でサポート）</li></ul>        | 非同期         | MySQL    |
| Spark Load      | <ul><li>HDFS</li><li>Hive</li></ul>  |<ul><li>HDFSやHiveから大容量のデータをApache Spark™クラスターを使用して移行します。</li><li>グローバルデータ辞書を使用してデータの重複排除を行いながらデータをロードします。</li></ul>| 数十GBから数TB                                      |<ul><li>CSV</li><li>ORC（v2.0以降でサポート）</li><li>Parquet（v2.0以降でサポート）</li></ul>       | 非同期         | MySQL    |
| INSERT INTO SELECT | <ul><li>StarRocksテーブル</li><li>外部テーブル</li><li>AWS S3</li></ul>**注意**<br />AWS S3からデータをロードする際は、Parquet形式またはORC形式のファイルのみがサポートされています。|<ul><li>外部テーブルからデータをロードします。</li><li>StarRocksテーブル間でデータをロードします。 </li></ul>| 固定されていません（メモリサイズに基づいてデータボリュームが変動します。） | StarRocksテーブル      | 同期         | MySQL    |
| INSERT INTO VALUES | <ul><li>Programs</li><li>ETL tools</li></ul>    |<ul><li>個々のレコードとして少量のデータを挿入します。</li><li>JDBCなどのAPIを使用してデータをロードします。</li></ul>| 少量のデータを                                          | SQL                   | 同期    | MySQL    |

ビジネスシナリオ、データ量、データソース、データファイル形式、およびロード頻度に基づいて、選択したロードメソッドを決定できます。また、次の点に注意してロードメソッドを選択する際に注意してください。

- Kafkaからデータをロードする場合、[Routine Load](../loading/RoutineLoad.md)を使用することをお勧めします。ただし、データが複数のテーブルの結合やエクストラクト、トランスフォーム、ロード（ETL）操作が必要な場合は、Apache Flink®を使用してKafkaからデータを読み取り、事前処理してから[flink-connector-starrocks](../loading/Flink-connector-starrocks.md)を使用してデータをStarRocksにロードすることができます。

- Hive、Iceberg、Hudi、またはDelta Lakeからデータをロードする場合、[Hiveカタログ](../data_source/catalog/hive_catalog.md)、[Icebergカタログ](../data_source/catalog/iceberg_catalog.md)、[Hudiカタログ](../data_source/catalog/hudi_catalog.md)、または[Delta Lakeカタログ](../data_source/catalog/deltalake_catalog.md)を作成し、[INSERT](../loading/InsertInto.md)を使用してデータをロードすることをお勧めします。

- 別のStarRocksクラスターまたはElasticsearchクラスターからデータをロードする場合、[StarRocks外部テーブル](../data_source/External_table.md#starrocks-external-table)または[Elasticsearch外部テーブル](../data_source/External_table.md#deprecated-elasticsearch-external-table)を作成し、[INSERT](../loading/InsertInto.md)を使用してデータをロードすることをお勧めします。

  > **注意**
  >
  > StarRocks外部テーブルはデータの書き込みのみをサポートします。データの読み込みはサポートされていません。

- MySQLデータベースからデータをロードする場合、[MySQL外部テーブル](../data_source/External_table.md#deprecated-mysql-external-table)を作成し、[INSERT](../loading/InsertInto.md)を使用してデータをロードすることをお勧めします。リアルタイムでデータをロードする場合は、[MySQLからのリアルタイム同期](../loading/Flink_cdc_load.md)で提供される手順に従ってデータをロードすることをお勧めします。

- Oracle、PostgreSQL、およびSQL Serverなどの他のデータソースからデータをロードする場合、[JDBC外部テーブル](../data_source/External_table.md#external-table-for-a-jdbc-compatible-database)を作成し、[INSERT](../loading/InsertInto.md)を使用してデータをロードすることをお勧めします。

以下の図は、StarRocksがサポートするさまざまなデータソースと、これらのデータソースからデータをロードするために使用できるロード方法の概要を示しています。

![データローディングソース](../assets/4.1-3.png)

## メモリ制限

StarRocksは、各ロードジョブのメモリ使用量を制限するためのパラメータを提供しており、これによりメモリ消費を特に高い同時実行シナリオで削減します。ただし、極端に低いメモリ使用量制限を指定しないでください。メモリ使用量制限が極端に低い場合、ロードジョブのメモリ使用量が指定された制限に達するたびにデータが頻繁にメモリからディスクにフラッシュされる可能性があります。ビジネスシナリオに基づいて適切なメモリ使用量制限を指定することをお勧めします。

メモリ使用量を制限するために使用されるパラメータは、各ロードメソッドごとに異なります。詳細については、[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)、および[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。ロードジョブは通常、複数のBEで実行されます。したがって、パラメータはすべての関連するBE上の各ロードジョブのメモリ使用量を制限します。

StarRocksは、個々のBEで実行されるすべてのロードジョブの合計メモリ使用量を制限するためのパラメータも提供しています。詳細については、このトピックの「[システム設定](../loading/Loading_intro.md#system-configurations)」セクションを参照してください。

## 使用上の注意

### ロード中に宛先カラムを自動的に埋める

データをロードする際、データファイルの特定のフィールドからデータをロードしないように選択できます。

- StarRocksテーブルを作成する際に、ソースフィールドにマッピングする宛先StarRocksテーブルカラムに`DEFAULT`キーワードを指定した場合、StarRocksは指定されたデフォルト値を自動的に宛先カラムに埋めます。

  [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、および[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)は、`DEFAULT current_timestamp`、`DEFAULT <default_value>`、および`DEFAULT (<expression>)`をサポートしています。[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)は`DEFAULT current_timestamp`と`DEFAULT <default_value>`のみをサポートしています。

  > **注意**
  >
  > `DEFAULT (<expression>)`は、`uuid()`および`uuid_numeric()`の機能のみをサポートしています。

- StarRocksテーブルを作成する際に、ソースフィールドにマッピングする宛先StarRocksテーブルカラムに`DEFAULT`キーワードを指定しなかった場合、StarRocksは自動的に宛先カラムに`NULL`を埋めます。

  > **注意**
  >
  > 宛先カラムが`NOT NULL`として定義されている場合、ロードは失敗します。

  [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、および[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)では、宛先カラムに埋める値を指定するために、列マッピングを指定するために使用されるパラメータを使用することもできます。

`NOT NULL`および`DEFAULT`の使用についての詳細については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。

### データローディング用の書き込みクォーラムの設定

StarRocksクラスターに複数のデータレプリカがある場合、テーブルごとに異なる書き込みクォーラムを設定できます。つまり、StarRocksがローディング成功を返すために必要なレプリカの数を指定できます。このプロパティは、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を使用してこのプロパティを追加するか、このプロパティを使用して既存のテーブルにプロパティを追加することで指定できます。このプロパティはv2.5からサポートされています。

## システム設定

このセクションでは、StarRocksが提供するすべてのロードメソッドに適用されるいくつかのパラメータ構成について説明します。

### FEの構成

各FEの構成ファイル**fe.conf**で、次のパラメータを構成できます。

- `max_load_timeout_second`と`min_load_timeout_second`
  
  これらのパラメータは、各ロードジョブの最大タイムアウト期間と最小タイムアウト期間を指定します。タイムアウト期間は秒単位で測定されます。デフォルトの最大タイムアウト期間は3日間で、デフォルトの最小タイムアウト期間は1秒です。指定する最大タイムアウト期間と最小タイムアウト期間は、1秒から3日の範囲内になければなりません。これらのパラメータは、同期ロードジョブと非同期ロードジョブの両方に有効です。

- `desired_max_waiting_jobs`
  
  このパラメータは、キューで保持できるロードジョブの最大数を指定します。デフォルト値は**1024**です（v2.4およびそれ以前で100、v2.5およびそれ以降で1024）。FE上の**PENDING**状態のロードジョブの数が指定した最大数に達すると、FEは新しいロードリクエストを拒否します。このパラメータは、非同期ロードジョブにのみ有効です。

- `max_running_txn_num_per_db`
  
  このパラメータは、StarRocksクラスターの各データベースで許可される実行中のロードトランザクションの最大数を指定します。ロードジョブには1つ以上のトランザクションが含まれることができます。デフォルト値は**100**です。データベースで実行されているロードトランザクションの数が指定した最大数に達すると、送信される後続のロードジョブはスケジュールされません。この状況では、同期ロードジョブを送信すると、ジョブが拒否されます。非同期ロードジョブを送信すると、ジョブはキューで保持されます。

  > **注意**
  >
  > StarRocksは、すべてのロードジョブを一括してカウントし、同期ロードジョブと非同期ロードジョブを区別しません。

- `label_keep_max_second`
  
  このパラメータは、完了して**FINISHED**または**CANCELLED**の状態にあるロードジョブの履歴レコードの保持期間を指定します。デフォルトの保持期間は3日間です。このパラメータは、同期ロードジョブと非同期ロードジョブの両方に有効です。

### BEの構成

各BEの構成ファイル**be.conf**で、次のパラメータを構成できます。

- `write_buffer_size`
  
このパラメータは、最大メモリブロックサイズを指定します。デフォルトサイズは100 MBです。ロードされたデータはまずBEのメモリブロックに書き込まれます。指定した最大メモリブロックサイズに達すると、データはディスクにフラッシュされます。ビジネスシナリオに基づいて適切な最大メモリブロックサイズを指定する必要があります。

- 最大メモリブロックサイズが極端に小さい場合、大量の小さなファイルがBEに生成される可能性があります。この場合、クエリのパフォーマンスが低下します。生成されるファイル数を減らすには、最大メモリブロックサイズを増やすことができます。
- 最大メモリブロックサイズが極端に大きい場合、リモートプロシージャコール（RPC）がタイムアウトする可能性があります。この場合、ビジネスニーズに基づいてこのパラメータの値を調整できます。

- `streaming_load_rpc_max_alive_time_sec`
  
  各Writerプロセスの待機タイムアウト期間を指定します。デフォルト値は600秒です。データのロードプロセス中、StarRocksは各タブレットからデータを受信して書き込むためにWriterプロセスを開始します。指定した待機タイムアウト期間内にWriterプロセスがデータを受信しない場合、StarRocksはWriterプロセスを停止します。StarRocksクラスタが低速でデータを処理する場合、Writerプロセスは長時間次のデータバッチを受信できず、「TabletWriter add batch with unknown id」エラーが発生する可能性があります。この場合、このパラメータの値を増やすことができます。

- `load_process_max_memory_limit_bytes` および `load_process_max_memory_limit_percent`
  
  これらのパラメータは、各個別のBE上で実行されるすべてのロードジョブに消費される最大メモリ量を指定します。StarRocksは、これら2つのパラメータの値のうちより小さいメモリ消費を最終的な許可されたメモリ消費として識別します。

  - `load_process_max_memory_limit_bytes`: 最大メモリサイズを指定します。デフォルトの最大メモリサイズは100 GBです。
  - `load_process_max_memory_limit_percent`: 最大メモリ使用率を指定します。デフォルト値は30%です。このパラメータは `mem_limit` パラメータとは異なります。`mem_limit` パラメータはStarRocksクラスタの総合的な最大メモリ使用量を指定し、デフォルト値は90% x 90%です。

    BEが配置されているマシンのメモリ容量をMとすると、ロードジョブに消費される最大メモリ量は次のように計算されます: `M x 90% x 90% x 30%`。

### システム変数の設定

次の[システム変数](../reference/System_variable.md)を設定できます:

- `query_timeout`

  クエリのタイムアウト期間です。単位: 秒。値の範囲: `1` から `259200`。デフォルト値: `300`。この変数は現在の接続に対するすべてのクエリステートメントおよびINSERTステートメントに影響を与えます。

## トラブルシューティング

詳細については、[データロードのFAQ](../faq/loading/Loading_faq.md)を参照してください。