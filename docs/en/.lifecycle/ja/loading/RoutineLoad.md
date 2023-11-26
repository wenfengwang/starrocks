---
displayed_sidebar: "Japanese"
---

# Apache Kafka®からデータを連続的にロードする

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

このトピックでは、Kafkaメッセージ（イベント）をStarRocksにストリームするためのRoutine Loadジョブの作成方法と、Routine Loadに関するいくつかの基本的な概念について説明します。

ストリームのメッセージをStarRocksに連続的にロードするには、メッセージストリームをKafkaトピックに保存し、Routine Loadジョブを作成してメッセージを消費します。Routine LoadジョブはStarRocksに永続化され、トピックのすべてまたは一部のパーティションでメッセージを消費するための一連のロードタスクを生成し、メッセージをStarRocksにロードします。

Routine Loadジョブは、データがStarRocksにロードされる際にデータの損失や重複が発生しないようにするために、正確に一度だけの配信セマンティクスをサポートしています。

Routine Loadは、データのロード時にデータの変換をサポートし、ロード中にUPSERTおよびDELETE操作によって行われたデータの変更をサポートしています。詳細については、[ロード時のデータ変換](../loading/Etl_in_loading.md)および[ロードを介したデータの変更](../loading/Load_to_Primary_Key_tables.md)を参照してください。

<InsertPrivNote />

## サポートされているデータ形式

Routine Loadは、KafkaクラスタからのCSV、JSON、およびAvro（v3.0.1以降でサポート）形式のデータの消費をサポートしています。

> **注意**
>
> CSVデータの場合、次の点に注意してください：
>
> - テキスト区切り記号として、UTF-8文字列（カンマ（,）、タブ、またはパイプ（|））を使用できます。ただし、長さが50バイトを超えないようにしてください。
> - Null値は`\N`を使用して示します。たとえば、データファイルは3つの列から構成されており、そのデータファイルのレコードは最初と3番目の列にデータを保持していますが、2番目の列にはデータがありません。この場合、2番目の列には`\N`を使用してnull値を示す必要があります。これは、レコードを`a,\N,b`としてコンパイルする必要があることを意味します。`a,,b`は、レコードの2番目の列に空の文字列が含まれていることを示します。

## 基本的な概念

![routine load](../assets/4.5.2-1.png)

### 用語

- **ロードジョブ**

   ルーチンロードジョブは、長時間実行されるジョブです。ステータスがRUNNINGである限り、ロードジョブはトピックの1つまたは複数のパーティションのメッセージを消費し、データをStarRocksにロードするための1つまたは複数の並行ロードタスクを連続的に生成します。

- **ロードタスク**

  ロードジョブは、特定のルールに基づいて複数のロードタスクに分割されます。ロードタスクはデータロードの基本単位です。個々のイベントとして、ロードタスクは[Stream Load](../loading/StreamLoad.md)に基づいたロードメカニズムを実装します。複数のロードタスクは、トピックの異なるパーティションからメッセージを同時に消費し、データをStarRocksにロードします。

### ワークフロー

1. **ルーチンロードジョブを作成します。**
   Kafkaからデータをロードするには、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)ステートメントを実行してルーチンロードジョブを作成する必要があります。FEは、指定したプロパティに従ってステートメントを解析し、ジョブを作成します。

2. **FEはジョブを複数のロードタスクに分割します。**

    FEは、特定のルールに基づいてジョブを複数のロードタスクに分割します。各ロードタスクは個別のトランザクションです。
    分割のルールは次のとおりです。
    - FEは、所望の並行数`desired_concurrent_number`、Kafkaトピックのパーティション数、および生存しているBEノードの数に基づいて、実際の並行数を計算します。
    - FEは、計算された実際の並行数に基づいてジョブをロードタスクに分割し、タスクをタスクキューに配置します。

    各Kafkaトピックは複数のパーティションで構成されています。トピックパーティションとロードタスクの関係は次のとおりです。
    - パーティションはロードタスクに一意に割り当てられ、パーティションのすべてのメッセージはロードタスクによって消費されます。
    - ロードタスクは1つ以上のパーティションからメッセージを消費できます。
    - すべてのパーティションはロードタスクに均等に分散されます。

