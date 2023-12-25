---
displayed_sidebar: Chinese
---

# Apache Kafka®からの連続インポート

このドキュメントでは、Routine Loadの基本原理と、Routine Loadを使用してApache Kafka®のメッセージを連続的に消費し、StarRocksにインポートする方法について説明します。

StarRocksにメッセージストリームを連続的にインポートする場合は、メッセージストリームをKafkaのトピックに保存し、StarRocksにRoutine Loadインポートジョブを提出します。StarRocksはこのインポートジョブを常に実行し、Kafkaクラスタの指定されたトピックのすべてまたは一部のパーティションのメッセージを消費し、StarRocksにインポートします。

Routine LoadはExactly-Onceセマンティクスをサポートし、データの損失や重複を防ぎます。

Routine Loadは、インポートプロセス中にデータ変換を行ったり、UPSERTおよびDELETE操作を使用してデータの変更を実現することができます。詳細については、[インポートプロセスでのデータ変換](../loading/Etl_in_loading.md)および[インポートによるデータ変更の実現](../loading/Load_to_Primary_Key_tables.md)を参照してください。

> **注意**
>
> Routine Load操作には、ターゲットテーブルへのINSERT権限が必要です。ユーザーアカウントにINSERT権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)を参照してユーザーに権限を付与してください。

## サポートされるデータファイル形式

Routine Loadは現在、KakfaクラスタからCSV、JSON、Avro（v3.0.1以降）形式のデータを消費することができます。

> **注意**
>
> CSV形式のデータの場合、次の2つの点に注意する必要があります：
>
> - StarRocksは、最大50バイトのUTF-8エンコード文字列を列区切り文字として設定することができます。一般的なカンマ（,）、タブ、パイプ（|）などが含まれます。
> - 空の値（null）は`\N`で表されます。たとえば、データファイルには3つの列があり、ある行のデータの1列目と3列目がそれぞれ`a`と`b`であり、2列目にデータがない場合、2列目は空の値を表すために`\N`を使用する必要があります。`a,\N,b`と書きます。`a,,b`は2列目が空の文字列を表します。

## 基本原理

![routine load](../assets/4.5.2-1.png)

### 概念

- **インポートジョブ（Load job）**

  インポートジョブは常駐して実行され、インポートジョブの状態が**RUNNING**の場合、連続的に1つまたは複数の並行インポートタスクを生成し、Kafkaクラスタの1つのトピックのメッセージを消費し、StarRocksにインポートします。

- **インポートタスク（Load task）**

  インポートジョブは、一定のルールに従って複数のインポートタスクに分割されます。インポートタスクはインポートの基本単位であり、[Stream Load](../loading/StreamLoad.md)メカニズムを使用して独立したトランザクションとして実行されます。複数のインポートタスクは、異なるパーティションのメッセージを並行して消費し、StarRocksにインポートします。

### ジョブのフロー

1. **常駐するインポートジョブの作成**

    StarRocksにインポートジョブの作成SQLステートメントを提出する必要があります。FEはSQLステートメントを解析し、常駐するインポートジョブを作成します。

2. **インポートジョブを複数のインポートタスクに分割**

    FEは一定のルールに従ってインポートジョブを複数のインポートタスクに分割します。インポートタスクは独立したトランザクションとして実行されます。

    分割ルールは次のとおりです：
    - FEは、期待されるタスクの並行度`desired_concurrent_number`、Kafkaトピックのパーティション数、アクティブなBE数などに基づいて、実際のタスクの並行度を計算します。
    - FEは実際のタスクの並行度に基づいて、インポートジョブを複数のインポートタスクに分割し、タスク実行キューに配置します。

    各トピックには複数のパーティションがあり、パーティションとインポートタスクの対応関係は次のとおりです：
    - 各パーティションのメッセージは、対応するインポートタスクによってのみ消費されます。
    - 1つのインポートタスクは1つ以上のパーティションを消費する場合があります。
    - パーティションはインポートタスクに均等に割り当てられるようになっています。

