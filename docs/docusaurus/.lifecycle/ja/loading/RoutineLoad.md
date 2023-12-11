---
displayed_sidebar: "English"
---

# Apache Kafka®からデータを連続的にロードする

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

このトピックでは、Routine Loadジョブを作成してKafkaメッセージ（イベント）をStarRocksにストリームし、Routine Loadに関する基本的な概念を紹介します。

ストリームのメッセージをStarRocksに連続的にロードするには、Kafkaのトピックにメッセージストリームを保存し、Routine Loadジョブを作成してメッセージを消費します。Routine LoadジョブはStarRocksに永続化され、トピックのすべてまたは一部のパーティションに含まれるメッセージを消費するための一連のロードタスクを生成し、メッセージをStarRocksにロードします。

Routine Loadジョブはデータをロードする際に、一度限りのデリバリーセマンティクスをサポートし、StarRocksにロードされるデータが失われたり重複したりしないように保証します。

Routine Loadは、データのロード時にデータ変換およびUPSERT、DELETE操作によるデータ変更をサポートしています。詳細については、[Transform data at loading](../loading/Etl_in_loading.md)および[Change data through loading](../loading/Load_to_Primary_Key_tables.md)を参照してください。

<InsertPrivNote />

## サポートされるデータ形式

Routine Loadは現在、KafkaクラスターからCSV、JSON、およびAvro（v3.0.1以降のサポート）形式のデータを消費することをサポートしています。

> **注記**
>
> CSVデータについては、次の点に注意してください：
>
> - テキストデリミターとして、UTF-8文字列を使用できます。 たとえば、カンマ（,）、タブ、またはパイプ（|）で、それぞれの長さが50バイトを超えないものです。
> - NULL値は`\N`を使用して示します。 たとえば、データファイルには3つの列があり、そのデータファイルのレコードには1番目と3番目の列にデータが含まれている場合がありますが、2番目の列にデータが含まれていないこともあります。 この場合、2番目の列にNULL値を示すために`\N`を使用する必要があります。 これは、レコードを`a,\N,b`のようにコンパイルする必要があります。`a,,b`ではなく、`a,,b`は、レコードの2番目の列に空の文字列が含まれていることを示します。

## 基本的な概念

![routine load](../assets/4.5.2-1.png)

### 用語

- **ロードジョブ**

   Routine Loadジョブは長時間実行されるジョブです。そのステータスがRUNNINGである限り、ロードジョブはトピックのKafkaクラスター内のメッセージを消費し、データをStarRocksに連続的にロードするために1つまたは複数の並行ロードタスクを生成します。

- **ロードタスク**

  ロードジョブは特定のルールに基づいて複数のロードタスクに分割されます。ロードタスクはデータロードの基本単位です。個々のイベントとして、ロードタスクは[Stream Load](../loading/StreamLoad.md)に基づいたロードメカニズムを実装します。複数のロードタスクが異なるパーティションからメッセージを同時に消費し、データをStarRocksにロードします。

### ワークフロー

1. **Routine Loadジョブを作成します。**
   Kafkaからデータをロードするには、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)ステートメントを実行してRoutine Loadジョブを作成する必要があります。FEはステートメントを解析し、指定したプロパティに従ってジョブを作成します。

2. **FEはジョブを複数のロードタスクに分割します。**

    FEは特定のルールに基づいてジョブを複数のロードタスクに分割します。各ロードタスクは個々のトランザクションです。
    分割のルールは次のとおりです：
    - FEは、望ましい並行数`desired_concurrent_number`、Kafkaトピックのパーティション数、および稼働しているBEノードの数に従って、実際の並行数を計算します。
    - FEは実際の並行数に基づいてジョブをロードタスクに分割し、タスクをタスクキューに配置します。

    各Kafkaトピックは複数のパーティションで構成されています。トピックパーティションとロードタスクの関係は次のとおりです：
    - パーティションはロードタスクに固有に割り当てられており、パーティションからのすべてのメッセージはロードタスクによって消費されます。
    - ロードタスクは1つまたは複数のパーティションからメッセージを消費できます。
    - すべてのパーティションは均等にロードタスクに分散されます。

