---
displayed_sidebar: English
---

# Apache Kafka® からデータを継続的にロードする

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

このトピックでは、Kafka メッセージ（イベント）を StarRocks にストリーミングするルーチンロードジョブの作成方法を紹介し、ルーチンロードに関するいくつかの基本概念について説明します。

ストリームのメッセージを StarRocks に継続的にロードするには、メッセージストリームを Kafka トピックに保存し、メッセージを消費するルーチンロードジョブを作成します。ルーチンロードジョブは StarRocks に保持され、トピック内の全てまたは一部のパーティションのメッセージを消費する一連のロードタスクを生成し、メッセージを StarRocks にロードします。

ルーチンロードジョブは exactly-once 配信セマンティクスをサポートし、StarRocks にロードされたデータが失われたり重複したりしないことを保証します。

ルーチンロードは、データロード時のデータ変換をサポートし、データロード中に UPSERT および DELETE 操作によるデータ変更をサポートします。詳細については、[データロード時の変換](../loading/Etl_in_loading.md) および [ロードを通じたデータ変更](../loading/Load_to_Primary_Key_tables.md) を参照してください。

<InsertPrivNote />

## サポートされているデータ形式

Routine Load は現在、Kafka クラスターから CSV、JSON、および Avro（v3.0.1 以降でサポート）形式のデータを消費することをサポートしています。

> **注記**
>
> CSV データについては、以下の点に注意してください：
>
> - コンマ（,）、タブ、パイプ（|）など、50 バイトを超えない長さの UTF-8 文字列をテキスト区切り文字として使用できます。
> - Null 値は `\N` を使用して表されます。例えば、データファイルが 3 つの列で構成されており、そのデータファイルのレコードが 1 列目と 3 列目にデータを保持しているが 2 列目にはデータがない場合、2 列目で null 値を示すために `\N` を使用する必要があります。つまり、レコードは `a,\N,b` としてコンパイルされるべきで、`a,,b` ではありません。`a,,b` はレコードの 2 列目に空文字列があることを示します。

## 基本概念

![ルーチンロード](../assets/4.5.2-1.png)

### 用語集

- **ロードジョブ**

   ルーチンロードジョブは長時間実行されるジョブです。ステータスが RUNNING である限り、ロードジョブは一つまたは複数の同時ロードタスクを継続的に生成し、Kafka クラスタのトピック内のメッセージを消費し、データを StarRocks にロードします。

- **ロードタスク**

  ロードジョブは特定のルールに基づいて複数のロードタスクに分割されます。ロードタスクはデータロードの基本単位です。個別のイベントとして、ロードタスクは [Stream Load](../loading/StreamLoad.md) に基づいてロードメカニズムを実装します。複数のロードタスクがトピックの異なるパーティションからメッセージを同時に消費し、データを StarRocks にロードします。

### ワークフロー

1. **ルーチンロードジョブを作成する。**
   Kafka からデータをロードするには、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) ステートメントを実行してルーチンロードジョブを作成する必要があります。FE はステートメントを解析し、指定されたプロパティに基づいてジョブを作成します。

2. **FE はジョブを複数のロードタスクに分割します。**

    FE は特定のルールに基づいてジョブを複数のロードタスクに分割します。各ロードタスクは個別のトランザクションです。
    分割ルールは以下の通りです：
    - FE は、希望する同時実行数 `desired_concurrent_number`、Kafka トピック内のパーティション数、および生存している BE ノードの数に基づいて、ロードタスクの実際の同時実行数を計算します。
    - FE は計算された実際の同時実行数に基づいてジョブをロードタスクに分割し、タスクキューにタスクを配置します。

    各 Kafka トピックは複数のパーティションで構成されています。トピックパーティションとロードタスクの関係は以下の通りです：
    - パーティションはロードタスクに一意に割り当てられ、そのパーティションからの全てのメッセージがロードタスクによって消費されます。
    - ロードタスクは一つ以上のパーティションからメッセージを消費することができます。
    - 全てのパーティションはロードタスク間で均等に分配されます。

