---
displayed_sidebar: "Japanese"
---

# データロードの概要

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

データロードとは、さまざまなデータソースからの生データをビジネス要件に基づいてクレンジングおよび変換し、その結果をStarRocksに読み込んで、高速なデータ分析を実現するプロセスです。

データはロードジョブを実行することでStarRocksに読み込むことができます。各ロードジョブには、ユーザーによって指定されるか、StarRocksによって自動的に生成されるユニークなラベルがあります。各ラベルは1つのロードジョブにのみ使用できます。ロードジョブが完了すると、そのラベルは他のロードジョブで再利用できません。失敗したロードジョブのラベルのみが再利用できます。この仕組みにより、特定のラベルに関連付けられたデータは1度だけ読み込めるため、At-Most-Onceセマンティクスが実装されます。

StarRocksが提供するすべてのデータのロード方式は、アトミシティを保証できます。アトミシティとは、ロードジョブ内の適格なデータがすべて正常にロードされるか、適格なデータの一部もロードされないことを意味します。つまり、適格なデータの一部がロードされ、他のデータがロードされないことはありません。なお、適格なデータにはデータ型変換エラーなどの品質の問題で除外されたデータは含まれません。

StarRocksは、ロードジョブを提出するために使用できる2つの通信プロトコル、MySQLおよびHTTPをサポートしています。各ロード方法がサポートするプロトコルについての詳細については、[ローディング方法](../loading/Loading_intro.md#loading-methods)のセクションを参照してください。

<InsertPrivNote />

## サポートされるデータ型

StarRocksはすべてのデータ型のデータを読み込むことができます。読み込み可能な特定のデータ型に関する制限については、[データ型](../sql-reference/sql-statements/data-types/BIGINT.md)を参照してください。

## ロードモード

StarRocksは、同期ロードモードと非同期ロードモードの2つのロードモードをサポートしています。

> **注意**
>
> 外部プログラムを使用してデータをロードする場合は、ローディング方法を選択する前に、ビジネス要件に最適なローディングモードを選択する必要があります。

### 同期ロード

同期ロードモードでは、ロードジョブを提出すると、StarRocksがジョブを同期的に実行してデータを読み込み、ジョブが完了した後に結果を返します。ジョブの結果に基づいてジョブが成功したかどうかを確認することができます。

StarRocksは、同期ロードをサポートする2つのローディング方法を提供しています: [Stream Load](../loading/StreamLoad.md) および [INSERT](../loading/InsertInto.md)。

同期ロードのプロセスは次のようになります:

1. ロードジョブを作成します。

2. StarRocksによって返されたジョブ結果を表示します。

3. ジョブの結果に基づいてジョブが成功したかどうかを確認します。ジョブの結果がロードの失敗を示す場合は、ジョブを再試行することができます。

### 非同期ロード

非同期ロードモードでは、ロードジョブを提出すると、StarRocksは即座にジョブの作成結果を返します。

- 結果がジョブの作成成功を示す場合、StarRocksはジョブを非同期で実行します。ただし、これはデータが正常に読み込まれたことを意味するわけではありません。ステートメントまたはコマンドを使用してジョブの状態を確認する必要があります。その後、ジョブのステータスに基づいてデータが正常に読み込まれたかどうかを判断できます。

- 結果がジョブの作成失敗を示す場合、失敗情報に基づいてジョブを再試行する必要があるかどうかを判断できます。

StarRocksは、非同期ロードをサポートする3つのローディング方法を提供しています: [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md), [Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md), および [Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)。

非同期ロードのプロセスは次のようになります:

1. ロードジョブを作成します。

2. StarRocksによって返されたジョブ作成結果を表示し、ジョブが正常に作成されたかどうかを判断します。
   a. ジョブの作成が成功した場合、ステップ3に進みます。
   b. ジョブの作成が失敗した場合、ステップ1に戻ります。

3. ステートメントまたはコマンドを使用してジョブの状態を確認し、ジョブのステータスが **FINISHED** または **CANCELLED** を示すまで状態を確認します。

ブローカーロードまたはスパークロードジョブのワークフローは、次の図に示される5つのステージで構成されています。

![ブローカーロードまたはスパークロードオーバーフロー](../assets/4.1-1.png)

ワークフローは次のように説明されます:

1. **PENDING**

   ジョブはFEによってスケジュールされるのを待っているキューにあります。

2. **ETL**

   FEは、クレンジング、パーティショニング、ソート、集計など、データの前処理を行います。
   > **注意**
   >
   > ジョブがブローカーロードの場合、このステージは直接完了します。

3. **LOADING**

   FEはデータをクレンジングおよび変換し、その後データをBEに送信します。すべてのデータが読み込まれた後、データは適用を待つキューにあります。この時点で、ジョブのステータスは **LOADING** のままです。

4. **FINISHED**

   すべてのデータが適用されると、ジョブのステータスは **FINISHED** になります。この時点で、データをクエリできます。 **FINISHED** は最終的なジョブ状態です。

5. **CANCELLED**

   ジョブのステータスが **FINISHED** になる前に、いつでもジョブをキャンセルすることができます。さらに、ロードエラーが発生した場合、StarRocksは自動的にジョブをキャンセルすることがあります。ジョブがキャンセルされると、ジョブのステータスは **CANCELLED** になります。 **CANCELLED** も最終的なジョブの状態です。

ルーチンジョブのワークフローは次のように説明されています:

1. ジョブはMySQLクライアントからFEに送信されます。

2. FEはジョブを複数のタスクに分割します。各タスクは複数のパーティションからデータを読み込むように設計されています。

3. FEは指定されたBEにタスクを配布します。

4. BEはタスクを実行し、タスクの実行が完了した後にFEに報告します。

5. FEはBEからの報告に基づいて、後続のタスクを生成し、失敗したタスクがあれば再試行したり、タスクのスケジューリングを中断したりします。

## ローディング方法

StarRocksは、様々なビジネスシナリオでデータを読み込むための5つのローディング方法を提供しています: [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md), [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md), [Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md), [Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md), および [INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

| ロード方法 | データソース                                        | ビジネスシナリオ                                            | 1つのロードジョブあたりのデータ容量                                     | データファイル形式                                | ローディングモード | プロトコル |
| ------------------ | -------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ----------------------------------------------- | ------------ | -------- |
| Stream Load        |  <ul><li>ローカルファイル</li><li>データストリーム</li></ul>| ローカルファイルシステムからデータファイルを読み込むか、プログラムを使用してデータストリームを読み込む。 | 10 GB 以下                             |<ul><li>CSV</li><li>JSON</li></ul>               | 同期  | HTTP     |
| Broker Load        | <ul><li>HDFS</li><li>Amazon S3</li><li>Google GCS</li><li>Microsoft Azure Storage</li><li>Alibaba Cloud OSS</li><li>Tencent Cloud COS</li><li>Huawei Cloud OBS</li><li>その他のS3互換ストレージシステム（MinIOなど）</li></ul>| HDFSやクラウドストレージからデータを読み込む。                        | 数十GBから数百GB                                | <ul><li>CSV</li><li>Parquet</li><li>ORC</li></ul>| 非同期 | MySQL    |
| Routine Load       | Apache Kafka®                                       | Kafkaからリアルタイムにデータを読み込む。                   | MBからGB単位のデータ（ミニバッチ）                           |<ul><li>CSV</li><li>JSON</li><li>Avro (v3.0.1以降でサポート)</li></ul>          | 非同期 | MySQL    |
| Spark Load         | <ul><li>HDFS</li><li>Hive</li></ul>     |<ul><li>HDFSやHiveから大量のデータをApache Spark™クラスタを使用して移行する。</li><li>デデュプリケーションのためのグローバルデータ辞書を使用しデータを読み込む。</li></ul>| 数十GBから数TB                                     |<ul><li>CSV</li><li>ORC (v2.0以降でサポート)</li><li>Parquet (v2.0以降でサポート)</li></ul>       | 非同期 | MySQL    |
| INSERT INTO SELECT | <ul><li>StarRocksテーブル</li><li>外部テーブル</li><li>AWS S3</li></ul>**注意**<br />AWS S3からデータを読み込む場合は、Parquet形式またはORC形式のファイルのみサポートされています。     |<ul><li>外部テーブルからデータを読み込む。</li><li>StarRocksテーブル間でデータを読み込む。</li></ul>| 固定されていません（データ容量はメモリサイズに応じて変化します。） | StarRocksテーブル      | 同期  | MySQL    |
| INSERT INTO VALUES | <ul><li>プログラム</li><li>ETLツール</li></ul> |<ul><li>個々のレコードとして少量のデータを挿入します。</li><li>JDBCなどのAPIを使用してデータをロードします。</li></ul>| 少量のデータを挿入する                                       | SQL                   | 同期          | MySQL    |

ビジネスシナリオ、データボリューム、データソース、データファイル形式、およびロード頻度に基づいて、選択したロード方法を決定できます。また、次の点に注意してロード方法を選択する場合があります:

- Kafkaからデータをロードする場合、[定期ロード](../loading/RoutineLoad.md)の使用をお勧めします。ただし、データが複数のテーブルの結合および抽出、変換、ロード (ETL) 操作を必要とする場合は、Apache Flink®を使用してデータをKafkaから事前処理し、その後 [flink-connector-starrocks](../loading/Flink-connector-starrocks.md)を使用してデータをStarRocksにロードできます。

- Hive、Iceberg、Hudi、またはDelta Lakeからデータをロードする場合は、[Hiveカタログ](../data_source/catalog/hive_catalog.md)、[Icebergカタログ](../data_source/catalog/iceberg_catalog.md)、[Hudiカタログ](../data_source/catalog/hudi_catalog.md)、または[Delta Lakeカタログ](../data_source/catalog/deltalake_catalog.md)を作成し、その後 [INSERT](../loading/InsertInto.md) を使用してデータをロードすることをお勧めします。

- 他のStarRocksクラスタやElasticsearchクラスタからデータをロードする場合、[StarRocks外部テーブル](../data_source/External_table.md#starrocks-external-table)または[Elasticsearch外部テーブル](../data_source/External_table.md#deprecated-elasticsearch-external-table)を作成し、その後 [INSERT](../loading/InsertInto.md) を使用してデータをロードすることをお勧めします。

  > **注意**
  >
  > StarRocks外部テーブルはデータの書き込みのみをサポートし、データの読み取りはサポートされていません。

- MySQLデータベースからデータをロードする場合、[MySQL外部テーブル](../data_source/External_table.md#deprecated-mysql-external-table)を作成し、その後 [INSERT](../loading/InsertInto.md) を使用してデータをロードすることをお勧めします。リアルタイムでデータをロードする場合、[MySQLからのリアルタイム同期](../loading/Flink_cdc_load.md)で提供される手順に従ってデータをロードすることをお勧めします。

- Oracle、PostgreSQL、およびSQL Serverなどの他のデータソースからデータをロードする場合、[JDBC外部テーブル](../data_source/External_table.md#external-table-for-a-jdbc-compatible-database)を作成し、その後 [INSERT](../loading/InsertInto.md) を使用してデータをロードすることをお勧めします。

以下の図は、StarRocksがサポートするさまざまなデータソースとそれらからデータをロードするために使用できるロード方法の概要を示しています。

![データロードソース](../assets/4.1-3.png)

## メモリ制限

StarRocksでは、各ロードジョブのメモリ使用量を制限するためのパラメータを提供しており、特に高い並行性のシナリオでメモリ消費を削減することができます。ただし、極端に低いメモリ使用制限を指定しないでください。メモリ使用制限が極端に低いと、ロードジョブのメモリ使用が指定された制限に達するたびにデータが頻繁にメモリからディスクにフラッシュされる可能性があります。ビジネスシナリオに基づいて適切なメモリ使用制限を指定することをお勧めします。

メモリ使用制限に使用されるパラメータは、各ロード方法ごとに異なります。詳細は、[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)、および [INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md) を参照してください。なお、ロードジョブは通常、複数のBEで実行されます。そのため、パラメータは、関与する各BEの各ロードジョブのメモリ使用を制限するものであり、すべての関与するBEのロードジョブの総メモリ使用を制限するものではありません。

StarRocksはまた、各個々のBEで実行されるすべてのロードジョブの総メモリ使用を制限するためのパラメータを提供しています。詳細は、本トピックの "[システム設定](../loading/Loading_intro.md#system-configurations)" セクションを参照してください。

## 使用上の注意

### データのロード中に宛先列を自動的に埋める

データをロードする際に、データファイルの特定のフィールドからデータをロードしないように選択できます。

- StarRocksのテーブルにおいて、ソースフィールドに対応する宛先StarRocksテーブル列に `DEFAULT` キーワードを指定した場合、StarRocksは指定されたデフォルト値を宛先列に自動的に埋めます。

   [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、および [INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md) は、`DEFAULT current_timestamp`、`DEFAULT <default_value>`、および `DEFAULT (<expression>)` をサポートしています。[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) は、`DEFAULT current_timestamp` と `DEFAULT <default_value>` のみをサポートしています。

  > **注**
  >
  > `DEFAULT (<expression>)` は、`uuid()` および`uuid_numeric()` の関数のみをサポートしています。

- StarRocksのテーブルにおいて、ソースフィールドに対応する宛先StarRocksテーブル列に `DEFAULT` キーワードを指定しなかった場合、StarRocksは宛先列に `NULL` を自動的に埋めます。

  > **注**
  >
  > 宛先列が `NOT NULL` として定義されている場合、ロードに失敗します。

  [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、および [Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) では、宛先列に埋める値を指定するためのパラメータを使用することもできます。

`NOT NULL` および `DEFAULT` の使用方法については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) を参照してください。

### データロードのための書き込みクォーラムを設定する

StarRocksクラスタに複数のデータレプリカがある場合、テーブルごとに異なる書き込みクォーラムを設定できます。つまり、StarRocksがロードが成功したと判断するために必要なレプリカの数を設定できます。このプロパティは、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) でプロパティ `write_quorum` を追加するか、既存のテーブルに [ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) を使用してこのプロパティを追加することができます。このプロパティは v2.5 からサポートされています。

## システム設定

このセクションでは、StarRocksが提供するすべてのロード方法に適用されるいくつかのパラメータ設定について説明します。

### FE設定

各FEの構成ファイル**fe.conf**で、次のパラメータを設定できます:

- `max_load_timeout_second` および `min_load_timeout_second`
  
  これらのパラメータは、各ロードジョブの最大タイムアウト期間と最小タイムアウト期間を指定します。タイムアウト期間は秒単位で測定されます。デフォルトの最大タイムアウト期間は 3 日間で、デフォルトの最小タイムアウト期間は 1 秒です。指定する最大タイムアウト期間と最小タイムアウト期間は、1秒から3日までの範囲内でなければなりません。これらのパラメータは同期ロードジョブと非同期ロードジョブの両方に適用されます。

- `desired_max_waiting_jobs`
  
  このパラメータは、キューで保持できる最大ロードジョブ数を指定します。デフォルト値は **1024** です (v2.4以前は100, v2.5以降は1024)。FE上の**PENDING**状態のロードジョブ数が指定した最大数に達すると、FEは新しいロードリクエストを拒否します。このパラメータは非同期ロードジョブにのみ適用されます。

- `max_running_txn_num_per_db`
  
  このパラメータは、StarRocksクラスタの各データベースで許可されている進行中のロードトランザクションの最大数を指定します。1つのロードジョブには1つ以上のトランザクションが含まれることができます。デフォルト値は **100** です。指定した最大数に達した場合、データベースで実行されているロードトランザクションの数が多い場合、サブセクエントなロードジョブはスケジュールされません。このような状況では、同期ロードジョブが提出された場合、ジョブは拒否されます。非同期ロードジョブが提出された場合、ジョブはキューに保持されます。

  > **注**
  >
  > StarRocksは、すべてのロードジョブを合計して数え、同期ロードジョブと非同期ロードジョブを区別しません。

- `label_keep_max_second`
  
  このパラメータは、完了したロードジョブの履歴レコードの保持期間を指定します。デフォルトの保持期間は 3 日間です。このパラメータは同期ロードジョブと非同期ロードジョブの両方に適用されます。

### BE設定

各BEの構成ファイル**be.conf**で、次のパラメータを設定できます:

- `write_buffer_size`
```markdown
  This parameter specifies the maximum memory block size. The default size is 100 MB. The loaded data is first written to a memory block on the BE. When the amount of data that is loaded reaches the maximum memory block size that you specify, the data is flushed to disk. You must specify a proper maximum memory block size based on your business scenario.

  - If the maximum memory block size is exceedingly small, a large number of small files may be generated on the BE. In this case, query performance degrades. You can increase the maximum memory block size to reduce the number of files generated.
  - If the maximum memory block size is exceedingly large, remote procedure calls (RPCs) may time out. In this case, you can adjust the value of this parameter based on your business needs.

- `streaming_load_rpc_max_alive_time_sec`
  
  The waiting timeout period for each Writer process. The default value is 600 seconds. During the data loading process, StarRocks starts a Writer process to receive data from and write data to each tablet. If a Writer process does not receive any data within the waiting timeout period that you specify, StarRocks stops the Writer process. When your StarRocks cluster processes data at low speeds, a Writer process may not receive the next batch of data within a long period of time and therefore reports a "TabletWriter add batch with unknown id" error. In this case, you can increase the value of this parameter.

- `load_process_max_memory_limit_bytes` and `load_process_max_memory_limit_percent`
  
  These parameters specify the maximum amount of memory that can be consumed for all load jobs on each individual BE. StarRocks identifies the smaller memory consumption among the values of the two parameters as the final memory consumption that is allowed.

  - `load_process_max_memory_limit_bytes`: specifies the maximum memory size. The default maximum memory size is 100 GB.
  - `load_process_max_memory_limit_percent`: specifies the maximum memory usage. The default value is 30%. This parameter differs from the `mem_limit` parameter. The `mem_limit` parameter specifies the total maximum memory usage of your StarRocks cluster, and the default value is 90% x 90%.

    If the memory capacity of the machine on which the BE resides is M, the maximum amount of memory that can be consumed for load jobs is calculated as follows: `M x 90% x 90% x 30%`.

### System variable configurations

You can configure the following [system variable](../reference/System_variable.md):

- `query_timeout`

  The query timeout duration. Unit: seconds. Value range: `1` to `259200`. Default value: `300`. This variable will act on all query statements in the current connection, as well as INSERT statements.

## Troubleshooting

For more information, see [FAQ about data loading](../faq/loading/Loading_faq.md).
```