3. **複数のロードタスクが同時に実行され、複数のKafkaトピックパーティションからメッセージを消費し、StarRocksにデータをロードします**

   1. **FEはロードタスクをスケジュールして送信します**: FEはロードタスクを定期的にタスクキューにスケジュールし、選択されたCoordinator BEノードに割り当てます。ロードタスク間の間隔は、設定項目`max_batch_interval`によって定義されます。FEはロードタスクをすべてのBEノードに均等に分散します。詳細については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md#example)を参照してください。

   2. Coordinator BEはロードタスクを開始し、パーティションのメッセージを消費し、データを解析およびフィルタリングします。ロードタスクは、定義済みのメッセージ量が消費されるか、定義済みの時間制限に達するまで続きます。メッセージのバッチサイズと時間制限は、FEの設定項目`max_routine_load_batch_size`および`routine_load_task_consume_second`で定義されています。詳細については、[Configuration](../administration/Configuration.md)を参照してください。その後、Coordinator BEはメッセージをExecutor BEに配布します。Executor BEはメッセージをディスクに書き込みます。

         > **注記**
         >
         > StarRocksは、SASL_SSL、SASLまたはSSLのセキュリティ認証メカニズムを介してKafkaへのアクセスをサポートしており、認証なしで接続する場合を例としています。 セキュリティ認証メカニズムを介してKafkaに接続する必要がある場合は、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

4. **FEはデータを連続的にロードするための新しいロードタスクを生成します。**
   Executor BEがデータをディスクに書き込んだ後、Coordinator BEはロードタスクの結果をFEに報告します。その結果に基づいて、FEはデータを連続的にロードするための新しいロードタスクを生成するか、失敗したタスクを再試行して、StarRocksにロードされたデータが失われたり重複したりしないようにします。

## Routine Loadジョブの作成

次の3つの例では、CSV形式、JSON形式、およびAvro形式のデータをKafkaから消費し、Routine Loadジョブを作成してStarRocksにデータをロードする方法について説明します。詳細な構文とパラメータの説明については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

### CSV形式データのロード

このセクションでは、Kafkaクラスター内のCSV形式データを消費し、StarRocksにデータをロードするためのRoutine Loadジョブを作成する方法について説明します。

#### データセットの準備

Kafkaクラスターのトピック`ordertest1`には、CSV形式のデータセットがあるとします。データセットの各メッセージには、注文ID、支払い日、顧客名、国籍、性別、および価格の6つのフィールドが含まれます。

```Plain
2020050802,2020-05-08,Johann Georg Faust,Deutschland,male,895
2020050802,2020-05-08,Julien Sorel,France,male,893
2020050803,2020-05-08,Dorian Grey,UK,male,1262
2020050901,2020-05-09,Anna Karenina",Russia,female,175
2020051001,2020-05-10,Tess Durbeyfield,US,female,986
2020051101,2020-05-11,Edogawa Conan,japan,male,8924
```

#### テーブルの作成

CSV形式データのフィールドに基づいて、データベース`example_db`内のテーブル`example_tbl1`を作成します。次の例では、CSV形式データの顧客性別を除く5つのフィールドを持つテーブルを作成します。

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
> v2.5.7以降、StarRocksは表の作成またはパーティションの追加時に、バケツの数（BUCKETS）を自動的に設定できるようになりました。バケツ数を手動で設定する必要はもはやありません。詳細については、[バケットの数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

#### Routine Loadジョブを送信する

次のステートメントを実行して、トピック`ordertest1`のメッセージを消費し、データをテーブル`example_tbl1`にロードするRoutine Loadジョブ（`example_tbl1_ordertest1`）を送信します。ロードタスクは、指定されたパーティションの初期オフセットからメッセージを消費します。

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
```json
    "kafka_partitions" ="0,1,2,3,4",
    "property.kafka_default_offsets" = "OFFSET_BEGINNING"
);
```

ロードジョブを提出した後、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) ステートメントを実行して、ロードジョブのステータスを確認できます。

