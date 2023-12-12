---
displayed_sidebar: "Japanese"
---

# Apache Kafka® からデータを連続的に読込む

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

このトピックでは、Kafka のメッセージ（イベント）を StarRocks にストリームする Routine Load ジョブを作成し、Routine Load についての基本的な概念について紹介します。

ストリームのメッセージを StarRocks に連続的に読み込むには、メッセージストリームを Kafka トピックに保存し、Routine Load ジョブを作成してメッセージを消費します。Routine Load ジョブは StarRocks に永続化され、トピック内のすべてまたは一部のパーティションのメッセージを消費して StarRocks にロードします。

Routine Load ジョブは、データが StarRocks にロードされた際にデータの損失や重複を保証するためのエクザクトリーワンスデリバリーセマンティクスをサポートしています。

Routine Load は、データの変換をデータの読み込み時にサポートし、UPSERT および DELETE 操作によるデータの変更をサポートしています。詳細については、[データの変換](../loading/Etl_in_loading.md) および [読み込みを通じたデータの変更](../loading/Load_to_Primary_Key_tables.md) を参照してください。

<InsertPrivNote />

## サポートされるデータ形式

Routine Load は現在、Kafka クラスターから CSV、JSON、Avro (v3.0.1 以降のサポート) 形式のデータの読み込みをサポートしています。

> **注意**
>
> CSV データの場合は、以下の点に注意してください:
>
> - テキストの区切り文字として、コンマ (,)、タブ、またはパイプ (|) のいずれかの UTF-8 文字列を使用できます。ただし、その長さが 50 バイトを超えないようにしてください。
> - Null 値は `\N` を使用して示します。たとえば、データファイルには 3 つの列が含まれており、そのデータファイルからのレコードには最初と三番目の列にデータが含まれているが、２番目の列にはデータが含まれていないとします。この場合、２番目の列には `\N` を使用して Null 値を示す必要があります。つまり、そのレコードは `a,\N,b` のようにコンパイルする必要があります。`a,,b` では、レコードの２番目の列に空の文字列が含まれていると解釈されます。

## 基本的な概念

![routine load](../assets/4.5.2-1.png)

### 用語

- **ロードジョブ**

   ルーチンロードジョブは長時間実行されるジョブです。そのステータスが RUNNING である限り、ロードジョブはトピック内のメッセージを消費し、StarRocks にデータを連続的にロードするための 1 つまたは複数の同時ロードタスクを生成します。

- **ロードタスク**

   ロードジョブは特定のルールで複数のロードタスクに分割されます。ロードタスクはデータロードの基本単位です。ロードタスクは個々のイベントであり、[Stream Load](../loading/StreamLoad.md) に基づいたロードメカニズムを実装します。複数のロードタスクは異なるトピックパーティションから同時にメッセージを消費し、データを StarRocks にロードします。

### ワークフロー

1. **ルーチンロードジョブを作成します。**
   Kafka からデータをロードするには、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) ステートメントを実行して Routine Load ジョブを作成する必要があります。FE はステートメントを解析し、指定したプロパティに基づいてジョブを作成します。

2. **FE はジョブを複数のロードタスクに分割します。**

    FE は特定のルールに基づいてジョブを複数のロードタスクに分割します。各ロードタスクは個別のトランザクションです。
    分割のルールは次のとおりです:
    - FE は、希望する並行数 `desired_concurrent_number`、Kafka トピック内のパーティション数、および生存している BE ノードの数に基づいて実際のロードタスクの同時数を計算します。
    - FE は、計算された実際の並行数に基づいてジョブをロードタスクに分割し、タスクをタスクキューに配置します。

   各 Kafka トピックは複数のパーティションから構成されます。トピックパーティションとロードタスクの関係は次のとおりです:
    - パーティションはロードタスクに一意に割り当てられ、そのパーティションからのすべてのメッセージはロードタスクによって消費されます。
   - ロードタスクは、1 つまたは複数のパーティションからメッセージを消費できます。
   - すべてのパーティションはロードタスクに均等に分散されます。

