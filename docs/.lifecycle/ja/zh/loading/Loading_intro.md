---
displayed_sidebar: Chinese
---

# インポート概要

データインポートとは、原始データをビジネス要件に従ってクリーニング、変換し、StarRocksにロードするプロセスであり、これによりStarRocksシステムで高速かつ統一されたデータ分析を行うことができます。

StarRocksはインポートジョブを通じてデータインポートを実現します。各インポートジョブにはラベル（Label）があり、ユーザーが指定するかシステムが自動生成するもので、そのインポートジョブを識別するために使用されます。各ラベルはデータベース内でユニークであり、成功したインポートジョブにのみ使用できます。インポートジョブが成功すると、そのラベルは他のインポートジョブを提出するために再利用することはできません。失敗したインポートジョブのラベルのみが他のインポートジョブを提出するために再利用できます。このメカニズムにより、任意のラベルに対応するデータが最大で一度だけインポートされることを保証し、「最大一回（At-Most-Once）」のセマンティクスを実現します。

StarRocksでは、すべてのインポート方法がアトミック性を保証しており、同じインポートジョブ内のすべての有効なデータは、すべてが有効になるか、すべてが無効になるかのどちらかであり、一部のデータのみがインポートされるという状況は発生しません。ここでの有効なデータには、型変換エラーなどのデータ品質の問題によりフィルタリングされたデータは含まれません。

StarRocksは、インポートジョブを提出するための2種類のアクセスプロトコル、MySQLプロトコルとHTTPプロトコルを提供しています。異なるインポート方法がサポートするアクセスプロトコルは異なりますので、詳細は「[インポート方法](../loading/Loading_intro.md#インポート方法)」の章を参照してください。

> **注意**
>
> インポート操作には対象テーブルのINSERT権限が必要です。お使いのユーザーアカウントにINSERT権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)を参照してユーザーに権限を付与してください。

## サポートされるデータ型

StarRocksはすべてのデータ型のインポートをサポートしています。一部のデータ型のインポートにはいくつかの制限がある場合がありますので、詳細は[データ型](../sql-reference/sql-statements/data-types/BIGINT.md)を参照してください。

## インポートモード

StarRocksは、同期インポートと非同期インポートの2つのインポートモードをサポートしています。

> **説明**
>
> 外部プログラムがStarRocksのインポートに接続する場合は、どのインポートモードを使用するかを先に判断し、その後で接続ロジックを決定する必要があります。

### 同期インポート

同期インポートとは、インポートジョブを作成した後、StarRocksがジョブを同期的に実行し、ジョブの実行完了後にインポート結果を返すことを指します。返されたインポート結果を確認して、インポートジョブが成功したかどうかを判断できます。

同期モードをサポートするインポート方法には、Stream LoadとINSERTがあります。

ユーザーの操作プロセスは以下の通りです：

1. インポートジョブを作成します。

2. StarRocksが返すインポート結果を確認します。

3. インポート結果を判断します。インポート結果が失敗した場合は、インポートジョブを再試行できます。

### 非同期インポート

非同期インポートとは、インポートジョブを作成した後、StarRocksが直接ジョブの作成結果を返すことを指します。

- インポートジョブの作成に成功した場合、StarRocksはインポートジョブを非同期的に実行します。しかし、ジョブの作成が成功したとしても、データのインポートがすでに成功したわけではありません。インポートジョブの状態を確認するためにステートメントやコマンドを使用し、インポートジョブの状態に基づいてデータのインポートが成功したかどうかを判断する必要があります。
- インポートジョブの作成に失敗した場合は、失敗情報に基づいて再試行が必要かどうかを判断できます。

非同期モードをサポートするインポート方法には、Broker Load、Routine Load、およびSpark Loadがあります。

ユーザーの操作プロセスは以下の通りです：

1. インポートジョブを作成します。

2. StarRocksが返すジョブの作成結果に基づいて、ジョブが正常に作成されたかどうかを判断します。

   - ジョブの作成に成功した場合は、ステップ3に進みます。
   - ジョブの作成に失敗した場合は、ステップ1に戻ってインポートジョブを再試行できます。