- **ロードジョブ名**

  テーブルには複数のロードジョブが存在する可能性があります。そのため、対応するKafkaトピックとロードジョブの提出時刻を含む名前を付けることをお勧めします。これにより、各テーブルのロードジョブを区別しやすくなります。

- **列のセパレータ**

  `COLUMN TERMINATED BY` プロパティは、CSV形式のデータの列セパレータを定義します。デフォルトは `\t` です。

- **Kafkaトピックのパーティションとオフセット**

  `kafka_partitions` プロパティおよび `kafka_offsets` プロパティを指定して、メッセージを消費するパーティションとオフセットを指定できます。例えば、ロードジョブがトピック `ordertest1` の Kafka パーティション `"0,1,2,3,4"` からメッセージを初期オフセットで消費するようにしたい場合、次のようにプロパティを指定できます：ロードジョブが Kafka パーティション `"0,1,2,3,4"` からメッセージを消費し、各パーティションに別々の開始オフセットを指定する場合、次のように構成できます：

    ```SQL
    "kafka_partitions" ="0,1,2,3,4",
    "kafka_offsets" = "OFFSET_BEGINNING, OFFSET_END, 1000, 2000, 3000"
    ```

  デフォルトのオフセットを全てのパーティションに設定することもできます。その場合は、プロパティ `property.kafka_default_offsets` を使用します。

    ```SQL
    "kafka_partitions" ="0,1,2,3,4",
    "property.kafka_default_offsets" = "OFFSET_BEGINNING"
    ```

  詳細な情報については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を参照してください。

- **データのマッピングと変換**

  CSV形式のデータとStarRocksテーブルの間のマッピングおよび変換の関係を指定するには、`COLUMNS` パラメータを使用する必要があります。

  **データのマッピング:**

  - StarRocksは、CSV形式のデータの列を**順序通り**に抽出し、`COLUMNS` パラメータで宣言されたフィールドにマッピングします。

  - StarRocksは、`COLUMNS` パラメータで宣言されたフィールドを名前によってStarRocksテーブルの列にマッピングします。

  **データ変換:**

  例では、CSV形式のデータから顧客の性別の列を除外しているため、`COLUMNS` パラメータの `temp_gender` フィールドはこのフィールドのプレースホルダとして使用されます。その他のフィールドは、直接StarRocksテーブル `example_tbl1` の列にマッピングされます。

  データ変換の詳細については、[Transform data at loading](./Etl_in_loading.md) を参照してください。

    > **注意**
    >
    > CSV形式のデータの列の名前、数、および順序が完全にStarRocksテーブルと対応している場合は、`COLUMNS` パラメータを指定する必要はありません。