3. **複数のロードタスクが同時に実行され、複数の Kafka トピックパーティションからメッセージを消費し、データを StarRocks にロードします**

   1. **FE はロードタスクをスケジュールして送信します**: FE はタイムリーにロードタスクをキューにスケジュールし、選択された Coordinator BE ノードに割り当てます。ロードタスク間のインターバルは構成項目 `max_batch_interval` によって定義されます。FE はロードタスクをすべての BE ノードに均等に分散します。`max_batch_interval` についての詳細は、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md#example) を参照してください。

   2. Coordinator BE はロードタスクを開始し、パーティション内のメッセージを消費し、データを解析してフィルタリングします。ロードタスクは、事前に定義されたメッセージの量が消費されるか、事前に定義された時間制限に達するまで続きます。メッセージのバッチサイズと時間制限は FE 構成の `max_routine_load_batch_size` および `routine_load_task_consume_second` で定義されています。詳細については、[Configuration](../administration/Configuration.md) を参照してください。その後、Coordinator BE はメッセージを Executor BE に配布します。Executor BE はメッセージをディスクに書き込みます。

         > **注意**
         >
         > StarRocks は、SASL_SSL、SASL、または SSL のセキュリティ認証メカニズムを介して Kafka にアクセスをサポートしています。このトピックでは、認証なしで Kafka に接続することを例としています。セキュリティ認証メカニズムを介して Kafka に接続する必要がある場合は、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を参照してください。

4. **FE は新しいロードタスクを生成してデータを連続的にロードします。**
   Executor BE がデータをディスクに書き込んだ後、Coordinator BE はロードタスクの結果を FE に報告します。FE はその結果に基づいて、データを連続的にロードするための新しいロードタスクを生成します。または、失敗したタスクを再試行して、StarRocks にロードされたデータが失われないようにします。

## Routine Load ジョブの作成

次の３つの例は、CSV 形式、JSON 形式、Avro 形式のデータを Kafka で消費し、Routine Load ジョブを作成して StarRocks にデータをロードする方法について説明しています。詳細な構文とパラメータの説明については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を参照してください。

### CSV 形式のデータの読み込み

このセクションでは、Kafka クラスター内の `ordertest1` トピックで CSV 形式のデータを消費し、StarRocks にデータをロードするための Routine Load ジョブを作成する方法について説明します。

#### データセットの準備

Kafka クラスターの `ordertest1` トピックには、CSV 形式のデータセットがあると仮定します。データセット内の各メッセージには、オーダーID、支払日、顧客名、国籍、性別、価格の 6 つのフィールドが含まれています。

```Plain
2020050802,2020-05-08,Johann Georg Faust,Deutschland,male,895
2020050802,2020-05-08,Julien Sorel,France,male,893
2020050803,2020-05-08,Dorian Grey,UK,male,1262
2020050901,2020-05-09,Anna Karenina",Russia,female,175
2020051001,2020-05-10,Tess Durbeyfield,US,female,986
2020051101,2020-05-11,Edogawa Conan,japan,male,8924
```

#### テーブルの作成

CSV 形式のデータのフィールドに従って、データベース `example_db` 内の `example_tbl1` テーブルを作成します。以下の例は、CSV 形式のデータにおける顧客の性別フィールドを除く 5 つのフィールドを持つテーブルを作成します。

```SQL
CREATE TABLE example_db.example_tbl1 ( 
    `order_id` bigint NOT NULL COMMENT "Order ID",
    `pay_dt` date NOT NULL COMMENT "Payment date", 
    `customer_name` varchar(26) NULL COMMENT "Customer name", 
    `nationality` varchar(26) NULL COMMENT "Nationality", 
    `price`double NULL COMMENT "Price"
) 
ENGINE=OLAP 
DUPLICATE KEY (order_id,pay_dt) 
DISTRIBUTED BY HASH(`order_id`); 
```