3. **複数のロードタスクが同時に実行され、複数のKafkaトピックパーティションからメッセージを消費し、データをStarRocksにロードします**

   1. **FEはロードタスクをスケジュールして提出します**：FEは、タスクキューのロードタスクを定期的にスケジュールし、選択したCoordinator BEノードに割り当てます。ロードタスク間のインターバルは、設定項目`max_batch_interval`によって定義されます。FEは、ロードタスクをすべてのBEノードに均等に配布します。詳細については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md#example)を参照してください。

   2. Coordinator BEは、ロードタスクを開始し、パーティション内のメッセージを消費し、データを解析およびフィルタリングします。ロードタスクは、事前に定義されたメッセージの量が消費されるか、事前に定義された時間制限が達成されるまで続きます。メッセージのバッチサイズと時間制限は、FEの設定`max_routine_load_batch_size`および`routine_load_task_consume_second`で定義されます。詳細については、[Configuration](../administration/Configuration.md)を参照してください。Coordinator BEは、その後、メッセージをExecutor BEに配布します。Executor BEは、メッセージをディスクに書き込みます。

         > **注意**
         >
         > StarRocksは、SASL_SSL、SASL、またはSSLのセキュリティ認証メカニズムを介してKafkaへのアクセスをサポートしています。このトピックでは、認証なしでKafkaに接続する例を示します。セキュリティ認証メカニズムを介してKafkaに接続する必要がある場合は、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

4. **FEは新しいロードタスクを生成してデータを連続的にロードします。**
   Executor BEがデータをディスクに書き込んだ後、Coordinator BEはロードタスクの結果をFEに報告します。FEは、結果に基づいて新しいロードタスクを生成してデータを連続的にロードします。または、FEは失敗したタスクを再試行して、StarRocksにロードされるデータが損失または重複しないようにします。

## ルーチンロードジョブの作成

次の3つの例では、CSV形式、JSON形式、およびAvro形式のデータをKafkaから消費し、ルーチンロードジョブを作成してデータをStarRocksにロードする方法について説明します。詳細な構文とパラメータの説明については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

### CSV形式のデータをロードする

このセクションでは、Kafkaクラスタのトピック`ordertest1`でCSV形式のデータを消費し、データをStarRocksにロードするためのルーチンロードジョブを作成する方法について説明します。

#### データセットの準備

Kafkaクラスタのトピック`ordertest1`にCSV形式のデータセットがあるとします。データセットの各メッセージには、注文ID、支払い日、顧客名、国籍、性別、価格の6つのフィールドが含まれています。

```Plain
2020050802,2020-05-08,Johann Georg Faust,Deutschland,male,895
2020050802,2020-05-08,Julien Sorel,France,male,893
2020050803,2020-05-08,Dorian Grey,UK,male,1262
2020050901,2020-05-09,Anna Karenina",Russia,female,175
2020051001,2020-05-10,Tess Durbeyfield,US,female,986
2020051101,2020-05-11,Edogawa Conan,japan,male,8924
```

#### テーブルの作成