3. インポートジョブの状態を定期的に確認し、状態が**FINISHED**または**CANCELLED**になるまで続けます。

Broker LoadとSpark Loadのインポートジョブの実行プロセスは主に5つの段階に分かれており、以下の図に示されています。

![Broker Load と Spark Load のフローチャート](../assets/4.1-1.png)

各段階の説明は以下の通りです：

1. **PENDING**

   この段階は、インポートジョブを提出した後、FEによる実行スケジューリングを待っている状態を指します。

2. **ETL**

   この段階では、データの事前処理が行われ、クリーニング、パーティショニング、ソート、集約などが含まれます。

   > **説明**
   >
   > Broker Loadのジョブの場合、この段階は直接完了します。

3. **LOADING**

   この段階では、まずデータをクリーニングして変換し、その後データをBEに送信して処理します。すべてのデータがインポートされた後、有効化を待つプロセスに入りますが、この時点でのインポートジョブの状態は依然として**LOADING**です。

4. **FINISHED**

   インポートジョブに関連するすべてのデータが有効になった後、ジョブの状態は**FINISHED**に変わります。この時点で、インポートされたデータはすべて照会可能です。**FINISHED**はインポートジョブの最終状態です。

5. **CANCELLED**

   インポートジョブの状態が**FINISHED**になる前に、いつでもジョブをキャンセルすることができます。また、インポートにエラーが発生した場合、StarRocksシステムは自動的にインポートジョブをキャンセルします。ジョブがキャンセルされると、**CANCELLED**状態になります。**CANCELLED**もインポートジョブの最終状態の一つです。

Routine Loadのインポートジョブの実行プロセスは以下の通りです：

1. ユーザーがMySQLプロトコルをサポートするクライアントを使用してFEにインポートジョブを提出します。

2. FEはそのインポートジョブを複数のタスクに分割し、各タスクが複数のパーティションのデータをインポートする責任を持ちます。

3. FEは各タスクを指定されたBEに割り当てて実行します。

4. BEが割り当てられたタスクを完了した後、FEに報告します。

5. FEは報告結果に基づいて、新しいタスクを生成し続けるか、失敗したタスクを再試行するか、タスクのスケジューリングを一時停止します。

## インポート方法

StarRocksは、[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)、および[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)の複数のインポート方法を提供しており、異なるビジネスシナリオでのデータインポートニーズに応えます。

| インポート方法            | データソース                                                                                          | ビジネスシナリオ                                                                                                     | データ量（単一ジョブ）      | データフォーマット                                            | 同期モード    | プロトコル   |
| ------------------ | ---------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- | ------------------ | ------------------------------------------------- | ---------- | ------ |
| Stream Load        |<ul><li>ローカルファイル</li><li>ストリームデータ</li></ul>                                                        | HTTPプロトコルを介してローカルファイルをインポートするか、プログラムを介してデータストリームをインポートします。                                                                 | 10 GB以内          |<ul><li>CSV</li><li>JSON</li></ul>                 | 同期       | HTTP  |
| Broker Load        |<ul><li>HDFS</li><li>Amazon S3</li><li>Google GCS</li><li>Microsoft Azure Storage</li><li>アリババクラウドOSS</li><li>テンセントクラウドCOS</li><li>ファーウェイクラウドOBS</li><li>その他のS3プロトコル互換オブジェクトストレージ（例：MinIO）</li></ul> | HDFSまたは外部クラウドストレージシステムからデータをインポートします。                                                                               | 数十GBから数百GB        |<ul><li>CSV</li><li>Parquet</li><li>ORC</li></ul> | 非同期        | MySQL |
| Routine Load       | Apache Kafka®                                                                                 | Kafkaからリアルタイムでデータストリームをインポートします。                                                                             | ミニバッチでMBからGBレベル |<ul><li>CSV</li><li>JSON</li><li>Avro（バージョン3.0.1以降でサポート）</li></ul>          | 非同期     | MySQL |
| Spark Load         | <ul><li>HDFS</li><li>Hive</li></ul>                                                            | <ul><li>Apache Spark™クラスターを介して初めてHDFSまたはHiveから大量のデータを移行インポートする。</li><li>グローバルデータディクショナリを使用して正確な重複排除を行う必要がある。</li></ul> | 数十GBからTBレベル    |<ul><li>CSV</li><li>ORC（バージョン2.0以降でサポート）</li><li>Parquet（バージョン2.0以降でサポート）</li></ul>       | 非同期     | MySQL |
| INSERT INTO SELECT |<ul><li>StarRocksテーブル</li><li>外部テーブル</li><li>AWS S3</li><li>HDFS</li></ul>**注意**<br />AWS S3またはHDFSからデータをインポートする場合、ParquetとORCフォーマットのデータのみをサポートします。                                                    |<ul><li>外部テーブルからのインポート。</li><li>StarRocksデータテーブル間のデータインポート。</li></ul>                                              | メモリに関連           | StarRocksテーブル                                     | 同期        | MySQL |
| INSERT INTO VALUES |<ul><li>プログラム</li><li>ETLツール</li></ul>                                                          |<ul><li>単一または少量のデータをバッチで挿入。</li><li>JDBCなどのインターフェースを介してインポートします。</li></ul>                                             | 単純なテスト用           | SQL                                              | 同期        | MySQL |