> **注意**
>
> v2.5.7 以降、StarRocks はテーブルを作成する際やパーティションを追加する際に、バケット数（BUCKETS）を自動的に設定できるようになりました。バケット数を手動で設定する必要はもはやありません。詳細については、[バケット数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets) を参照してください。

#### Routine Load ジョブの送信

以下のステートメントを実行して、`ordertest1` トピック内のメッセージを消費し、データを `example_tbl1` テーブルにロードする `example_tbl1_ordertest1` という名前の Routine Load ジョブを送信します。ロードタスクはトピック内の指定されたパーティションから初期オフセットのメッセージを消費します。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, temp_gender, price)
PROPERTIES
(
    "desired_concurrent_number" = "5"
)
FROM KAFKA
(
    "kafka_broker_list" = "<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1",
```
```sql
      + {T}
      + {T}
    + {T}
  + {T}
```
  **データマッピング:**

  - StarRocksはJSON形式のデータの`name`と`code`のキーを抽出し、`jsonpaths`プロパティで宣言されたキーにマッピングします。

  - StarRocksは`jsonpaths`プロパティで宣言されたキーを抽出し、それらを`COLUMNS`パラメータで宣言されたフィールドに**シーケンスに**マッピングします。

  - StarRocksは`COLUMNS`パラメータで宣言されたフィールドを抽出し、それらをStarRocksテーブルの列に名前によってマッピングします。

  **データ変換:**

  - 例えば`pay_time`のキーをDATEデータ型に変換し、StarRocksテーブルの`pay_dt`列にデータを読み込む必要があるため、`COLUMNS`パラメータでfrom_unixtime関数を使用する必要があります。他のフィールドは`example_tbl2`テーブルのフィールドに直接マッピングされます。

  - また、例えばJSON形式のデータから顧客の性別の列が除外されている場合、`COLUMNS`パラメータの`temp_gender`をこのフィールドのプレースホルダとして使用します。他のフィールドはStarRocksテーブル`example_tbl1`の列に直接マッピングされます。

    データ変換の詳細については、[データのロード時に変換](./Etl_in_loading.md)を参照してください。

    > **注意**
    >
    > JSONオブジェクト内のキーの名前と数が完全にStarRocksテーブルのフィールドと一致する場合は、`COLUMNS`パラメータを指定する必要はありません。

### Avro形式のデータをロード

v3.0.1以降、StarRocksはルーチンロードを使用してAvroデータのロードをサポートしています。

#### データセットの準備

##### Avroスキーマ

1. 以下のAvroスキーマファイル`avro_schema.avsc`を作成します。

      ```JSON
      {
          "type": "record",
          "name": "sensor_log",
          "fields" : [
              {"name": "id", "type": "long"},
              {"name": "name", "type": "string"},
              {"name": "checked", "type" : "boolean"},
              {"name": "data", "type": "double"},
              {"name": "sensor_type", "type": {"type": "enum", "name": "sensor_type_enum", "symbols" : ["TEMPERATURE", "HUMIDITY", "AIR-PRESSURE"]}}  
          ]
      }
      ```

2. Avroスキーマを[Schema Registry](https://docs.confluent.io/cloud/current/get-started/schema-registry.html#create-a-schema)に登録します。

##### Avroデータ

Avroデータを準備し、それをKafkaのトピック`topic_0`に送信します。

#### テーブルの作成

Avroデータのフィールドに基づいて、StarRocksクラスターの対象データベース`example_db`内に`sensor_log`テーブルを作成します。テーブルの列名は、Avroデータのフィールド名に一致する必要があります。テーブルの列とAvroデータのフィールドのデータ型マッピングについては、[データ型のマッピング](#データ型のマッピング)を参照してください。

```SQL
CREATE TABLE example_db.sensor_log ( 
    `id` bigint NOT NULL COMMENT "sensor id",
    `name` varchar(26) NOT NULL COMMENT "sensor name", 
    `checked` boolean NOT NULL COMMENT "checked", 
    `data` double NULL COMMENT "sensor data", 
    `sensor_type` varchar(26) NOT NULL COMMENT "sensor type"
) 
ENGINE=OLAP 
DUPLICATE KEY (id) 
DISTRIBUTED BY HASH(`id`);
```

> **注意**
>
> v2.5.7以降、StarRocksはテーブルを作成したりパーティションを追加する際にバケット数（BUCKETS）を自動的に設定できるようになりました。バケット数を手動で設定する必要はもはやありません。詳細については、[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

#### ルーチンロードジョブの送信

以下のステートメントを実行して、`sensor_log`テーブルにAvroメッセージを消費し、データをStarRocksテーブル`sensor_log`にロードする、`sensor_log_load_job`という名前のルーチンロードジョブを送信します。このロードジョブは、トピックの指定されたパーティションで初期オフセットからメッセージを消費します。

```SQL
CREATE ROUTINE LOAD example_db.sensor_log_load_job ON sensor_log  
PROPERTIES  
(  
    "format" = "avro"  
)  
FROM KAFKA  
(  
    "kafka_broker_list" = "<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>,...",
    "confluent.schema.registry.url" = "http://172.xx.xxx.xxx:8081",  
    "kafka_topic" = "topic_0",  
    "kafka_partitions" = "0,1,2,3,4,5",  
    "property.kafka_default_offsets" = "OFFSET_BEGINNING"  
);
```

- データ形式

  データ形式がAvroであることを定義するために、句`PROPERTIES`内で`"format = "avro"`を指定する必要があります。

- スキーマレジストリ

  Avroスキーマが登録されているSchema RegistryのURLを指定するには、`confluent.schema.registry.url`を構成する必要があります。StarRocksはこのURLを使用してAvroスキーマを取得します。形式は以下のとおりです：

  ```Plaintext
  confluent.schema.registry.url = http[s]://[<schema-registry-api-key>:<schema-registry-api-secret>@]<hostname|ip address>[:<port>]
  ```

- データマッピングと変換

  Avro形式のデータとStarRocksテーブルの間のマッピングや変換関係を指定するために、`COLUMNS`パラメータと`jsonpaths`プロパティを指定する必要があります。`COLUMNS`パラメータで指定されるフィールドの順序は、`jsonpaths`プロパティで指定されたフィールドの順序に一致している必要があり、フィールドの名前はStarRocksテーブルのものと一致している必要があります。`jsonpaths`プロパティはAvroデータから必要なフィールドを抽出するために使用され、これらのフィールドは`COLUMNS`プロパティで名前が付けられます。

  データ変換の詳細については、[データのロード時に変換](https://docs.starrocks.io/en-us/latest/loading/Etl_in_loading)を参照してください。

  > 注意
  >
  > Avroレコードのフィールドの名前と数が完全にStarRocksテーブルの列と一致する場合は、`COLUMNS`パラメータを指定する必要はありません。

ロードジョブを送信した後、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)ステートメントを実行してロードジョブの状態を確認できます。

#### データ型のマッピング

ロードするAvroデータのフィールドとStarRocksテーブルの列との間のデータ型のマッピングは以下の通りです：

##### 基本型

| Avro    | StarRocks |
| ------- | --------- |
| nul     | NULL      |
| boolean | BOOLEAN   |
| int     | INT       |
| long    | BIGINT    |
| float   | FLOAT     |
| double  | DOUBLE    |
| bytes   | STRING    |
| string  | STRING    |

##### 複合型

| Avro           | StarRocks                                                    |
| -------------- | ------------------------------------------------------------ |
| record         | RECORDまたはそのサブフィールド全体をJSONとしてStarRocksにロードします。 |
| enums          | STRING                                                       |
| arrays         | ARRAY                                                        |
| maps           | JSON                                                         |
| union(T, null) | NULLABLE(T)                                                  |
| fixed          | STRING                                                       |

#### 制限事項

- 現在、StarRocksはスキーマの進化をサポートしていません。
- 各Kafkaメッセージは1つのAvroデータレコードだけを含んでいなければなりません。

## ロードジョブとタスクの確認

### ロードジョブの確認

[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)ステートメントを実行して、`example_tbl2_ordertest2`というロードジョブの状態を確認できます。StarRocksは実行状態`State`、統計情報（消費された合計行数とロードされた合計行数）`Statistics`、およびロードジョブの進捗`progress`を返します。

ロードジョブの状態が自動的に**PAUSED**に変更された場合、エラー行数が閾値を超えた可能性があります。この閾値の設定の詳細な手順については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。問題を特定しトラブルシューティングするには、`ReasonOfStateChanged`と`ErrorLogUrls`ファイルを確認できます。問題を修正した後、**PAUSED**ロードジョブを再開するために[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md)ステートメントを実行できます。

ロードジョブの状態が**CANCELLED**になった場合、ロードジョブが例外（例えばテーブルが削除されたなど）に遭遇した可能性があります。問題を特定しトラブルシューティングするために、`ReasonOfStateChanged`と`ErrorLogUrls`ファイルを確認できます。しかし、**CANCELLED**ロードジョブは再開することはできません。

```SQL
MySQL [example_db]> SHOW ROUTINE LOAD FOR example_tbl2_ordertest2 \G
*************************** 1. row ***************************
                  Id: 63013
                Name: example_tbl2_ordertest2
          CreateTime: 2022-08-10 17:09:00
           PauseTime: NULL
EndTime: NULL
              DbName: default_cluster:example_db
           TableName: example_tbl2
               State: RUNNING
      DataSourceType: KAFKA
      CurrentTaskNum: 3
       JobProperties: {"partitions":"*","partial_update":"false","columnToColumnExpr":"commodity_id,customer_name,country,pay_time,pay_dt=from_unixtime(`pay_time`, '%Y%m%d'),price","maxBatchIntervalS":"20","whereExpr":"*","dataFormat":"json","timezone":"Asia/Shanghai","format":"json","json_root":"","strict_mode":"false","jsonpaths":"[\"$.commodity_id\",\"$.customer_name\",\"$.country\",\"$.pay_time\",\"$.price\"]","desireTaskConcurrentNum":"3","maxErrorNum":"0","strip_outer_array":"false","currentTaskConcurrentNum":"3","maxBatchRows":"200000"}
DataSourceProperties: {"topic":"ordertest2","currentKafkaPartitions":"0,1,2,3,4","brokerList":"<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>"}
    CustomProperties: {"kafka_default_offsets":"OFFSET_BEGINNING"}
           Statistic: {"receivedBytes":230,"errorRows":0,"committedTaskNum":1,"loadedRows":2,"loadRowsRate":0,"abortedTaskNum":0,"totalRows":2,"unselectedRows":0,"receivedBytesRate":0,"taskExecuteTimeMs":522}
            Progress: {"0":"1","1":"OFFSET_ZERO","2":"OFFSET_ZERO","3":"OFFSET_ZERO","4":"OFFSET_ZERO"}
ReasonOfStateChanged: 
        ErrorLogUrls: 
            OtherMsg: 

> **注意**
>
> 停止或尚未启动的加载作业无法进行检查。

### 检查加载任务

执行 [SHOW ROUTINE LOAD TASK](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD_TASK.md) 语句来检查加载作业 `example_tbl2_ordertest2` 的加载任务，比如当前正在运行的任务数，被消费的Kafka主题分区和消费进度 `DataSourceProperties`，以及相应的协调器 BE 节点 `BeId`。

```SQL
MySQL [example_db]> SHOW ROUTINE LOAD TASK WHERE JobName = "example_tbl2_ordertest2" \G
*************************** 1. row ***************************
              TaskId: 18c3a823-d73e-4a64-b9cb-b9eced026753
               TxnId: -1
           TxnStatus: UNKNOWN
               JobId: 63013
          CreateTime: 2022-08-10 17:09:05
   LastScheduledTime: 2022-08-10 17:47:27
    ExecuteStartTime: NULL
             Timeout: 60
                BeId: -1
DataSourceProperties: {"1":0,"4":0}
             Message: kafka 中没有新数据，等待20秒后再次进行调度
*************************** 2. row ***************************
              TaskId: f76c97ac-26aa-4b41-8194-a8ba2063eb00
               TxnId: -1
           TxnStatus: UNKNOWN
               JobId: 63013
          CreateTime: 2022-08-10 17:09:05
   LastScheduledTime: 2022-08-10 17:47:26
    ExecuteStartTime: NULL
             Timeout: 60
                BeId: -1
DataSourceProperties: {"2":0}
             Message: kafka 中没有新数据，等待20秒后再次进行调度
*************************** 3. row ***************************
              TaskId: 1a327a34-99f4-4f8d-8014-3cd38db99ec6
               TxnId: -1
           TxnStatus: UNKNOWN
               JobId: 63013
          CreateTime: 2022-08-10 17:09:26
   LastScheduledTime: 2022-08-10 17:47:27
            ExecuteStartTime: NULL
             Timeout: 60
                BeId: -1
DataSourceProperties: {"0":2,"3":0}
             Message: kafka 中没有新数据，等待20秒后再次进行调度
```

## 暂停加载作业

您可以执行 [PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md) 语句来暂停加载作业。在执行该语句后，加载作业的状态将变为 **PAUSED**。然而，它并未停止。您可以执行 [RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) 语句来恢复它。您也可以使用 [SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) 语句来检查其状态。

以下示例暂停加载作业 `example_tbl2_ordertest2`：

```SQL
PAUSE ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## 恢复加载作业

您可以执行 [RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) 语句来恢复已暂停的加载作业。加载作业的状态将暂时变为 **NEED_SCHEDULE**（因为加载作业正在重新调度），然后变为 **RUNNING**。您可以使用 [SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) 语句来检查其状态。

以下示例恢复已暂停的加载作业 `example_tbl2_ordertest2`：

```SQL
RESUME ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## 修改加载作业

在修改加载作业之前，您必须使用 [PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md) 语句来暂停它。之后，您可以执行 [ALTER ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md)。在修改完成之后，您可以执行 [RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) 语句来恢复它，并使用 [SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) 语句来检查其状态。

假设正在运行的 BE 节点数增加至 `6`，而要被消费的 Kafka 主题分区是 `"0,1,2,3,4,5,6,7"`。如果要增加实际加载任务的并发数，您可以执行以下语句，将预期的任务并发数 `desired_concurrent_number` 增加至 `6`（大于或等于正在运行的 BE 节点数），并指定 Kafka 主题分区和初始偏移量。

> **注意**
>
> 由于实际任务并发数由多个参数的最小值确定，您必须确保 FE 动态参数 `max_routine_load_task_concurrent_num` 的值大于或等于 `6`。

```SQL
ALTER ROUTINE LOAD FOR example_tbl2_ordertest2
PROPERTIES
(
    "desired_concurrent_number" = "6"
)
FROM kafka
(
    "kafka_partitions" = "0,1,2,3,4,5,6,7",
    "kafka_offsets" = "OFFSET_BEGINNING,OFFSET_BEGINNING,OFFSET_BEGINNING,OFFSET_BEGINNING,OFFSET_END,OFFSET_END,OFFSET_END,OFFSET_END"
);
```

## 停止加载作业

您可以执行 [STOP ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/STOP_ROUTINE_LOAD.md) 语句来停止加载作业。在执行该语句后，加载作业的状态将变为 **STOPPED**，并且您将无法恢复已停止的加载作业。使用 [SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) 语句无法检查已停止的加载作业的状态。

以下示例停止加载作业 `example_tbl2_ordertest2`：

```SQL
STOP ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## 常见问题解答

请参阅[Routine Load FAQ](../faq/loading/Routine_load_faq.md)。