- **タスクの同時実行数**

  多くのKafkaトピックパーティションや十分なBEノードがある場合、タスクの同時実行数を増やすことでロードを加速できます。

  実際のロードタスクの同時実行数を増やすには、ルーチンロードジョブを作成する際に `desired_concurrent_number` を増やします。また、FEの動的構成項目である `max_routine_load_task_concurrent_number` (デフォルトの最大ロードタスク同時数) をより大きな値に設定することもできます。`max_routine_load_task_concurrent_number` の詳細については、[FE configuration items](../administration/Configuration.md#fe-configuration-items) を参照してください。

  実際のタスクの同時実行数は、生きているBEノードの数、事前に指定されたKafkaトピックパーティションの数、および`desired_concurrent_number`と`max_routine_load_task_concurrent_number`の値のうち最小の値によって定義されます。

  この例では、生きているBEノードの数は `5`、事前に指定されたKafkaトピックパーティションの数は `5`、`max_routine_load_task_concurrent_number`の値は `5` です。ロードタスクの実際の同時実行数を増やすには、デフォルト値の `3` から `desired_concurrent_number` を `5` に増やします。

  プロパティの詳細については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を、ロードを加速する詳細な手順については、[Routine Load FAQ](../faq/loading/Routine_load_faq.md) を参照してください。

### JSON形式のデータをロード

このセクションでは、Kafkaクラスタ内のJSON形式のデータを消費し、StarRocksにデータをロードするためのルーチンロードジョブを作成する方法について説明します。

#### データセットの準備

Kafkaクラスタ内のトピック `ordertest2` にJSON形式のデータセットがあると仮定します。このデータセットには、商品ID、顧客名、国籍、支払い時刻、価格の6つのキーが含まれています。さらに、支払い時刻の列をDATE型に変換し、StarRocksテーブルの`pay_dt`列にロードしたいとします。

```JSON
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}
```

> **注意** 1つのJSONオブジェクトは1つのKafkaメッセージ内にある必要があります。それ以外の場合、JSONの解析エラーが返されます。

#### テーブルの作成

JSON形式のデータのキーに基づいて、データベース `example_db` 内にテーブル `example_tbl2` を作成します。

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
> v2.5.7以降、StarRocksは自動的にバケットの数（BUCKETS）を設定できます。テーブルを作成するか、パーティションを追加する際にバケットの数を手動で設定する必要はありません。詳細については、[バケットの数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets) を参照してください。

#### ルーチンロードジョブの提出

次のステートメントを実行して、`ordertest2` トピック内のメッセージを消費し、データをテーブル `example_tbl2` にロードするためのルーチンロードジョブ `example_tbl2_ordertest2` を提出します。ロードタスクは、指定されたトピックのパーティションの初期オフセットからメッセージを消費します。

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

ロードジョブを提出した後、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) ステートメントを実行して、ロードジョブのステータスを確認できます。

- **データのフォーマット**

  データのフォーマットがJSONであることを定義するために、`PROPERTIES` 句で `"format" = "json"` を指定する必要があります。

- **データのマッピングと変換**

  JSON形式のデータとStarRocksテーブルの間のマッピングおよび変換の関係を指定するには、`COLUMNS` パラメータと `jsonpaths` プロパティを指定する必要があります。`COLUMNS` パラメータで指定されたフィールドの順序はJSON形式のデータと一致し、フィールドの名前はStarRocksテーブルと一致している必要があります。`jsonpaths` プロパティは、必要なフィールドをJSONデータから抽出するために使用されます。これらのフィールドは、`COLUMNS` プロパティによって名前が付けられます。

  この例では、支払い時刻のフィールドをDATEデータ型に変換し、StarRocksテーブルの `pay_dt` 列にデータをロードする必要があるため、from_unixtime関数を使用する必要があります。その他のフィールドは、テーブル `example_tbl2` のフィールドにマッピングされます。

**データマッピング：**

- StarRocksはJSONフォーマットのデータから`name`と`code`のキーを抽出し、`jsonpaths`プロパティで宣言されたキーにマッピングします。

- StarRocksは`jsonpaths`プロパティで宣言されたキーを抽出し、それらを`COLUMNS`パラメータで宣言されたフィールドに**シーケンスで**マッピングします。

- StarRocksは`COLUMNS`パラメータで宣言されたフィールドを抽出し、それらをStarRocksテーブルの列に**名前で**マッピングします。

**データ変換：**

- この例では、キー`pay_time`をDATEデータ型に変換し、StarRocksテーブルの`pay_dt`列にデータを読み込む必要があります。そのために、`COLUMNS`パラメータでfrom_unixtime関数を使用する必要があります。その他のフィールドは、`example_tbl2`テーブルのフィールドに直接マップされます。

- また、この例では、JSON形式のデータから顧客の性別の列を除外しています。そのため、`COLUMNS`パラメータの`temp_gender`はこのフィールドのプレースホルダとして使用されます。その他のフィールドは、`example_tbl1`テーブルの列に直接マッピングされます。

データ変換の詳細については、[ロード時のデータ変換](./Etl_in_loading.md)を参照してください。

> **注意**
>
> JSONオブジェクトのキーの名前と数がStarRocksテーブルのフィールドと完全に一致する場合、`COLUMNS`パラメータを指定する必要はありません。