CSV形式のデータのフィールドに基づいて、データベース`example_db`のテーブル`example_tbl1`を作成します。次の例では、CSV形式のデータのカスタマージェンダーのフィールドを除外して、5つのフィールドでテーブル`example_tbl1`を作成します。

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
> v2.5.7以降、StarRocksはテーブルまたはパーティションを作成する際にバケット数（BUCKETS）を自動的に設定できます。バケット数を手動で設定する必要はありません。詳細については、[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

#### ルーチンロードジョブの提出

次のステートメントを実行して、ルーチンロードジョブ`example_tbl1_ordertest1`を提出し、トピック`ordertest1`のメッセージを消費し、データをテーブル`example_tbl1`にロードするためのロードタスクを作成します。ロードタスクは、トピックの指定されたパーティションの初期オフセットからメッセージを消費します。

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
    "kafka_partitions" = "0,1,2,3,4",
    "property.kafka_default_offsets" = "OFFSET_BEGINNING"
);
```

ロードジョブを提出した後、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)ステートメントを実行して、ロードジョブのステータスを確認できます。

- **ロードジョブ名**

  テーブルには複数のロードジョブがある場合があります。したがって、各テーブルのロードジョブを区別するために、ロードジョブには対応するKafkaトピックとロードジョブが提出された時刻を含めることをお勧めします。

- **カラムセパレータ**

  プロパティ`COLUMN TERMINATED BY`は、CSV形式のデータのカラムセパレータを定義します。デフォルトは`\t`です。

- **Kafkaトピックのパーティションとオフセット**

  メッセージを消費するためのパーティションとオフセットを指定するために、プロパティ`kafka_partitions`および`kafka_offsets`を指定できます。たとえば、ロードジョブをトピック`ordertest1`のKafkaパーティション`"0,1,2,3,4"`からすべての初期オフセットでメッセージを消費するようにしたい場合は、次のようにプロパティを指定できます。ロードジョブをKafkaパーティション`"0,1,2,3,4"`からメッセージを消費する場合、各パーティションに個別の開始オフセットを指定する必要がある場合は、次のように構成できます。

    ```SQL
    "kafka_partitions" ="0,1,2,3,4",
    "kafka_offsets" = "OFFSET_BEGINNING, OFFSET_END, 1000, 2000, 3000"
    ```

  または、すべてのパーティションのデフォルトオフセットをプロパティ`property.kafka_default_offsets`で設定することもできます。

    ```SQL
    "kafka_partitions" ="0,1,2,3,4",
    "property.kafka_default_offsets" = "OFFSET_BEGINNING"
    ```

  詳細については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

- **データのマッピングと変換**

  CSV形式のデータとStarRocksテーブルの間のマッピングと変換の関係を指定するには、`COLUMNS`パラメータを使用する必要があります。

  **データのマッピング**:

  - StarRocksは、CSV形式のデータの列を抽出し、`COLUMNS`パラメータで宣言されたフィールドに**順番に**マッピングします。

  - StarRocksは、`COLUMNS`パラメータで宣言されたフィールドを抽出し、StarRocksテーブルの列に**名前で**マッピングします。

  **データの変換**:

  この例では、CSV形式のデータからカスタマージェンダーの列を除外するため、`COLUMNS`パラメータの`temp_gender`フィールドをこのフィールドのプレースホルダーとして使用します。他のフィールドは、StarRocksテーブル`example_tbl1`の列に直接マッピングされます。

  データ変換の詳細については、[ロード時のデータ変換](./Etl_in_loading.md)を参照してください。

    > **注意**
    >
    > CSV形式のデータの名前、数、および順序がStarRocksテーブルと完全に一致する場合、`COLUMNS`パラメータを指定する必要はありません。

- **タスクの並行性**

  多くのKafkaトピックパーティションと十分なBEノードがある場合、タスクの並行性を高めることでロードを高速化できます。

  実際のロードタスクの並行性を高めるには、ルーチンロードジョブを作成する際に`desired_concurrent_number`を増やすことができます。また、FEの動的設定項目`max_routine_load_task_concurrent_num`（デフォルトの最大ロードタスクの並行数）を大きな値に設定することもできます。`max_routine_load_task_concurrent_num`の詳細については、[FEの設定項目](../administration/Configuration.md#fe-configuration-items)を参照してください。

  実際のタスクの並行性は、生存しているBEノードの数、事前に指定されたKafkaトピックのパーティション数、および`desired_concurrent_number`および`max_routine_load_task_concurrent_num`の値のうち最小の値で定義されます。

  この例では、生存しているBEノードの数は`5`、事前に指定されたKafkaトピックのパーティション数は`5`、`max_routine_load_task_concurrent_num`の値は`5`です。実際のロードタスクの並行性を高めるには、`desired_concurrent_number`をデフォルト値`3`から`5`に増やすことができます。

  プロパティについての詳細は、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。ロードの高速化についての詳細な手順については、[Routine Load FAQ](../faq/loading/Routine_load_faq.md)を参照してください。

### JSON形式のデータをロードする

このセクションでは、Kafkaクラスタのトピック`ordertest2`でJSON形式のデータを消費し、データをStarRocksにロードするためのルーチンロードジョブを作成する方法について説明します。

#### データセットの準備

Kafkaクラスタのトピック`ordertest2`にJSON形式のデータセットがあるとします。データセットには、商品ID、顧客名、国籍、支払い時間、価格の5つのキーが含まれています。また、支払い時間の列をDATE型に変換し、StarRocksテーブルの`pay_dt`列にロードすることを目指しています。

```JSON
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}
```

> **注意** 1つのKafkaメッセージには、1つのJSONオブジェクトのみを含める必要があります。それ以外の場合、JSONの解析エラーが返されます。

#### テーブルの作成

JSON形式のデータのキーに基づいて、データベース`example_db`のテーブル`example_tbl2`を作成します。

```SQL
CREATE TABLE `example_tbl2` ( 
    `commodity_id` varchar(26) NULL COMMENT "Commodity ID", 
    `customer_name` varchar(26) NULL COMMENT "Customer name", 
    `country` varchar(26) NULL COMMENT "Country", 
    `pay_time` bigint(20) NULL COMMENT "Payment time", 
    `pay_dt` date NULL COMMENT "Payment date", 
    `price`double SUM NULL COMMENT "Price"
) 
ENGINE=OLAP
AGGREGATE KEY(`commodity_id`,`customer_name`,`country`,`pay_time`,`pay_dt`) 
DISTRIBUTED BY HASH(`commodity_id`); 
```

> **注意**
>
> v2.5.7以降、StarRocksはテーブルまたはパーティションを作成する際にバケット数（BUCKETS）を自動的に設定できます。バケット数を手動で設定する必要はありません。詳細については、[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

#### ルーチンロードジョブの提出

次のステートメントを実行して、ルーチンロードジョブ`example_tbl2_ordertest2`を提出し、トピック`ordertest2`のメッセージを消費し、データをテーブル`example_tbl2`にロードするためのロードタスクを作成します。ロードタスクは、トピックの指定されたパーティションの初期オフセットからメッセージを消費します。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl2_ordertest2 ON example_tbl2
COLUMNS(commodity_id, customer_name, country, pay_time, price, pay_dt=from_unixtime(pay_time, '%Y%m%d'))
PROPERTIES
(
    "desired_concurrent_number" = "5",
    "format" = "json",
    "jsonpaths" = "[\"$.commodity_id\",\"$.customer_name\",\"$.country\",\"$.pay_time\",\"$.price\"]"
 )
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest2",
    "kafka_partitions" ="0,1,2,3,4",
    "property.kafka_default_offsets" = "OFFSET_BEGINNING"
);
```