3. **複数のインポートタスクを並行して実行し、Kafkaの複数のパーティションのメッセージを消費し、StarRocksにインポートする**

    1. **インポートタスクのスケジュールと提出**：FEは定期的にタスク実行キューのインポートタスクをスケジュールし、選択されたCoordinator BEに割り当てます。タスクのスケジュール間隔は`max_batch_interval`パラメータによって制御され、FEは可能な限りすべてのBEに均等にインポートタスクを割り当てます。`max_batch_interval`パラメータの詳細については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md#job_properties)を参照してください。

    2. **インポートタスクの実行**：Coordinator BEはインポートタスクを実行し、パーティションのメッセージを消費し、データを解析およびフィルタリングします。インポートタスクは十分な数のメッセージを消費するか、十分な時間を消費する必要があります。消費時間とデータ量は、FEの`max_routine_load_batch_size`、`routine_load_task_consume_second`設定項目によって制御されます。これらの設定項目の詳細については、[設定パラメータ](../administration/FE_configuration.md#インポートとエクスポート)を参照してください。その後、Coordinator BEはメッセージを関連するExecutor BEノードに配信し、Executor BEノードはメッセージをディスクに書き込みます。

      > **注意**
      >
      > StarRocksは、SASL_SSL認証、SASL認証、SSL認証、および非認証の方法を使用してKafkaに接続することができます。このドキュメントでは非認証の方法を使用してKafkaに接続する例を説明しています。安全認証メカニズムを使用してKafkaに接続する場合は、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

4. **新しいインポートタスクを継続的に生成し、データを連続的にインポートする**

    Executor BEノードがデータの書き込みに成功した後、Coordonator BEはFEにインポート結果を報告します。

    FEは報告結果に基づいて新しいインポートタスクを継続的に生成し、失敗したインポートタスクをリトライし、データを連続的にインポートし、データの損失や重複を防ぎます。

## 準備作業

Kafkaをダウンロードしてインストールします。トピックを作成し、データをアップロードします。操作手順については、[Apache Kafkaクイックスタート](https://kafka.apache.org/quickstart)を参照してください。

## インポートジョブの作成

ここでは、Routine Loadを使用してKafkaのCSV、JSON、Avro形式のデータを連続的に消費し、StarRocksにインポートする方法について、3つの簡単な例を紹介します。詳細な構文とパラメータの説明については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

### CSVデータのインポート

このセクションでは、Routine Loadを使用してKafkaクラスタのCSV形式のデータを連続的に消費し、StarRocksにインポートする方法について説明します。

#### データセット

Kafkaクラスタのトピックに次のCSV形式のデータが存在すると仮定します。CSVデータの各列の意味は、順番に注文番号、支払い日、顧客名、国籍、性別、支払い金額です。

```Plain text
2020050802,2020-05-08,Johann Georg Faust,Deutschland,male,895
2020050802,2020-05-08,Julien Sorel,France,male,893
2020050803,2020-05-08,Dorian Grey,UK,male,1262
2020050901,2020-05-09,Anna Karenina",Russia,female,175
2020051001,2020-05-10,Tess Durbeyfield,US,female,986
2020051101,2020-05-11,Edogawa Conan,japan,male,8924
```

#### ターゲットデータベースとテーブル

CSVデータのうち、StarRocksにインポートする必要のある5つの列（性別以外）に基づいて、StarRocksクラスタのターゲットデータベース`example_db`に`example_tbl1`テーブルを作成します。

```SQL
CREATE TABLE example_db.example_tbl1 ( 
    `order_id` bigint NOT NULL COMMENT "注文番号",
    `pay_dt` date NOT NULL COMMENT "支払い日", 
    `customer_name` varchar(26) NULL COMMENT "顧客名", 
    `nationality` varchar(26) NULL COMMENT "国籍", 
    `price`double NULL COMMENT "支払い金額"
) 
ENGINE=OLAP 
DUPLICATE KEY (order_id,pay_dt) 
DISTRIBUTED BY HASH(`order_id`); 
```

> **注意**
>
> StarRocksは、テーブルの作成およびパーティションの追加時に自動的にバケット数（BUCKETS）を設定するようになりました（バージョン2.5.7以降）。バケット数を手動で設定する必要はありません。詳細については、[データ分散の決定](../table_design/Data_distribution.md#データ分散の決定)を参照してください。

#### インポートジョブ

次のステートメントを使用して、StarRocksにRoutine Loadインポートジョブ`example_tbl1_ordertest1`を提出し、Kafkaクラスタの`ordertest1`トピックのメッセージを連続的に消費し、データベース`example_db`のテーブル`example_tbl1`にインポートします。また、インポートジョブは、指定されたトピックのパーティションの最初のオフセットから消費を開始します。

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

インポートジョブを提出した後、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)を実行して、インポートジョブの実行状況を確認できます。

- **インポートジョブの名前**

  1つのテーブルには複数のインポートジョブが存在する場合があります。インポートジョブの名前は、Kafkaトピックの名前やインポートジョブの作成時刻などの情報を使用して命名することをお勧めします。これにより、同じテーブル上の異なるインポートジョブを区別するのに役立ちます。

- **列の区切り文字**

  `COLUMN TERMINATED BY`は、CSVデータの列の区切り文字を指定します。デフォルトは`\t`です。

- **パーティションの消費と開始オフセット**

  パーティションや開始オフセットを指定する場合は、`kafka_partitions`、`kafka_offsets`パラメータを設定できます。たとえば、消費するパーティションが`"0,1,2,3,4"`であり、各パーティションに個別の開始オフセットを指定する場合は、次のように設定できます：

  ```SQL
    "kafka_partitions" ="0,1,2,3,4",
    "kafka_offsets" = "OFFSET_BEGINNING, OFFSET_END, 1000, 2000, 3000"
  ```

  `property.kafka_default_offsets` を使用して、すべてのパーティションのデフォルトの消費オフセットを設定することもできます。

  ```SQL
    "kafka_partitions" ="0,1,2,3,4",
    "property.kafka_default_offsets" = "OFFSET_BEGINNING"
  ```

  詳細なパラメーター説明については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md#data_sourcedata_source_properties)を参照してください。

- **データ変換**

  ソースデータとターゲットテーブルの間の列のマッピングと変換関係を指定する必要がある場合は、`COLUMNS` を設定する必要があります。`COLUMNS`内の列の**順序**は **CSVデータ**の列の順序と一致し、**名前**は**ターゲットテーブル**の列名に対応しています。この例では、CSVデータの5番目の列をターゲットテーブルにインポートする必要がないため、`COLUMNS`で5番目の列を `temp_gender` として一時的に名付けてプレースホルダーとして使用し、他の列は直接`example_tbl1`テーブルにマッピングされます。

  データ変換の詳細については、[データ変換の実装](./Etl_in_loading.md)を参照してください。

  > **注記**
  >
  > CSVデータの列の名前、順序、数量がターゲットテーブルの列と完全に一致する場合は、`COLUMNS`を設定する必要はありません。

- **実際のタスクの並列度を増やしてインポート速度を向上させる**

  パーティションの数が多く、BEノードの数が十分にある場合、インポート速度を上げたいと考えているなら、実際のタスクの並列度を増やして、インポートジョブをできるだけ多くの並列タスクに分割して実行することができます。

  実際のタスクの並列度は、以下の複数のパラメーターによって決まる式で決定されます。実際のタスクの並列度を増やすには、Routine Loadインポートジョブを作成する際に、個々のインポートジョブに対して高い希望タスク並列度 `desired_concurrent_number` を設定し、Routine Loadインポートジョブのデフォルトの最大タスク並列度 `max_routine_load_task_concurrent_num` を高く設定します。このパラメーターはFEの動的パラメーターであり、詳細な説明については [設定パラメーター](../administration/FE_configuration.md#导入和导出)を参照してください。この場合、実際のタスクの並列度の上限は、生存しているBEノードの数または指定されたパーティションの数になります。

  この例では、生存しているBEの数は5、指定されたパーティションの数も5で、`max_routine_load_task_concurrent_num` はデフォルト値 `5` です。実際のタスクの並列度を上限まで増やすには、`desired_concurrent_number` を `5` に設定する必要があります（デフォルト値は3）。これにより、実際のタスクの並列度は5になります。

  ```Plain
    min(aliveBeNum, partitionNum, desired_concurrent_number, max_routine_load_task_concurrent_num)
  ```
  
  インポート速度の調整についての詳細は、[Routine LoadのFAQ](../faq/loading/Routine_load_faq.md)を参照してください。

### JSONデータのインポート

#### データセット

KafkaクラスタのTopic `ordertest2` に以下のようなJSON形式のデータが存在するとします。1つのJSONオブジェクトのkeyはそれぞれ 商品ID、顧客名、顧客国籍、支払日、支払金額を意味します。

また、インポート時にデータ変換を行い、JSONデータの `pay_time` キーをDATE型に変換し、ターゲットテーブルの列 `pay_dt` にインポートしたいと考えています。

```JSON
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}
```

> **注意**
>
> ここでは、各行のJSONオブジェクトが1つのKafkaメッセージに含まれている必要があります。そうでない場合は、「JSON解析エラー」が発生する可能性があります。

#### ターゲットデータベースとテーブル

JSONデータに含まれる必要なkeyに基づいて、StarRocksクラスタのターゲットデータベース `example_db` にテーブル `example_tbl2` を作成します。

```SQL
CREATE TABLE `example_tbl2` ( 
    `commodity_id` varchar(26) NULL COMMENT "商品ID", 
    `customer_name` varchar(26) NULL COMMENT "顧客名", 
    `country` varchar(26) NULL COMMENT "顧客国籍", 
    `pay_time` bigint(20) NULL COMMENT "支払時間", 
    `pay_dt` date NULL COMMENT "支払日", 
    `price` double SUM NULL COMMENT "支払金額"
)
ENGINE=OLAP
AGGREGATE KEY(`commodity_id`,`customer_name`,`country`,`pay_time`,`pay_dt`) 
DISTRIBUTED BY HASH(`commodity_id`); 
```

> **注意**
>
> バージョン2.5.7以降、StarRocksはテーブル作成とパーティション追加時に自動的にバケット数（BUCKETS）を設定する機能をサポートしています。手動でバケット数を設定する必要はありません。詳細は、[バケット数の決定](../table_design/Data_distribution.md#确定分桶数量)を参照してください。

#### インポートジョブ

以下のステートメントを使用して、StarRocksにRoutine Loadインポートジョブ `example_tbl2_ordertest2` を提出し、KafkaクラスタのTopic `ordertest2` のメッセージを継続的に消費し、`example_tbl2` テーブルにインポートします。インポートジョブは、指定されたTopicの指定されたパーティションの最初のオフセットから消費を開始します。

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
    "kafka_broker_list" = "<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest2",
    "kafka_partitions" = "0,1,2,3,4",
    "property.kafka_default_offsets" = "OFFSET_BEGINNING"
);
```

インポートジョブを提出した後、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)を実行して、インポートジョブの実行状況を確認することができます。

- **データフォーマット**

  `PROPERTIES` 子句の `"format" = "json"` でデータフォーマットをJSONとして指定する必要があります。

- **データの抽出と変換**

  ソースデータとターゲットテーブルの間の列のマッピングと変換関係を指定するために、`COLUMNS` と `jsonpaths` パラメーターを設定することができます。`COLUMNS` の列名は**ターゲットテーブル**の列名に対応し、列の順序は**ソースデータ**の列の順序に対応します。`jsonpaths` パラメーターは、必要なフィールドデータをJSONデータから抽出するために使用され、新しく生成されたCSVデータのように機能します。その後、`COLUMNS` パラメーターは `jsonpaths` のフィールドに**順番に**一時的な名前を付けます。

  ソースデータの `pay_time` キーはDATE型に変換され、ターゲットテーブルの `pay_dt` 列にインポートする必要があるため、`COLUMNS` で `from_unixtime` 関数を使用して変換する必要があります。他のフィールドは直接 `example_tbl2` テーブルにマッピングできます。

  データ変換の詳細については、[データ変換の実装](./Etl_in_loading.md)を参照してください。

  > **注記**
  >
  > 各行のJSONオブジェクトのkeyの名前と数量が（順序は必要ありませんが）ターゲットテーブルの列に対応している場合は、`COLUMNS`を設定する必要はありません。

### Avroデータのインポート

バージョン3.0.1から、StarRocksはRoutine Loadを使用してAvroデータをインポートすることをサポートしています。

#### データセット

**Avroスキーマ**

1. 以下のようなAvroスキーマファイル `avro_schema.avsc` を作成します：

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

2. そのAvroスキーマを[Schema Registry](https://docs.confluent.io/cloud/current/get-started/schema-registry.html#create-a-schema)に登録します。

**Avroデータ**

Avroデータを構築し、Kafkaクラスタのトピック `topic_0` に送信します。

#### ターゲットデータベースとテーブル

Avroデータに含まれる必要なフィールドに基づいて、StarRocksクラスタのターゲットデータベース `sensor` にテーブル `sensor_log` を作成します。テーブルの列名はAvroデータのフィールド名と一致しています。両者のデータ型のマッピングについては、[データ型のマッピング](#データ型のマッピング)を参照してください。

```SQL
CREATE TABLE example_db.sensor_log ( 
    `id` bigint NOT NULL COMMENT "センサーID",
    `name` varchar(26) NOT NULL COMMENT "センサー名", 
    `checked` boolean NOT NULL COMMENT "チェック済み", 
    `data` double NULL COMMENT "センサーデータ", 
    `sensor_type` varchar(26) NOT NULL COMMENT "センサータイプ"
) 
ENGINE=OLAP 
DUPLICATE KEY (id) 
DISTRIBUTED BY HASH(`id`); 
```

> **注意**
>
> バージョン2.5.7以降、StarRocksはテーブル作成とパーティション追加時に自動的にバケット数（BUCKETS）を設定する機能をサポートしています。手動でバケット数を設定する必要はありません。詳細は、[バケット数の決定](../table_design/Data_distribution.md#确定分桶数量)を参照してください。

#### インポートジョブ

Kafkaクラスタのトピック `topic_0` のAvroメッセージを継続的に消費し、データベース `sensor` のテーブル `sensor_log` にインポートするインポートジョブ `sensor_log_load_job` を提出します。インポートジョブは、指定されたトピックの指定されたパーティションの最初のオフセットから消費を開始します。

```sql
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

  `PROPERTIES` 子句で `"format" = "avro"` を設定して、データフォーマットをAvroとして指定します。

- スキーマレジストリ

  `confluent.schema.registry.url` パラメータを使用して、Avro スキーマを登録する [Schema Registry](https://docs.confluent.io/platform/current/schema-registry/index.html) の URL を指定します。StarRocks はこの URL から Avro スキーマを取得します。形式は以下の通りです：

  ```Plaintext
  confluent.schema.registry.url = http[s]://[<schema-registry-api-key>:<schema-registry-api-secret>@]<hostname|ip address>[:<port>]
  ```

- データマッピングと変換

  ソースデータとターゲットテーブルの間のカラムのマッピングと変換関係を指定する必要がある場合は、`COLUMNS` と `jsonpaths` パラメータを設定できます。`COLUMNS` 内のカラム名はターゲットテーブルのカラム名に対応し、カラムの順序は `jsonpaths` 内のフィールドの順序に対応します。`jsonpaths` パラメータは、Avro データから必要なフィールドデータを抽出するために使用され、その後（新しく生成された CSV データのように）`COLUMNS` パラメータによって順番に一時的に名前が付けられます。

  データ変換の詳細については、[データ変換の実装](../loading/Etl_in_loading.md)を参照してください。

  > 説明
  >
  > Avro レコード内のフィールド名と数（順序は必要ありません）がターゲットテーブルのカラムに対応している場合、`COLUMNS` の設定は不要です。

インポートジョブを提出した後、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) を実行して、インポートジョブの実行状況を確認できます。

#### データ型のマッピング

StarRocks はすべてのタイプの Avro データのインポートをサポートしています。Avro データを StarRocks にインポートする際の型マッピングは以下の通りです：

**プリミティブ型**

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

**複合型**

| Avro           | StarRocks                                        |
| -------------- | ------------------------------------------------ |
| record         | RECORD 型のフィールドまたはそのサブフィールドを JSON としてインポート |
| enums          | STRING                                           |
| arrays         | ARRAY                                            |
| maps           | JSON                                             |
| union(T, null) | NULLABLE(T)                                      |
| fixed          | STRING                                           |

#### 使用制限

- StarRocks は現在、スキーマ進化をサポートしていません。スキーマレジストリサービスからは常に最新バージョンのスキーマ情報のみを取得します。

- 各 Kafka メッセージには単一の Avro データのみを含める必要があります。

## インポートジョブとタスクの確認

### インポートジョブの確認

[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) を実行して、`example_tbl2_ordertest2` という名前のインポートジョブの情報を確認します。例えば、インポートジョブの状態 `State`、インポートジョブの統計情報 `Statistic`（消費された総データ行数、インポートされた行数など）、消費されたパーティションと進捗 `Progress` などです。

インポートジョブの状態が自動的に **PAUSED** に変わった場合、インポートタスクのエラー行数がしきい値を超えた可能性があります。エラー行数のしきい値の設定方法については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md#job_properties) を参照してください。`ReasonOfStateChanged` や `ErrorLogUrls` のエラーを参照して問題を調査し、修正できます。修正後、[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) を使用して **PAUSED** 状態のインポートジョブを再開できます。

インポートジョブの状態が **CANCELLED** になった場合、インポートタスクの実行中に例外が発生した可能性があります（例：テーブルが削除された）。`ReasonOfStateChanged` や `ErrorLogUrls` のエラーを参照して問題を調査し、修正できます。しかし、修正後も **CANCELLED** 状態のインポートジョブを再開することはできません。

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
> StarRocks は現在実行中のインポートジョブのみを確認できます。停止されたり、開始されていないインポートジョブは確認できません。

### インポートタスクの確認

[SHOW ROUTINE LOAD TASK](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD_TASK.md) を実行して、インポートジョブ `example_tbl2_ordertest2` 内の一つまたは複数のインポートタスクの情報を確認します。例えば、現在いくつのタスクが実行中か、消費されたパーティションと進捗 `DataSourceProperties`、および対応する Coordinator BE ノード `BeId` などです。

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

## インポートジョブの一時停止

[PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md) を実行すると、インポートジョブが一時停止されます。インポートジョブは **PAUSED** 状態になりますが、インポートジョブは終了していません。[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) を実行してインポートジョブを再開できます。また、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) を実行して、一時停止されたインポートジョブの状態を確認できます。

例えば、以下のステートメントを使用して、インポートジョブ `example_tbl2_ordertest2` を一時停止できます：

```SQL
PAUSE ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## インポートジョブの再開

[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) を実行してインポートジョブを再開します。インポートジョブは一時的に **NEED_SCHEDULE** 状態になり、インポートジョブの再スケジュールが行われます。しばらくすると **RUNNING** 状態に戻り、Kafka メッセージの消費とデータのインポートを続けます。[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) を実行して、再開されたインポートジョブを確認できます。

例えば、以下のステートメントを使用して、インポートジョブ `example_tbl2_ordertest2` を再開できます：

```SQL
RESUME ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## インポートジョブの変更

変更する前に、[PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md) を実行してインポートジョブを一時停止する必要があります。その後、[ALTER ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md) を実行してインポートジョブのパラメータ設定を変更します。変更が成功した後、[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) を実行してインポートジョブを再開します。そして、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) を実行して、変更後のインポートジョブを確認します。

例えば、アクティブな BE ノードの数が 6 に増え、消費されるパーティションが `"0,1,2,3,4,5,6,7"` になった場合、実際のインポートの並行度を高めたい場合は、以下のステートメントを使用して、期待されるタスクの並行度 `desired_concurrent_number` を `6`（アクティブな BE ノードの数以上）に増やし、消費されるパーティションと開始消費オフセットを調整できます。

> **説明**
>
> 実際のインポートの並行度は、複数のパラメータの最小値によって決まるため、この時点で FE の動的パラメータ `max_routine_load_task_concurrent_num` の値が `6` 以上であることを確認する必要があります。

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

## インポートジョブの停止

[STOP ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/STOP_ROUTINE_LOAD.md)を実行すると、インポートジョブを停止できます。インポートジョブは**STOPPED**状態になり、このインポートジョブは終了したことを意味し、復旧することはできません。SHOW ROUTINE LOADステートメントを再実行しても、既に停止したインポートジョブは表示されません。

例えば、以下のステートメントで`example_tbl2_ordertest2`のインポートジョブを停止できます：

```SQL
STOP ROUTINE LOAD FOR example_tbl2_ordertest2;
```

## よくある質問

[Routine Loadのよくある質問](../faq/loading/Routine_load_faq.md)を参照してください。