ビジネスシナリオ、データ量、データソース、データフォーマット、インポート頻度などに基づいて、適切なインポート方法を選択できます。また、インポート方法を選択する際には、以下の点に注意してください：

- Kafkaからデータをインポートする場合は、[Routine Load](../loading/RoutineLoad.md)を使用することをお勧めします。ただし、インポートプロセス中に複雑なマルチテーブルの結合やETLの事前処理が必要な場合は、Apache Flink®を使用してKafkaからデータを読み取り、データを処理した後、StarRocksが提供する標準プラグイン[flink-connector-starrocks](../loading/Flink-connector-starrocks.md)を介して処理済みのデータをStarRocksにインポートすることをお勧めします。

- Hive、Iceberg、Hudi、Delta Lake からデータをインポートする場合は、[Hive catalog](../data_source/catalog/hive_catalog.md)、[Iceberg catalog](../data_source/catalog/iceberg_catalog.md)、[Hudi Catalog](../data_source/catalog/hudi_catalog.md)、[Delta Lake Catalog](../data_source/catalog/deltalake_catalog.md) を作成し、[INSERT](../loading/InsertInto.md) を使用してインポートすることをお勧めします。

- 別の StarRocks クラスターや Elasticsearch からデータをインポートする場合は、[StarRocks 外部テーブル](../data_source/External_table.md#starrocks-外部表)または [Elasticsearch 外部テーブル](../data_source/External_table.md#deprecated-elasticsearch-外部表) を作成し、[INSERT](../loading/InsertInto.md) を使用してインポートすることをお勧めします。または、[DataX](../loading/DataX-starrocks-writer.md) を使用してインポートすることもできます。

  > **注意**
  >
  > StarRocks 外部テーブルはデータの書き込みのみをサポートし、データの読み取りはサポートしていません。

- MySQL からデータをインポートする場合は、[MySQL 外部テーブル](../data_source/External_table.md#deprecated-mysql-外部表) を作成し、[INSERT](../loading/InsertInto.md) を使用してインポートすることをお勧めします。または、[DataX](../loading/DataX-starrocks-writer.md) を使用してインポートすることもできます。リアルタイムデータをインポートする場合は、[MySQL からのリアルタイム同期](../loading/Flink_cdc_load.md) を参照してインポートすることをお勧めします。

- Oracle、PostgreSQL、SQL Server などのデータソースからデータをインポートする場合は、[JDBC 外部テーブル](../data_source/External_table.md#更多数据库jdbc的外部表) を作成し、[INSERT](../loading/InsertInto.md) を使用してインポートすることをお勧めします。または、[DataX](../loading/DataX-starrocks-writer.md) を使用してインポートすることもできます。

以下の図は、さまざまなデータソースのシナリオでどのインポート方法を選択するべきかを詳細に示しています。

![データソースとインポート方法の関係図](../assets/4.1-3.png)

## メモリ制限

インポートジョブのメモリ使用量を制限するパラメータを設定することで、特にインポートの並行性が高い場合にインポートジョブが多くのメモリを消費するのを防ぐことができます。同時に、メモリ使用量の上限を小さくしすぎないように注意する必要があります。メモリ使用量の上限が小さすぎると、インポート中にメモリのデータが頻繁にディスクにフラッシュされ、インポートの効率に影響を与える可能性があります。具体的なビジネスシナリオの要件に基づいて、メモリ使用量の上限を合理的に設定することをお勧めします。

異なるインポート方法でのメモリ制限の方法は若干異なります。詳細は [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) および [INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md) を参照してください。注意すべき点は、インポートジョブは通常複数の BE で実行されるため、これらのメモリパラメータは単一の BE 上でのインポートジョブのメモリ使用量を制限するものであり、クラスタ全体のメモリ使用量の合計ではありません。

また、単一の BE 上で実行されるすべてのインポートジョブの総メモリ使用量の上限を制限するパラメータを設定することもできます。詳細は「[システム設定](../loading/Loading_intro.md#システム設定)」セクションを参照してください。

## 使用説明

### インポート時の自動割り当て

データをインポートする際に、データファイルの特定のフィールドをインポートしないように指定することができます。この場合：

- StarRocks テーブルを作成する際に `DEFAULT` キーワードを使用して目的の列にデフォルト値を指定している場合、StarRocks はインポート時にその列に `DEFAULT` で指定されたデフォルト値を自動的に埋めます。

  [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、および [INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md) の4つのインポート方法は現在 `DEFAULT current_timestamp`、`DEFAULT <デフォルト値>`、および `DEFAULT (<式>)` をサポートしています。[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) のインポート方法は現在 `DEFAULT current_timestamp` と `DEFAULT <デフォルト値>` のみをサポートしており、`DEFAULT (<式>)` はサポートしていません。

  > **説明**
  >
  > 現在 `DEFAULT (<式>)` は `uuid()` および `uuid_numeric()` 関数のみをサポートしています。

- StarRocks テーブルを作成する際に目的の列にデフォルト値を指定していない場合、StarRocks はインポート時にその列に自動的に `NULL` 値を埋めます。

  > **説明**
  >
  > 列を作成する際に `NOT NULL` と定義している場合、インポートはエラーとなり、ジョブは失敗します。

  [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、および [Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) では、インポートする列のパラメータを指定する際に関数を使用してその列に埋める値を指定することもできます。

`NOT NULL` と `DEFAULT` の使用方法については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) を参照してください。

### データインポートのセキュリティレベル設定

StarRocks クラスタに複数のデータレプリカがある場合、ビジネス要件に応じてテーブルの異なるデータインポートセキュリティレベルを設定できます。つまり、いくつかのデータレプリカが成功した後に StarRocks がインポート成功を返すかを設定できます。[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) 時に属性（PROPERTIES）`write_quorum` を追加してデータインポートのセキュリティレベルを指定することができます。または、[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) ステートメントを使用して既存のテーブルにこの属性を追加することができます。この属性はバージョン 2.5 からサポートされています。

## システム設定

このセクションでは、すべてのインポート方法に共通するパラメータ設定について説明します。

### FE 設定

各 FE の設定ファイル **fe.conf** を変更して以下のパラメータを設定できます：

- `max_load_timeout_second` と `min_load_timeout_second`

  インポートのタイムアウト時間の最大値と最小値を秒単位で設定します。デフォルトの最大タイムアウト時間は3日、最小タイムアウト時間は1秒です。カスタムのインポートタイムアウト時間はこの最大値と最小値の範囲を超えることはできません。このパラメータ設定はすべてのインポートモードのジョブに適用されます。

- `desired_max_waiting_jobs`

  待機キューに収容できるインポートジョブの最大数を設定します。デフォルト値は1024です（2.4以前のバージョンではデフォルト値は100、2.5以降のバージョンではデフォルト値が1024に変更されました）。FE に **PENDING** 状態のインポートジョブが最大数に達した場合、FE は新しいインポートリクエストを拒否します。このパラメータ設定は非同期実行のインポートにのみ適用されます。

- `max_running_txn_num_per_db`

  StarRocks クラスタの各データベースで実行中のインポートトランザクションの最大数（1つのインポートジョブには複数のトランザクションが含まれる場合があります）を設定します。デフォルト値は1000です（3.1バージョン以降、デフォルト値は100から1000に変更されました）。データベースで実行中のインポートトランザクションが最大数に達した場合、後続のインポートジョブは実行されません。同期インポートジョブの場合はジョブが拒否され、非同期インポートジョブの場合はキューで待機します。

  > **説明**
  >
  > すべてのインポートモードのジョブが含まれ、統一的にカウントされます。

- `label_keep_max_second`

  完了して **FINISHED** または **CANCELLED** 状態のインポートジョブレコードが StarRocks システムに保持される期間を設定します。デフォルト値は3日です。このパラメータ設定はすべてのインポートモードのジョブに適用されます。

### BE 設定

各 BE の設定ファイル **be.conf** を変更して以下のパラメータを設定できます：

- `write_buffer_size`

  BE 上のメモリブロックのサイズのしきい値を設定します。デフォルトのしきい値は100MBです。インポートされたデータはBE上で最初にメモリブロックに書き込まれ、メモリブロックのサイズがこのしきい値に達した後にディスクに書き戻されます。しきい値が小さすぎると、BE上に多くの小さなファイルが存在し、クエリのパフォーマンスに影響を与える可能性があります。この場合はしきい値を適切に増やしてファイル数を減らすことができます。しきい値が大きすぎると、RPC（Remote Procedure Call）のタイムアウトを引き起こす可能性があります。この場合はこのパラメータの値を適切に調整することができます。

- `streaming_load_rpc_max_alive_time_sec`

  Writer プロセスの待機タイムアウト時間を指定します。デフォルトは600秒です。インポートプロセス中、StarRocks は各 Tablet に対してデータの受信と書き込みを行う Writer プロセスを開始します。パラメータで指定された時間内に Writer プロセスがデータを受信しない場合、StarRocks システムは自動的にこの Writer プロセスを破棄します。システムの処理速度が遅い場合、Writer プロセスは次のデータバッチを長時間受信できず、"TabletWriter add batch with unknown id" エラーを報告することがあります。この場合はこのパラメータの値を適切に増やすことができます。

- `load_process_max_memory_limit_bytes` と `load_process_max_memory_limit_percent`

  インポートに使用される最大メモリ量と最大メモリ使用率を設定し、単一の BE 上で実行されるすべてのインポートジョブのメモリ使用量の合計の上限を制限します。StarRocks システムは、これら2つのパラメータのうち小さい方を最終的な使用上限として採用します。

  - `load_process_max_memory_limit_bytes`：BE 上の最大メモリ使用量を指定します。デフォルトは100GBです。
  - `load_process_max_memory_limit_percent`：BE 上の最大メモリ使用率を指定します。デフォルトは30%です。このパラメータは `mem_limit` パラメータとは異なります。`mem_limit` パラメータはBEプロセスのメモリ上限を指定し、デフォルトの硬い上限はBEが存在するマシンのメモリの90%、ソフト上限はBEが存在するマシンのメモリの90% x 90%です。

    BEサーバーの物理メモリサイズがMである場合、インポートに使用されるメモリの上限は：`M x 90% x 90% x 30%`です。

### セッション変数

以下の[セッション変数](../reference/System_variable.md)を設定できます:

- `query_timeout`

  クエリのタイムアウト時間を設定するために使用します。単位は秒です。範囲：`1` ~ `259200`。デフォルト値：`300`、これは5分に相当します。この変数は、現在の接続におけるすべてのクエリ文とINSERT文に適用されます。

## よくある質問

[インポートに関するよくある質問](../faq/loading/Loading_faq.md)をご覧ください。