ロードジョブを提出した後、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)ステートメントを実行して、ロードジョブのステータスを確認できます。

- **データ形式**

  データ形式がJSONであることを定義するために、`PROPERTIES`句で`"format" = "json"`を指定する必要があります。

- **スキーマレジストリ**

  Avroスキーマが登録されているスキーマレジストリのURLを指定するために、`confluent.schema.registry.url`を設定する必要があります。StarRocksは、このURLを使用してAvroスキーマを取得します。形式は次のようになります。

  ```Plaintext
  confluent.schema.registry.url = http[s]://[<schema-registry-api-key>:<schema-registry-api-secret>@]<hostname|ip address>[:<port>]
  ```

- **データのマッピングと変換**

  Avro形式のデータとStarRocksテーブルの間のマッピングと変換の関係を指定するには、`COLUMNS`パラメータとプロパティ`jsonpaths`を指定する必要があります。`COLUMNS`パラメータで指定されたフィールドの順序は、`jsonpaths`プロパティのフィールドの順序と一致し、フィールドの名前はStarRocksテーブルのフィールドの名前と一致する必要があります。`jsonpaths`プロパティは、Avroデータから必要なフィールドを抽出するために使用されます。これらのフィールドは、`COLUMNS`プロパティで名前が付けられます。

  データの変換の詳細については、[ロード時のデータ変換](./Etl_in_loading.md)を参照してください。

  > **注意**
  >
  > Avroレコードのフィールドの名前と数がStarRocksテーブルの列と完全に一致する場合、`COLUMNS`パラメータを指定する必要はありません。

ロードジョブを提出した後、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)ステートメントを実行して、ロードジョブのステータスを確認できます。

#### データ型のマッピング

ロードしたいAvroデータのフィールドとStarRocksテーブルの列との間のデータ型のマッピングは次のとおりです。

##### プリミティブ型

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
| record         | レコードまたはそのサブフィールド全体をJSONとしてStarRocksにロードします。 |
| enums          | STRING                                                       |
| arrays         | ARRAY                                                        |
| maps           | JSON                                                         |
| union(T, null) | NULLABLE(T)                                                  |
| fixed          | STRING                                                       |

#### 制限事項