### Avroフォーマットデータのロード

v3.0.1以降、StarRocksはRoutine Loadを使用してAvroデータをロードすることがサポートされています。 

#### データセットの準備

##### Avroスキーマ

1. 以下のAvroスキーマファイル`avro_schema.avsc`を作成してください：

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

2. Avroスキーマを[Schema Registry](https://docs.confluent.io/cloud/current/get-started/schema-registry.html#create-a-schema)に登録してください。 

##### Avroデータ

Avroデータを準備し、それをKafkaトピック`topic_0`に送信してください。

#### テーブルの作成

Avroデータのフィールドに基づいて、StarRocksクラスタ内のターゲットデータベース`example_db`にテーブル`sensor_log`を作成してください。テーブルの列名は、Avroデータのフィールド名と一致する必要があります。テーブル列とAvroデータフィールドのデータ型のマッピングについては、[データ型のマッピング](#データ型のマッピング)を参照してください。

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

> **お知らせ**
>
> v2.5.7以降、テーブルを作成するかパーティションを追加する際に、バケツの数（BUCKETS）をStarRocksは自動的に設定できます。バケツの数を手動で設定する必要はもはやありません。詳細については、[バケツの数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

#### Routine Loadジョブの提出

以下の文を実行して、Routine Loadジョブ `sensor_log_load_job`を作成してください。これにより、Kafkaトピック`topic_0`からAvroメッセージを消費し、データをデータベース`sensor`内のテーブル `sensor_log`に読み込みます。ロードジョブはトピックの指定されたパーティションから、最初のオフセットからメッセージを消費します。

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

  データフォーマットがAvroであることを定義するために、`PROPERTIES`句で`"format = "avro"`を指定する必要があります。

- スキーマレジストリ

  Avroスキーマが登録されているSchema RegistryのURLを指定するために、`confluent.schema.registry.url`を構成する必要があります。StarRocksは、このURLを使用してAvroスキーマを取得します。フォーマットは以下の通りです：

  ```Plaintext
  confluent.schema.registry.url = http[s]://[<schema-registry-api-key>:<schema-registry-api-secret>@]<hostname|ip address>[:<port>]
  ```

- データマッピングおよびデータ変換

  AvroフォーマットのデータとStarRocksテーブルの間のマッピングと変換の関係を指定するために、`COLUMNS`パラメータと`jsonpaths`プロパティを指定する必要があります。`COLUMNS`パラメータで指定されたフィールドの順序は、`jsonpaths`プロパティのフィールドと一致している必要があり、フィールドの名前はStarRocksテーブルのものと一致している必要があります。`jsonpaths`プロパティは、Avroデータから必要なフィールドを抽出します。これらのフィールドは、`COLUMNS`プロパティによって命名されます。

  データ変換の詳細については、[ロード時のデータ変換](https://docs.starrocks.io/en-us/latest/loading/Etl_in_loading)を参照してください。

  > 注意
  >
  > Avroレコードのフィールドの名前と数がStarRocksテーブルの列と完全に一致する場合、`COLUMNS`パラメータを指定する必要はありません。

ロードジョブを提出した後、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)文を実行して、ロードジョブのステータスを確認できます。

#### データ型のマッピング

ロードするAvroデータフィールドとStarRocksテーブル列間のデータ型マッピングは、以下の通りです：

##### プリミティブタイプ

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

##### 複合タイプ

| Avro           | StarRocks                                                    |
| -------------- | ------------------------------------------------------------ |
| record         | レコード全体またはそのサブフィールドをJSONとしてStarRocksに読み込みます。 |
| enums          | STRING                                                       |
| arrays         | ARRAY                                                        |
| maps           | JSON                                                         |
| union(T, null) | NULLABLE(T)                                                  |
| fixed          | STRING                                                       |

#### 制限

- 現在、StarRocksはスキーマ進化をサポートしていません。
- 各Kafkaメッセージは1つのAvroデータレコードのみを含む必要があります。

## ロードジョブおよびタスクの確認

### ロードジョブの確認

[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)文を実行して、ロードジョブ`example_tbl2_ordertest2`の実行状態`State`、統計情報(消費された総行数とロードされた総行数)`Statistics`、およびロードジョブの進行状況`progress`を確認できます。

ロードジョブの状態が自動的に**PAUSED**に変更される場合、エラーレコードの数が閾値を超えた可能性があります。この閾値の設定方法の詳細については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。問題を修正した後、[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md)文を実行して、**PAUSED**状態のロードジョブを再開できます。

ロードジョブの状態が**CANCELLED**の場合、ロードジョブで例外が発生した可能性があります（たとえば、テーブルが削除された場合）。問題を特定してトラブルシューティングするために、ファイル`ReasonOfStateChanged`および`ErrorLogUrls`を確認できます。しかし、**CANCELLED**状態のロードジョブを再開することはできません。

```SQL
MySQL [example_db]> SHOW ROUTINE LOAD FOR example_tbl2_ordertest2 \G
*************************** 1. row ***************************
                  Id: 63013
                Name: example_tbl2_ordertest2
          CreateTime: 2022-08-10 17:09:00
           PauseTime: NULL
```
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

> **注意事項**
>
> 停止中またはまだ開始していないロード ジョブをチェックすることはできません。

### ロード タスクのチェック

[SHOW ROUTINE LOAD TASK](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD_TASK.md) ステートメントを実行して、ロード ジョブ `example_tbl2_ordertest2` のロード タスクを確認します。実行中のタスクの数、消費される Kafka トピック パーティションと消費進行状況 `DataSourceProperties`、および対応するコーディネータ BE ノード `BeId` などがわかります。

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

## ロード ジョブの一時停止

[PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md) ステートメントを実行して、ロード ジョブを一時停止できます。ステートメントの実行後、ロード ジョブの状態は**PAUSED**になります。ただし、停止していません。[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) ステートメントを実行して再開できます。[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) ステートメントで状態を確認できます。

次の例では、ロード ジョブ `example_tbl2_ordertest2` を一時停止します。

```SQL
PAUSE ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## ロード ジョブの再開

[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) ステートメントを実行して、一時停止中のロード ジョブを再開できます。ロード ジョブの状態は一時的に**NEED_SCHEDULE**になります（ロード ジョブが再スケジュールされているため）、その後**RUNNING**になります。[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) ステートメントで状態を確認できます。

次の例では、一時停止中のロード ジョブ `example_tbl2_ordertest2` を再開します。

```SQL
RESUME ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## ロード ジョブの変更

ロード ジョブを変更する前に、[PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md) ステートメントで一時停止する必要があります。その後、[ALTER ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md) を実行できます。変更が完了した後、[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) ステートメントを実行して再開し、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) ステートメントで状態を確認できます。

BE ノードの数が`6`に増え、消費される Kafka トピック パーティションが`"0,1,2,3,4,5,6,7"`に増えたとします。実際のロード タスクの並行性を増やしたい場合、次のステートメントを実行して、望ましいタスクの並行性 `desired_concurrent_number` の数を`6`（BE ノードの数以上）に増やし、Kafka トピック パーティションと初期オフセットを指定できます。

> **注意**
>
> 実際のタスクの並行性は、複数のパラメータの最小値で決まるため、FE 動的パラメータ `max_routine_load_task_concurrent_num` の値が`6`以上であることを確認してください。

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

## ロード ジョブの停止

[STOP ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/STOP_ROUTINE_LOAD.md) ステートメントを実行して、ロード ジョブを停止できます。ステートメントの実行後、ロード ジョブの状態は**STOPPED**になり、停止したロード ジョブを再開できません。[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) ステートメントで停止したロード ジョブの状態を確認できません。

次の例では、ロード ジョブ `example_tbl2_ordertest2` を停止します。

```SQL
STOP ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## よくある質問

[Routine Load FAQ](../faq/loading/Routine_load_faq.md) を参照してください。