3. **複数のロードタスクが同時に実行され、複数の Kafka トピックパーティションからのメッセージを消費し、StarRocks にデータをロードします。**

   1. **FE はロードタスクをスケジュールし、送信します**：FE はキュー内のロードタスクを定期的にスケジュールし、選択されたコーディネータ BE ノードに割り当てます。ロードタスク間の間隔は `max_batch_interval` 設定項目によって定義されます。FE はロードタスクを全 BE ノードに均等に配分します。`max_batch_interval` の詳細については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md#example) を参照してください。

   2. コーディネータ BE はロードタスクを開始し、パーティション内のメッセージを消費し、データを解析およびフィルタリングします。ロードタスクは、事前に定義されたメッセージ量が消費されるか、事前に定義された時間制限に達するまで続きます。メッセージバッチサイズと時間制限は、FE の設定 `max_routine_load_batch_size` と `routine_load_task_consume_second` で定義されます。詳細については、[設定](../administration/BE_configuration.md) を参照してください。その後、コーディネータ BE はメッセージをエグゼキュータ BE に配布します。エグゼキュータ BE はメッセージをディスクに書き込みます。

         > **注記**
         >
         > StarRocks は、セキュリティ認証メカニズム SASL_SSL、SASL、SSL、または認証なしで Kafka へのアクセスをサポートしています。このトピックでは、認証なしで Kafka への接続を例として取り上げます。セキュリティ認証メカニズムを使用して Kafka に接続する必要がある場合は、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を参照してください。

4. **FE はデータを継続的にロードするために新しいロードタスクを生成します。**
   エグゼキュータ BE がデータをディスクに書き込んだ後、コーディネータ BE はロードタスクの結果を FE に報告します。その結果に基づいて、FE はデータを継続的にロードするための新しいロードタスクを生成します。または、FE は失敗したタスクを再試行して、StarRocks にロードされたデータが失われたり重複したりしないようにします。

## ルーチンロードジョブを作成する

以下の 3 つの例では、Kafka で CSV 形式、JSON 形式、Avro 形式のデータを消費し、ルーチンロードジョブを作成してデータを StarRocks にロードする方法を説明します。構文およびパラメータの詳細な説明については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を参照してください。

### CSV 形式のデータをロードする

このセクションでは、Kafka クラスタで CSV 形式のデータを消費するルーチンロードジョブを作成し、そのデータを StarRocks にロードする方法について説明します。

#### データセットを準備する

Kafka クラスタのトピック `ordertest1` に CSV 形式のデータセットがあるとします。データセット内のすべてのメッセージには、注文 ID、支払い日、顧客名、国籍、性別、価格の 6 つのフィールドが含まれています。

```Plain
2020050802,2020-05-08,Johann Georg Faust,Deutschland,male,895
2020050802,2020-05-08,Julien Sorel,France,male,893
2020050803,2020-05-08,Dorian Grey,UK,male,1262
2020050901,2020-05-09,Anna Karenina,Russia,female,175
2020051001,2020-05-10,Tess Durbeyfield,US,female,986
```