- 現在、StarRocksはスキーマの進化をサポートしていません。
- 各Kafkaメッセージには、1つのAvroデータレコードのみを含める必要があります。

## ロードジョブとタスクの確認

### ロードジョブの確認

[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)ステートメントを実行して、ロードジョブ`example_tbl2_ordertest2`のステータスを確認します。StarRocksは、実行状態`State`、統計情報（総消費行数と総ロード行数を含む）`Statistics`、およびロードジョブの進行状況`progress`を返します。

ロードジョブのステータスが自動的に**PAUSED**に変更された場合、エラーレコードの数が閾値を超えた可能性があります。この閾値を設定する方法の詳細については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。問題を特定してトラブルシューティングするために、`ReasonOfStateChanged`および`ErrorLogUrls`のファイルを確認できます。問題を修正した後、[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md)ステートメントを実行して**PAUSED**のロードジョブを再開できます。

ロードジョブのステータスが**CANCELLED**になった場合、ロードジョブが例外（テーブルが削除されたなど）に遭遇した可能性があります。問題を特定してトラブルシューティングするために、`ReasonOfStateChanged`および`ErrorLogUrls`のファイルを確認できます。ただし、**CANCELLED**のロードジョブを再開することはできません。

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
```

> **注意**
>
> 停止またはまだ開始していないロードジョブを確認することはできません。

### ロードタスクを確認する

`example_tbl2_ordertest2`というロードジョブのロードタスクを確認するには、[SHOW ROUTINE LOAD TASK](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD_TASK.md)ステートメントを実行します。現在実行中のタスクの数、消費されているKafkaトピックのパーティション、消費の進捗状況 `DataSourceProperties`、および対応するコーディネータBEノード `BeId`などが確認できます。

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
             Message: there is no new data in kafka, wait for 20 seconds to schedule again
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
             Message: there is no new data in kafka, wait for 20 seconds to schedule again
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
             Message: there is no new data in kafka, wait for 20 seconds to schedule again
```

## ロードジョブを一時停止する

ロードジョブを一時停止するには、[PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md)ステートメントを実行します。ステートメントが実行されると、ロードジョブの状態は**PAUSED**になりますが、停止されていません。再開するには、[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md)ステートメントを実行することができます。また、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)ステートメントでその状態を確認することもできます。

以下の例では、ロードジョブ`example_tbl2_ordertest2`を一時停止します：

```SQL
PAUSE ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## ロードジョブを再開する

ロードジョブを一時停止した後、[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md)ステートメントを実行して、一時停止されたロードジョブを再開することができます。ロードジョブの状態は一時的に**NEED_SCHEDULE**になります（ロードジョブが再スケジュールされているため）、その後**RUNNING**になります。[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)ステートメントでその状態を確認することができます。

以下の例では、一時停止されたロードジョブ`example_tbl2_ordertest2`を再開します：

```SQL
RESUME ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## ロードジョブを変更する

ロードジョブを変更する前に、[PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md)ステートメントで一時停止する必要があります。その後、[ALTER ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md)を実行することができます。変更した後、[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md)ステートメントを実行して再開し、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)ステートメントでその状態を確認することができます。

BEノードの数が`6`に増え、消費するKafkaトピックのパーティションが`"0,1,2,3,4,5,6,7"`になったとします。実際のロードタスクの並行性を増やしたい場合は、次のステートメントを実行して、希望するタスクの並行性 `desired_concurrent_number` を`6`（BEノードの数以上）に増やし、Kafkaトピックのパーティションと初期オフセットを指定します。

> **注意**
>
> 実際のタスクの並行性は、複数のパラメータの最小値によって決まるため、FEの動的パラメータ `max_routine_load_task_concurrent_num` の値が`6`以上であることを確認してください。

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

## ロードジョブを停止する

ロードジョブを停止するには、[STOP ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/STOP_ROUTINE_LOAD.md)ステートメントを実行します。ステートメントが実行されると、ロードジョブの状態は**STOPPED**になり、停止したロードジョブを再開することはできません。停止したロードジョブの状態を[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)ステートメントで確認することはできません。

以下の例では、ロードジョブ`example_tbl2_ordertest2`を停止します：

```SQL
STOP ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## よくある質問

[ローディングのよくある質問](../faq/loading/Routine_load_faq.md)をご覧ください。