2020051101,2020-05-11,Edogawa Conan,japan,male,8924
```

#### テーブルの作成

CSV形式のデータのフィールドに基づいて、データベース`example_db`にテーブル`example_tbl1`を作成します。以下の例では、CSV形式のデータから顧客の性別フィールドを除外した5つのフィールドを持つテーブルを作成します。

```SQL
CREATE TABLE example_db.example_tbl1 ( 
    `order_id` bigint NOT NULL COMMENT "Order ID",
    `pay_dt` date NOT NULL COMMENT "Payment date", 
    `customer_name` varchar(26) NULL COMMENT "Customer name", 
    `nationality` varchar(26) NULL COMMENT "Nationality", 
    `price` double NULL COMMENT "Price"
) 
ENGINE=OLAP 
DUPLICATE KEY (order_id, pay_dt) 
DISTRIBUTED BY HASH(`order_id`); 
```

> **注意**
>
> v2.5.7以降、StarRocksはテーブル作成時やパーティション追加時にバケット数(BUCKETS)を自動的に設定できるようになりました。バケット数を手動で設定する必要はありません。詳細は[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

#### ルーチンロードジョブのサブミット

以下のステートメントを実行して、トピック`ordertest1`のメッセージを消費し、テーブル`example_tbl1`にデータをロードするルーチンロードジョブ`example_tbl1_ordertest1`をサブミットします。ロードタスクは、トピックの指定されたパーティションの最初のオフセットからメッセージを消費します。

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

ロードジョブをサブミットした後、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)ステートメントを実行してロードジョブの状態を確認できます。

- **ロードジョブ名**

  一つのテーブルに複数のロードジョブが存在する可能性があるため、ロードジョブには対応するKafkaトピックとロードジョブがサブミットされた時刻を含む名前を付けることを推奨します。これにより、各テーブルのロードジョブを区別するのに役立ちます。

- **カラムセパレータ**

  `COLUMN TERMINATED BY`プロパティは、CSV形式のデータのカラムセパレータを定義します。デフォルトは`\t`です。

- **Kafkaトピックのパーティションとオフセット**

  `kafka_partitions`と`kafka_offsets`プロパティを指定して、メッセージを消費するパーティションとオフセットを指定できます。例えば、トピック`ordertest1`のKafkaパーティション`"0,1,2,3,4"`からすべてのメッセージを最初のオフセットで消費する場合、次のようにプロパティを指定できます。各パーティションに個別の開始オフセットを指定する必要がある場合は、次のように設定できます：

    ```SQL
    "kafka_partitions" = "0,1,2,3,4",
    "kafka_offsets" = "OFFSET_BEGINNING, OFFSET_END, 1000, 2000, 3000"
    ```

  すべてのパーティションのデフォルトオフセットを`property.kafka_default_offsets`プロパティで設定することもできます。

    ```SQL
    "kafka_partitions" = "0,1,2,3,4",
    "property.kafka_default_offsets" = "OFFSET_BEGINNING"
    ```

  詳細は[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

- **データマッピングと変換**

  CSV形式のデータとStarRocksテーブル間のマッピングと変換関係を指定するには、`COLUMNS`パラメータを使用する必要があります。

  **データマッピング：**

  - StarRocksはCSV形式のデータからカラムを抽出し、`COLUMNS`パラメータで宣言されたフィールドに**順番に**マッピングします。

  - StarRocksは`COLUMNS`パラメータで宣言されたフィールドを抽出し、それらをStarRocksテーブルのカラムに**名前で**マッピングします。

  **データ変換：**

  この例ではCSV形式のデータから顧客の性別カラムを除外しているため、`COLUMNS`パラメータの`temp_gender`フィールドがこのフィールドのプレースホルダとして使用されます。他のフィールドはStarRocksテーブル`example_tbl1`のカラムに直接マッピングされます。

  データ変換の詳細については、[読み込み時のデータ変換](./Etl_in_loading.md)を参照してください。

    > **注記**
    >
    > CSV形式のデータのカラムの名前、数、順序がStarRocksテーブルのカラムと完全に一致する場合は、`COLUMNS`パラメータを指定する必要はありません。

- **タスクの同時実行性**

  Kafkaトピックのパーティションが多く、十分なBEノードがある場合、タスクの同時実行性を高めることでロードを加速できます。

  実際のロードタスクの同時実行性を高めるには、ルーチンロードジョブを作成する際に希望するロードタスクの同時実行数`desired_concurrent_number`を増やすことができます。また、FEの動的設定項目`max_routine_load_task_concurrent_num`（デフォルトの最大ロードタスク同時実行数）をより大きな値に設定することもできます。`max_routine_load_task_concurrent_num`の詳細については、[FE設定項目](../administration/FE_configuration.md#fe-configuration-items)を参照してください。

  実際のタスクの同時実行性は、生きているBEノードの数、事前に指定されたKafkaトピックのパーティションの数、および`desired_concurrent_number`と`max_routine_load_task_concurrent_num`の値の最小値によって定義されます。

  この例では、生きているBEノードの数は`5`、事前に指定されたKafkaトピックのパーティションの数は`5`、`max_routine_load_task_concurrent_num`の値は`5`です。実際のロードタスクの同時実行性を高めるには、デフォルト値`3`から`5`に`desired_concurrent_number`を増やすことができます。

  プロパティの詳細については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。ロードを加速する詳細な手順については、[ルーチンロードFAQ](../faq/loading/Routine_load_faq.md)を参照してください。

### JSON形式のデータのロード

このセクションでは、Kafkaクラスター内のJSON形式のデータを消費し、StarRocksにロードするルーチンロードジョブを作成する方法について説明します。

#### データセットの準備

Kafkaクラスターのトピック`ordertest2`にJSON形式のデータセットがあるとします。データセットには、商品ID、顧客名、国籍、支払い時間、価格の6つのキーが含まれています。さらに、支払い時間のカラムをDATE型に変換し、StarRocksテーブルの`pay_dt`カラムにロードしたいとします。

```JSON
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US", "pay_time": 1589191487, "price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK", "pay_time": 1589191487, "price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry", "country": "France", "pay_time": 1589191487, "price": 895}
```

> **注意** 各行のJSONオブジェクトは1つのKafkaメッセージに含まれている必要があり、そうでない場合はJSON解析エラーが返されます。

#### テーブルの作成

JSON形式のデータのキーに基づいて、データベース`example_db`にテーブル`example_tbl2`を作成します。

```SQL
CREATE TABLE `example_tbl2` ( 
    `commodity_id` varchar(26) NULL COMMENT "Commodity ID", 
    `customer_name` varchar(26) NULL COMMENT "Customer name", 
    `country` varchar(26) NULL COMMENT "Country", 
    `pay_time` bigint(20) NULL COMMENT "Payment time", 
    `pay_dt` date NULL COMMENT "Payment date", 
    `price` double SUM NULL COMMENT "Price"
) 
ENGINE=OLAP
AGGREGATE KEY(`commodity_id`, `customer_name`, `country`, `pay_time`, `pay_dt`) 
DISTRIBUTED BY HASH(`commodity_id`); 
```

> **注意**
>
> v2.5.7以降、StarRocksはテーブル作成時やパーティション追加時にバケット数(BUCKETS)を自動的に設定できるようになりました。バケット数を手動で設定する必要はありません。詳細は[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

#### Routine Loadジョブのサブミット

以下のステートメントを実行して、`example_tbl2_ordertest2`という名前のRoutine Loadジョブをサブミットし、トピック`ordertest2`のメッセージを消費してテーブル`example_tbl2`にデータをロードします。ロードタスクは、トピックの指定されたパーティションの初期オフセットからメッセージを消費します。

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

ロードジョブをサブミットした後、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)ステートメントを実行してロードジョブのステータスを確認できます。

- **データフォーマット**

  データフォーマットがJSONであることを定義するために、`PROPERTIES`句で`"format" = "json"`を指定する必要があります。

- **データマッピングと変換**

  JSON形式のデータとStarRocksテーブル間のマッピングと変換関係を指定するには、`COLUMNS`パラメータと`jsonpaths`プロパティを指定する必要があります。`COLUMNS`パラメータで指定されたフィールドの順序は、JSON形式のデータのそれと一致し、フィールド名はStarRocksテーブルのそれと一致する必要があります。`jsonpaths`プロパティは、JSONデータから必要なフィールドを抽出するために使用されます。これらのフィールドは、`COLUMNS`プロパティによって命名されます。

  例では、支払い時間フィールドをDATEデータ型に変換し、StarRocksテーブルの`pay_dt`列にデータをロードする必要があるため、`from_unixtime`関数を使用します。他のフィールドは、`example_tbl2`テーブルのフィールドに直接マッピングされます。

  **データマッピング:**

  - StarRocksは、JSON形式のデータの`name`および`code`キーを抽出し、`jsonpaths`プロパティで宣言されたキーにマッピングします。

  - StarRocksは、`jsonpaths`プロパティで宣言されたキーを抽出し、`COLUMNS`パラメータで宣言されたフィールドに**順番に**マッピングします。

  - StarRocksは、`COLUMNS`パラメータで宣言されたフィールドを抽出し、それらを**名前で**StarRocksテーブルの列にマッピングします。

  **データ変換**:

  - 例では、`pay_time`キーをDATEデータ型に変換し、StarRocksテーブルの`pay_dt`列にデータをロードする必要があるため、`COLUMNS`パラメータで`from_unixtime`関数を使用します。他のフィールドは、`example_tbl2`テーブルのフィールドに直接マッピングされます。

  - また、例ではJSON形式のデータから顧客性別の列を除外しているため、`COLUMNS`パラメータの`temp_gender`フィールドがこのフィールドのプレースホルダーとして使用されます。他のフィールドは、`example_tbl1`テーブルの列に直接マッピングされます。

    データ変換の詳細については、[読み込み時のデータ変換](./Etl_in_loading.md)を参照してください。

    > **注記**
    >
    > JSONオブジェクト内のキーの名前と数がStarRocksテーブルのフィールドと完全に一致する場合、`COLUMNS`パラメータを指定する必要はありません。

### Avro形式データのロード

v3.0.1以降、StarRocksはRoutine Loadを使用してAvroデータのロードをサポートしています。

#### データセットの準備

##### Avroスキーマ

1. 以下のAvroスキーマファイル`avro_schema.avsc`を作成します：

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

2. [Schema Registry](https://docs.confluent.io/cloud/current/get-started/schema-registry.html#create-a-schema)にAvroスキーマを登録します。

##### Avroデータ

Avroデータを準備し、Kafkaトピック`topic_0`に送信します。

#### テーブルの作成

Avroデータのフィールドに基づいて、StarRocksクラスタのターゲットデータベース`example_db`にテーブル`sensor_log`を作成します。テーブルの列名はAvroデータのフィールド名と一致する必要があります。テーブル列とAvroデータフィールド間のデータタイプマッピングについては、[データタイプのマッピング](#Data-types-mapping)を参照してください。

```SQL
CREATE TABLE example_db.sensor_log ( 
    `id` bigint NOT NULL COMMENT "sensor id",
    `name` varchar(26) NOT NULL COMMENT "sensor name", 
    `checked` boolean NOT NULL COMMENT "checked", 
    `data` double NULL COMMENT "sensor data", 
    `sensor_type` varchar(26) NOT NULL COMMENT "sensor type"
) 
ENGINE=OLAP 
DUPLICATE KEY (`id`) 
DISTRIBUTED BY HASH(`id`); 
```

> **通知**
>
> v2.5.7以降、StarRocksはテーブル作成時やパーティション追加時にバケット数(BUCKETS)を自動的に設定できるようになりました。バケット数を手動で設定する必要はありません。詳細は[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

#### Routine Loadジョブのサブミット

以下のステートメントを実行して、`sensor_log_load_job`という名前のRoutine Loadジョブをサブミットし、Kafkaトピック`topic_0`のAvroメッセージを消費してデータベース`sensor`のテーブル`sensor_log`にデータをロードします。ロードジョブは、トピックの指定されたパーティションの初期オフセットからメッセージを消費します。

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

- データフォーマット

  データフォーマットがAvroであることを定義するために、`PROPERTIES`句で`"format" = "avro"`を指定する必要があります。

- スキーマレジストリ

  `confluent.schema.registry.url`を設定して、Avroスキーマが登録されているSchema RegistryのURLを指定します。StarRocksはこのURLを使用してAvroスキーマを取得します。形式は以下の通りです：

  ```Plaintext
  confluent.schema.registry.url = http[s]://[<schema-registry-api-key>:<schema-registry-api-secret>@]<hostname|ip address>[:<port>]
  ```

- データマッピングと変換

  Avro形式のデータとStarRocksテーブル間のマッピングと変換関係を指定するには、`COLUMNS`パラメータと`jsonpaths`プロパティを指定する必要があります。`COLUMNS`パラメータで指定されたフィールドの順序は、`jsonpaths`プロパティのフィールドの順序と一致し、フィールド名はStarRocksテーブルのそれと一致する必要があります。`jsonpaths`プロパティは、Avroデータから必要なフィールドを抽出するために使用されます。これらのフィールドは、`COLUMNS`プロパティによって命名されます。

  データ変換の詳細については、[読み込み時のデータ変換](https://docs.starrocks.io/en-us/latest/loading/Etl_in_loading)を参照してください。

  > 注記
  >
  > `COLUMNS` パラメータを指定する必要はありません。Avro レコードのフィールドの名前と数が StarRocks テーブルのカラムと完全に一致する場合。

ロードジョブを送信した後、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) ステートメントを実行してロードジョブの状態を確認できます。

#### データ型のマッピング

Avro データフィールドと StarRocks テーブルカラム間のデータ型マッピングは以下の通りです：

##### プリミティブ型

| Avro    | StarRocks |
| ------- | --------- |
| null    | NULL      |
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
| record         | StarRocks に JSON として RECORD 全体またはそのサブフィールドをロードします。 |
| enums          | STRING                                                       |
| arrays         | ARRAY                                                        |
| maps           | JSON                                                         |
| union(T, null) | NULLABLE(T)                                                  |
| fixed          | STRING                                                       |

#### 制限事項

- 現在、StarRocks はスキーマ進化をサポートしていません。
- 各 Kafka メッセージには単一の Avro データレコードのみを含める必要があります。

## ロードジョブとタスクの確認

### ロードジョブの確認

[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) ステートメントを実行して、ロードジョブ `example_tbl2_ordertest2` の状態を確認します。StarRocks は実行状態 `State`、統計情報（消費された合計行数とロードされた合計行数を含む）`Statistics`、およびロードジョブの進捗 `Progress` を返します。

ロードジョブの状態が自動的に **PAUSED** に変更された場合、エラー行数がしきい値を超えた可能性があります。このしきい値の設定方法については [CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を参照してください。`ReasonOfStateChanged` と `ErrorLogUrls` を確認して問題を特定し、トラブルシューティングを行います。問題を解決した後、[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) ステートメントを実行して **PAUSED** 状態のロードジョブを再開できます。

ロードジョブの状態が **CANCELLED** である場合、ロードジョブが例外（テーブルが削除されたなど）に遭遇した可能性があります。`ReasonOfStateChanged` と `ErrorLogUrls` を確認して問題を特定し、トラブルシューティングを行います。ただし、**CANCELLED** 状態のロードジョブを再開することはできません。

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
> 停止しているか、まだ開始していないロードジョブは確認できません。

### ロードタスクの確認

[SHOW ROUTINE LOAD TASK](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD_TASK.md) ステートメントを実行して、ロードジョブ `example_tbl2_ordertest2` のロードタスクを確認します。現在実行中のタスク数、消費されている Kafka トピックのパーティション、消費進行状況 `DataSourceProperties`、および対応する Coordinator BE ノード `BeId` などが確認できます。

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

## ロードジョブの一時停止

[PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md) ステートメントを実行してロードジョブを一時停止できます。ステートメント実行後、ロードジョブの状態は **PAUSED** になりますが、停止はしていません。[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) ステートメントを実行して再開することができます。また、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) ステートメントでその状態を確認することもできます。

次の例では、ロードジョブ `example_tbl2_ordertest2` を一時停止します。

```SQL
PAUSE ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## ロードジョブの再開

[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) ステートメントを実行して一時停止したロードジョブを再開できます。ロードジョブの状態は一時的に **NEED_SCHEDULE** になります（ロードジョブが再スケジュールされているため）、その後 **RUNNING** になります。その状態は [SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) ステートメントで確認できます。

次の例では、一時停止されたロードジョブ `example_tbl2_ordertest2` を再開します。

```SQL
RESUME ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## ロードジョブの変更

ロードジョブを変更する前に、[PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md) ステートメントでロードジョブを一時停止する必要があります。その後、[ALTER ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md) を実行できます。変更後、[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) ステートメントを実行して再開し、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) ステートメントでその状態を確認できます。

稼働中の BE ノードの数が `6` に増加し、消費される Kafka トピックのパーティションが `"0,1,2,3,4,5,6,7"` になったとします。実際のロードタスクの同時実行数を増やす場合、次のステートメントを実行して、希望するタスクの同時実行数 `desired_concurrent_number` を `6`（稼働中の BE ノードの数以上）に増やし、Kafka トピックのパーティションと初期オフセットを指定できます。

> **注記**
>
> 実際のタスクの同時実行数は複数のパラメーターの最小値によって決定されるため、FE の動的パラメーター `max_routine_load_task_concurrent_num` の値が `6` 以上であることを確認する必要があります。

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

## ロードジョブの停止

[STOP ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/STOP_ROUTINE_LOAD.md) ステートメントを実行することで、ロードジョブを停止できます。ステートメント実行後、ロードジョブの状態は **STOPPED** になり、停止したロードジョブを再開することはできません。また、停止したロードジョブの状態は [SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) ステートメントで確認することはできません。

以下の例では、ロードジョブ `example_tbl2_ordertest2` を停止します：

```SQL
STOP ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## FAQ

ルーチンロードのFAQは[こちら](../faq/loading/Routine_load_faq.md)をご覧ください。