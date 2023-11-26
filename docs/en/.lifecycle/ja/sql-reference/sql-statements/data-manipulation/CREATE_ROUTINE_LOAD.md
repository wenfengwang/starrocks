# CREATE ROUTINE LOAD

## 説明

Routine Loadは、Apache Kafka®からメッセージを連続的に消費し、データをStarRocksにロードすることができます。Routine Loadは、KafkaクラスタからCSV、JSON、およびAvro（v3.0.1以降でサポート）形式のデータを消費し、`plaintext`、`ssl`、`sasl_plaintext`、および`sasl_ssl`を含む複数のセキュリティプロトコルを介してKafkaにアクセスすることができます。

このトピックでは、CREATE ROUTINE LOADステートメントの構文、パラメータ、および例について説明します。

> **注意**
>
> - Routine Loadのアプリケーションシナリオ、原則、および基本的な操作については、[Apache Kafka®からデータを連続的にロードする](../../../loading/RoutineLoad.md)を参照してください。
> - StarRocksテーブルにデータをロードするには、StarRocksテーブルにINSERT権限を持つユーザーとしてのみデータをロードできます。INSERT権限がない場合は、[GRANT](../account-management/GRANT.md)の手順に従って、StarRocksクラスタに接続するために使用するユーザーにINSERT権限を付与してください。

## 構文

```SQL
CREATE ROUTINE LOAD <database_name>.<job_name> ON <table_name>
[load_properties]
[job_properties]
FROM data_source
[data_source_properties]
```

## パラメータ

### `database_name`、`job_name`、`table_name`

`database_name`

オプションです。StarRocksデータベースの名前です。

`job_name`

必須です。Routine Loadジョブの名前です。テーブルは複数のRoutine Loadジョブからデータを受け取ることができます。複数のRoutine Loadジョブを区別するために、識別可能な情報（たとえば、Kafkaトピック名とおおよそのジョブ作成時間）を使用して、意味のあるRoutine Loadジョブ名を設定することをお勧めします。Routine Loadジョブの名前は、同じデータベース内で一意である必要があります。

`table_name`

必須です。データがロードされるStarRocksテーブルの名前です。

### `load_properties`

オプションです。データのプロパティです。構文は次のとおりです。

```SQL
[COLUMNS TERMINATED BY '<column_separator>'],
[ROWS TERMINATED BY '<row_separator>'],
[COLUMNS (<column1_name>[, <column2_name>, <column_assignment>, ... ])],
[WHERE <expr>],
[PARTITION (<partition1_name>[, <partition2_name>, ...])]
[TEMPORARY PARTITION (<temporary_partition1_name>[, <temporary_partition2_name>, ...])]
```

`COLUMNS TERMINATED BY`

CSV形式のデータの列区切り記号です。デフォルトの列区切り記号は`\t`（タブ）です。たとえば、列区切り記号をカンマとして指定するには、`COLUMNS TERMINATED BY ","`と指定します。

> **注意**
>
> - ここで指定する列区切り記号が、ロードするデータの列区切り記号と同じであることを確認してください。
> - テキスト区切り記号として、カンマ（`,`）、タブ、またはパイプ（`|`）などのUTF-8文字列を使用できます。長さが50バイトを超えないようにしてください。
> - ヌル値は`\N`を使用して示します。たとえば、データレコードが3つの列から構成され、データレコードが最初の列と3番目の列にデータを保持しているが、2番目の列にデータを保持していない場合、2番目の列には`\N`を使用してヌル値を示す必要があります。これは、レコードを`a,\N,b`ではなく`a,,b`とコンパイルする必要があることを意味します。`a,,b`は、レコードの2番目の列に空の文字列が保持されていることを示します。

`ROWS TERMINATED BY`

CSV形式のデータの行区切り記号です。デフォルトの行区切り記号は`\n`です。

`COLUMNS`

ソースデータの列とStarRocksテーブルの列のマッピングです。詳細については、このトピックの[列マッピング](#column-mapping)を参照してください。

- `column_name`：ソースデータの列が計算なしでStarRocksテーブルの列にマッピングできる場合、列名のみを指定する必要があります。これらの列はマッピングされた列と呼ばれます。
- `column_assignment`：ソースデータの列が直接StarRocksテーブルの列にマッピングできず、データのロード前に関数を使用して列の値を計算する必要がある場合、`expr`で計算関数を指定する必要があります。これらの列は派生列と呼ばれます。
  StarRocksはまずマッピングされた列を解析しますので、派生列はマッピングされた列の後に配置することをお勧めします。

`WHERE`

フィルタ条件です。フィルタ条件を満たすデータのみがStarRocksにロードされます。たとえば、`col1`の値が`100`より大きく、`col2`の値が`1000`と等しい行のみをインジェストしたい場合は、`WHERE col1 > 100 and col2 = 1000`を使用できます。

> **注意**
>
> フィルタ条件で指定された列は、ソース列または派生列のいずれかです。

`PARTITION`

StarRocksテーブルがパーティションp0、p1、p2、およびp3に分散しており、StarRocksにロードされるデータをp1、p2、およびp3のみにロードし、p0に格納されるデータをフィルタリングする場合、`PARTITION(p1, p2, p3)`とフィルタ条件として指定できます。デフォルトでは、このパラメータを指定しない場合、データはすべてのパーティションにロードされます。例：

```SQL
PARTITION (p1, p2, p3)
```

`TEMPORARY PARTITION`

データをロードする[一時パーティション](../../../table_design/Temporary_partition.md)の名前です。カンマ（,）で区切られた複数の一時パーティションを指定できます。

### `job_properties`

必須です。ロードジョブのプロパティです。構文は次のとおりです。

```SQL
PROPERTIES ("<key1>" = "<value1>"[, "<key2>" = "<value2>" ...])
```

| **プロパティ**              | **必須** | **説明**                                                     |
| ------------------------- | -------- | ------------------------------------------------------------ |
| desired_concurrent_number | いいえ   | 単一のRoutine Loadジョブの期待されるタスク並列性。デフォルト値：`3`。実際のタスク並列性は、複数のパラメータの最小値によって決まります：`min(alive_be_number, partition_number, desired_concurrent_number, max_routine_load_task_concurrent_num)`。<ul><li>`alive_be_number`：Alive BEノードの数。</li><li>`partition_number`：消費するパーティションの数。</li><li>`desired_concurrent_number`：単一のRoutine Loadジョブの期待されるタスク並列性。デフォルト値：`3`。</li><li>`max_routine_load_task_concurrent_num`：Routine Loadジョブのデフォルトの最大タスク並列性で、`5`です。[FE動的パラメータ](../../../administration/Configuration.md#configure-fe-dynamic-parameters)を参照してください。</li></ul>最大の実際のタスク並列性は、Alive BEノードの数または消費するパーティションの数によって決まります。|
| max_batch_interval        | いいえ   | タスクのスケジュール間隔、つまりタスクが実行される頻度です。単位：秒。値の範囲：`5`〜`60`。デフォルト値：`10`。`10`より短いスケジュールの場合、過剰なロード頻度により、過剰なタブレットバージョンが生成されますので、`10`より大きな値を設定することをお勧めします。 |
| max_batch_rows            | いいえ   | このプロパティは、エラー検出のウィンドウを定義するためにのみ使用されます。ウィンドウは、単一のRoutine Loadタスクによって消費されるデータの行数です。値は`10 * max_batch_rows`です。デフォルト値は`10 * 200000 = 200000`です。Routine Loadタスクは、エラー検出ウィンドウ内のエラーデータを検出します。エラーデータとは、無効なJSON形式のデータなど、StarRocksが解析できないデータのことです。 |
| max_error_number          | いいえ   | エラー検出ウィンドウ内で許可されるエラーデータ行の最大数です。エラーデータ行の数がこの値を超えると、ロードジョブは一時停止します。[SHOW ROUTINE LOAD](./SHOW_ROUTINE_LOAD.md)を実行し、`ErrorLogUrls`を使用してエラーログを表示できます。その後、エラーログに従ってKafkaのエラーを修正できます。デフォルト値は`0`で、エラーデータ行は許可されません。<br />**注意** <br />エラーデータ行には、WHERE句でフィルタリングされたデータ行は含まれません。 |
| strict_mode               | いいえ   | [厳密モード](../../../loading/load_concept/strict_mode.md)を有効にするかどうかを指定します。有効な値：`true`および`false`。デフォルト値：`false`。厳密モードが有効になっている場合、ロードデータの列の値が`NULL`であり、ターゲットテーブルがこの列に`NULL`値を許可していない場合、データ行はフィルタリングされます。 |
| log_rejected_record_num | いいえ | ログに記録できる不適格なデータ行の最大数を指定します。このパラメータはv3.1以降でサポートされています。有効な値：`0`、`-1`、およびゼロ以外の正の整数。デフォルト値：`0`。<ul><li>値`0`は、フィルタリングされたデータ行はログに記録されません。</li><li>値`-1`は、フィルタリングされたすべてのデータ行がログに記録されます。</li><li>`n`などのゼロ以外の正の整数は、各BEでフィルタリングされた最大`n`個のデータ行をログに記録できます。</li></ul> |
| timezone                  | いいえ   | ロードジョブで使用するタイムゾーンです。デフォルト値：`Asia/Shanghai`。このパラメータの値は、strftime()、alignment_timestamp()、およびfrom_unixtime()などの関数によって返される結果に影響を与えます。このパラメータで指定されたタイムゾーンは、セッションレベルのタイムゾーンです。詳細については、[タイムゾーンの設定](../../../administration/timezone.md)を参照してください。 |
| merge_condition           | いいえ   | データを更新するかどうかを判断するための条件として使用する列の名前を指定します。この列にロードするデータの値が、この列の現在の値以上である場合にのみデータが更新されます。詳細については、[ロードを介してデータを変更する](../../../loading/Load_to_Primary_Key_tables.md)を参照してください。<br />**注意**<br />条件付き更新は、プライマリキーテーブルのみサポートされています。指定する列はプライマリキーカラムではない必要があります。 |
| format                    | いいえ   | ロードするデータの形式です。有効な値：`CSV`、`JSON`、および`Avro`（v3.0.1以降でサポート）。デフォルト値：`CSV`。 |
| trim_space                | いいえ   | データファイルがCSV形式の場合、列区切り記号の前後のスペースを削除するかどうかを指定します。タイプ：BOOLEAN。デフォルト値：`false`。<br />一部のデータベースでは、データをCSV形式のデータファイルとしてエクスポートすると、列区切り記号にスペースが追加されます。このようなスペースは、その位置に応じて先行スペースまたは後続スペースと呼ばれます。`trim_space`パラメータを設定することで、StarRocksによってデータロード時にそのような不要なスペースを削除できます。<br />StarRocksは、`enclose`で指定された文字で囲まれたフィールド内のスペース（先行スペースおよび後続スペースを含む）を削除しません。たとえば、次のフィールド値は、パイプ（`|`）を列区切り記号とし、ダブルクォーテーションマーク（`"`）を`enclose`で指定された文字として使用しています：`| "Love StarRocks" |`。`trim_space`を`true`に設定すると、StarRocksは、先行フィールド値を`| "Love StarRocks" |`として処理します。 |
| enclose                   | いいえ   | CSV形式のデータファイルのフィールド値を、[RFC4180](https://www.rfc-editor.org/rfc/rfc4180)に従って囲むために使用する文字を指定します。タイプ：半角文字。デフォルト値：`NONE`。最も一般的な文字はシングルクォーテーションマーク（`'`）とダブルクォーテーションマーク（`"`）です。<br />`enclose`で指定された文字で囲まれたすべての特殊文字（行区切り記号および列区切り記号を含む）は、通常の記号と見なされます。StarRocksはRFC4180よりも多くのことができるため、`enclose`で指定された任意の半角文字を`enclose`で指定された文字として使用できます。<br />フィールド値に`enclose`で指定された文字が含まれる場合は、同じ文字を使用してその`enclose`で指定された文字をエスケープできます。たとえば、`enclose`を`"`に設定し、フィールド値が`a "quoted" c`の場合、フィールド値を`"a ""quoted"" c"`としてデータファイルに入力できます。 |
| escape                    | いいえ   | 行区切り記号、列区切り記号、エスケープ文字、および`enclose`で指定された文字など、さまざまな特殊文字をエスケープするために使用する文字を指定します。タイプ：半角文字。デフォルト値：`NONE`。最も一般的な文字はスラッシュ（`\`）ですが、SQLステートメントではスラッシュを2重に書く必要があります（`\\`）。<br />**注意**<br />`escape`で指定された文字は、`enclose`で指定された文字のペアの内側と外側の両方に適用されます。<br />2つの例を以下に示します。<br /><ul><li>`enclose`を`"`に設定し、`escape`を`\`に設定した場合、StarRocksは`"say \"Hello world\""`を`say "Hello world"`と解析します。</li><li>列区切り記号がカンマ（`,`）であるとします。`escape`を`\`に設定した場合、StarRocksは`a, b\, c`を2つの別々のフィールド値`a`と`b, c`に解析します。</li></ul> |
| strip_outer_array         | いいえ   | JSON形式のデータの最も外側の配列構造を削除するかどうかを指定します。有効な値：`true`および`false`。デフォルト値：`false`。実世界のビジネスシナリオでは、JSON形式のデータには、角括弧`[]`で示される最も外側の配列構造が含まれる場合があります。この場合、`true`にこのパラメータを設定することをお勧めします。StarRocksは、最も外側の角括弧`[]`を削除し、各内部配列を個別のデータレコードとしてロードします。`false`にこのパラメータを設定すると、StarRocksは、JSON形式のデータ全体を1つの配列として解析し、配列を単一のデータレコードとしてロードします。次のJSON形式のデータ`[{"category" : 1, "author" : 2}, {"category" : 3, "author" : 4} ]`を例に挙げます。このパラメータを`true`に設定すると、`{"category" : 1, "author" : 2}`と`{"category" : 3, "author" : 4}`が2つの別々のデータレコードとして解析され、2つのStarRocksデータ行にロードされます。 |
| jsonpaths                 | いいえ   | JSON形式のデータからロードするフィールドの名前です。このパラメータの値は有効なJsonPath式です。詳細については、このトピックの[StarRocksテーブルには式を使用して生成された派生列が含まれています](#starrocks-table-contains-derived-columns-whose-values-are-generated-by-using-expressions)を参照してください。 |
| json_root                 | いいえ   | ロードするJSON形式のデータのルート要素です。StarRocksは、解析のためにルートノードの要素を`json_root`を介して抽出します。デフォルトでは、このパラメータの値は空であり、すべてのJSON形式のデータがロードされます。詳細については、このトピックの[ロードするJSON形式のデータのルート要素を指定する](#specify-the-root-element-of-the-json-formatted-data-to-be-loaded)を参照してください。 |
| task_consume_second | いいえ | 指定されたRoutine Loadジョブ内の各Routine Loadタスクがデータを消費する最大時間。単位：秒。[FE動的パラメータ](../../../administration/Configuration.md) `routine_load_task_consume_second`（クラスタ内のすべてのRoutine Loadジョブに適用される）とは異なり、このパラメータは個々のRoutine Loadジョブに固有であり、より柔軟です。このパラメータはv3.1.0以降でサポートされています。<ul> <li>`task_consume_second`と`task_timeout_second`が設定されていない場合、StarRocksはFE動的パラメータ`routine_load_task_consume_second`および`routine_load_task_timeout_second`を使用してロード動作を制御します。</li> <li>`task_consume_second`のみが設定されている場合、`task_timeout_second`のデフォルト値は`task_consume_second` * 4となります。</li> <li>`task_timeout_second`のみが設定されている場合、`task_consume_second`のデフォルト値は`task_timeout_second`/4となります。</li> </ul> |
|task_timeout_second|いいえ|指定されたRoutine Loadジョブ内の各Routine Loadタスクのタイムアウト期間。単位：秒。[FE動的パラメータ](../../../administration/Configuration.md) `routine_load_task_timeout_second`（クラスタ内のすべてのRoutine Loadジョブに適用される）とは異なり、このパラメータは個々のRoutine Loadジョブに固有であり、より柔軟です。このパラメータはv3.1.0以降でサポートされています。<ul> <li>`task_consume_second`と`task_timeout_second`が設定されていない場合、StarRocksはFE動的パラメータ`routine_load_task_consume_second`および`routine_load_task_timeout_second`を使用してロード動作を制御します。</li> <li>`task_timeout_second`のみが設定されている場合、`task_consume_second`のデフォルト値は`task_timeout_second`/4となります。</li> <li>`task_consume_second`のみが設定されている場合、`task_timeout_second`のデフォルト値は`task_consume_second` * 4となります。</li> </ul>|

### `data_source`、`data_source_properties`

必須です。データソースと関連するプロパティです。

```sql
FROM <data_source>
 ("<key1>" = "<value1>"[, "<key2>" = "<value2>" ...])
```

`data_source`

必須です。ロードするデータのソースです。有効な値：`KAFKA`。

`data_source_properties`

データソースのプロパティです。

| プロパティ          | 必須 | 説明                                                  |
| ----------------- | ---- | ----------------------------------------------------- |
| kafka_broker_list | はい  | Kafkaのブローカー接続情報です。形式は`<kafka_broker_ip>:<broker_ port>`です。複数のブローカーはカンマ（,）で区切られます。Kafkaブローカーのデフォルトポートは`9092`です。例：`"kafka_broker_list" = ""xxx.xx.xxx.xx:9092,xxx.xx.xxx.xx:9092"`。 |
| kafka_topic       | はい  | 消費するKafkaトピックです。Routine Loadジョブは1つのトピックからのメッセージのみを消費できます。 |
| kafka_partitions  | いいえ | 消費するKafkaパーティションです。たとえば、`"kafka_partitions" = "0, 1, 2, 3"`と指定できます。このプロパティが指定されていない場合、デフォルトですべてのパーティションが消費されます。 |
| kafka_offsets     | いいえ | `kafka_partitions`で指定されたKafkaパーティションでデータを消費する開始オフセットです。このプロパティは、`kafka_partitions`で指定されたKafkaパーティションでデータを消費する開始オフセットを指定します。このプロパティが指定されていない場合、Routine Loadジョブは`kafka_partitions`で指定された最新のオフセットからデータを消費します。有効な値：<ul><li>特定のオフセット：特定のオフセットからデータを消費します。</li><li>`OFFSET_BEGINNING`：最も早いオフセットからデータを消費します。</li><li>`OFFSET_END`：最新のオフセットからデータを消費します。</li></ul>複数の開始オフセットはカンマ（,）で区切られます。たとえば、`"kafka_offsets" = "1000, OFFSET_BEGINNING, OFFSET_END, 2000"`です。|
| property.kafka_default_offsets| いいえ|すべてのコンシューマーパーティションのデフォルトの開始オフセットを指定します。このプロパティのサポートされる値は、`kafka_offsets`プロパティと同じです。|
| confluent.schema.registry.url|いいえ|Avroスキーマが登録されているスキーマレジストリのURLです。StarRocksは、このURLを使用してAvroスキーマを取得します。形式は次のとおりです。<br />`confluent.schema.registry.url = http[s]://[<schema-registry-api-key>:<schema-registry-api-secret>@]<hostname or ip address>[:<port>]`|

#### その他のデータソース関連のプロパティ

Kafkaに関連する追加のデータソース（Kafka）関連のプロパティを指定できます。サポートされるプロパティの詳細については、[librdkafka configuration properties](https://github.com/confluentinc/librdkafka/blob/master/CONFIGURATION.md)のKafkaコンシューマクライアントのプロパティを参照してください。

> **注意**
>
> プロパティの値がファイル名の場合は、ファイル名の前にキーワード`FILE:`を追加してください。ファイルの作成方法については、[CREATE FILE](../Administration/CREATE_FILE.md)を参照してください。

- **消費するすべてのパーティションのデフォルトの初期オフセットを指定する**

```SQL
"property.kafka_default_offsets" = "OFFSET_BEGINNING"
```

- **Routine Loadジョブで使用されるコンシューマーグループのIDを指定する**

```SQL
"property.group.id" = "group_id_0"
```

`property.group.id`が指定されていない場合、StarRocksはRoutine Loadジョブの名前に基づいてランダムな値を生成し、`{job_name}_{random uuid}`の形式でランダムな値を生成します。たとえば、`simple_job_0a64fe25-3983-44b2-a4d8-f52d3af4c3e8`です。

- **BEがKafkaにアクセスするために使用するセキュリティプロトコルと関連するパラメータを指定する**

  セキュリティプロトコルは、`plaintext`（デフォルト）、`ssl`、`sasl_plaintext`、または`sasl_ssl`として指定できます。指定したセキュリティプロトコルに応じて関連するパラメータを設定する必要があります。

  セキュリティプロトコルを`ssl`として使用してKafkaにアクセスする場合：

  ```SQL
  -- セキュリティプロトコルをSSLとして指定します。
  "property.security.protocol" = "ssl"
  -- カフカブローカーのキーを検証するためのCA証明書（またはディレクトリ）のファイルパスです。
  -- Kafkaサーバーがクライアント認証を有効にしている場合、次の3つのパラメータも必要です。
  -- 認証に使用されるクライアントの公開鍵のパスです。
  "property.ssl.certificate.location" = "FILE:client.pem"
  -- 認証に使用されるクライアントの秘密鍵のパスです。
  "property.ssl.key.location" = "FILE:client.key"
  -- クライアントの秘密鍵のパスワードです。
  "property.ssl.key.password" = "xxxxxx"
  ```

  SASL_PLAINTEXTセキュリティプロトコルとSASL/PLAIN認証メカニズムを使用してKafkaにアクセスする場合：

  ```SQL
  -- セキュリティプロトコルをSASL_PLAINTEXTとして指定します。
  "property.security.protocol" = "SASL_PLAINTEXT"
  -- PLAINというシンプルなユーザー名/パスワード認証メカニズムとして指定します。
  "property.sasl.mechanism" = "PLAIN" 
  -- SASLユーザー名
  "property.sasl.username" = "admin"
  -- SASLパスワード
  "property.sasl.password" = "xxxxxx"
  ```

### FEおよびBEの設定項目

Routine Loadに関連するFEおよびBEの設定項目については、[設定項目](../../../administration/Configuration.md)を参照してください。

## 列マッピング

### CSV形式のデータのロードのための列マッピングの構成

CSV形式のデータの列がStarRocksテーブルの列と1対1でマッピングできる場合、データファイルとStarRocksテーブルの列の間の列マッピングを構成する必要はありません。

CSV形式のデータの列がStarRocksテーブルの列と1対1でマッピングできない場合、`columns`パラメータを使用してデータファイルとStarRocksテーブルの列の間の列マッピングを構成する必要があります。これには次の2つの使用例があります。

- **列の数は同じですが、列の順序が異なります。また、データファイルのデータは、一致するStarRocksテーブルの列にロードされる前に関数によって計算する必要はありません。**

  - `columns`パラメータで、データファイルの列と同じ順序でStarRocksテーブルの列の名前を指定する必要があります。

  - たとえば、StarRocksテーブルは、`col1`、`col2`、および`col3`の3つの列で構成され、データファイルも3つの列で構成されており、データファイルの列はStarRocksテーブルの列`col3`、`col2`、および`col1`にマッピングできる場合、`"columns: col3, col2, col1"`を指定する必要があります。

- **列の数が異なり、列の順序も異なります。また、データファイルのデータは、一致するStarRocksテーブルの列にロードされる前に関数によって計算する必要があります。**

  `columns`パラメータで、データファイルの列と同じ順序でStarRocksテーブルの列の名前を指定し、データを計算するために使用する関数を指定する必要があります。2つの例を以下に示します。

  - StarRocksテーブルは、`col1`、`col2`、および`col3`の3つの列で構成されています。データファイルは4つの列で構成されており、最初の3つの列はStarRocksテーブルの列`col1`、`col2`、および`col3`に順番にマッピングでき、4番目の列はStarRocksテーブルのいずれの列にもマッピングできません。この場合、データファイルの4番目の列に一時的な名前を指定する必要があります。一時的な名前は、StarRocksテーブルの列名とは異なる名前である必要があります。たとえば、`"columns: col1, col2, col3, temp"`と指定できます。データファイルの4番目の列は一時的に`temp`という名前になります。

  - StarRocksテーブルは、`year`、`month`、および`day`の3つの列で構成されています。データファイルは、`yyyy-mm-dd hh:mm:ss`形式の日時値を収容する1つの列のみで構成されています。この場合、`"columns: col, year = year(col), month=month(col), day=day(col)"`と指定できます。ここで、`col`はデータファイルの列の一時的な名前であり、関数`year = year(col)`、`month=month(col)`、および`day=day(col)`は、データファイルの列`col`からデータを抽出し、マッピングされたStarRocksテーブルの列にデータをロードするために使用されます。たとえば、`year = year(col)`は、データファイルの列`col`から`yyyy`データを抽出し、データをStarRocksテーブルの列`year`にロードするために使用されます。

詳細な例については、[列マッピングの構成](#configure-column-mapping)を参照してください。

### JSON形式またはAvro形式のデータのロードのための列マッピングの構成

> **注意**
>
> v3.0.1以降、StarRocksはルーチンロードを使用してAvroデータをロードすることができます。JSONデータまたはAvroデータをロードする場合、列のマッピングと変換の設定は同じです。そのため、このセクションでは、JSONデータを例として使用して設定の紹介を行います。

JSON形式のデータのキーがStarRocksテーブルの列と同じ名前である場合、シンプルモードを使用してJSON形式のデータをロードすることができます。シンプルモードでは、`jsonpaths`パラメータを指定する必要はありません。このモードでは、JSON形式のデータは`{}`で囲まれたオブジェクトである必要があります。例えば、`{"category": 1, "author": 2, "price": "3"}`というデータです。この例では、`category`、`author`、`price`はキー名であり、これらのキーはStarRocksテーブルの`category`、`author`、`price`の列と1対1でマッピングされます。詳細な例については、[シンプルモード](#ターゲットテーブルの列名がJSONキーと一致している)を参照してください。

JSON形式のデータのキーがStarRocksテーブルの列と異なる名前である場合、マッチモードを使用してJSON形式のデータをロードすることができます。マッチモードでは、`jsonpaths`および`COLUMNS`パラメータを使用して、JSON形式のデータとStarRocksテーブルの列との間の列マッピングを指定する必要があります。

- `jsonpaths`パラメータでは、JSON形式のデータ内のキーをJSON形式のデータの配置順に指定します。
- `COLUMNS`パラメータでは、JSON形式のデータのキーとStarRocksテーブルの列とのマッピングを指定します：
  - `COLUMNS`パラメータで指定された列名は、JSON形式のデータと1対1でマッピングされます。
  - `COLUMNS`パラメータで指定された列名は、StarRocksテーブルの列と1対1で名前によってマッピングされます。

詳細な例については、[StarRocksテーブルには式を使用して生成された派生列が含まれている](#StarRocksテーブルには式を使用して生成された派生列が含まれている)を参照してください。

## 例

### CSV形式のデータをロードする

このセクションでは、CSV形式のデータを使用して、さまざまなパラメータ設定と組み合わせを使用して、さまざまなロード要件を満たす方法を説明します。

**データセットの準備**

データセットからCSV形式のデータをロードすることを考えます。データセットの各メッセージには、6つの列（注文ID、支払い日、顧客名、国籍、性別、価格）が含まれています。

```plaintext
2020050802,2020-05-08,Johann Georg Faust,Deutschland,male,895
2020050802,2020-05-08,Julien Sorel,France,male,893
2020050803,2020-05-08,Dorian Grey,UK,male,1262
2020050901,2020-05-09,Anna Karenina,Russia,female,175
2020051001,2020-05-10,Tess Durbeyfield,US,female,986
2020051101,2020-05-11,Edogawa Conan,japan,male,8924
```

**テーブルの作成**

CSV形式のデータの列に基づいて、データベース`example_db`内に`example_tbl1`という名前のテーブルを作成します。

```SQL
CREATE TABLE example_db.example_tbl1 ( 
    `order_id` bigint NOT NULL COMMENT "Order ID",
    `pay_dt` date NOT NULL COMMENT "Payment date", 
    `customer_name` varchar(26) NULL COMMENT "Customer name", 
    `nationality` varchar(26) NULL COMMENT "Nationality", 
    `gender` varchar(26) NULL COMMENT "Gender", 
    `price` double NULL COMMENT "Price") 
DUPLICATE KEY (order_id,pay_dt) 
DISTRIBUTED BY HASH(`order_id`); 
```

#### 指定されたパーティションとオフセットからデータを消費する

ルーチンロードジョブが指定されたパーティションとオフセットからデータを消費する必要がある場合、`kafka_partitions`および`kafka_offsets`パラメータを設定する必要があります。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1",
    "kafka_partitions" ="0,1,2,3,4", -- 消費するパーティション
    "kafka_offsets" = "1000, OFFSET_BEGINNING, OFFSET_END, 2000" -- 対応する初期オフセット
);
```

#### タスクの並列性を向上させることでロードパフォーマンスを向上させる

ロードパフォーマンスを向上させ、蓄積的な消費を回避するために、ルーチンロードジョブを作成する際に`desired_concurrent_number`の値を増やすことで、タスクの並列性を向上させることができます。タスクの並列性により、1つのルーチンロードジョブを可能な限り多くの並列タスクに分割することができます。

> **注意**
>
> ロードパフォーマンスを向上させる他の方法については、[ルーチンロードFAQ](../../../faq/loading/Routine_load_faq.md)を参照してください。

実際のタスクの並列性は、次の複数のパラメータの最小値によって決まります。

```SQL
min(alive_be_number, partition_number, desired_concurrent_number, max_routine_load_task_concurrent_num)
```

> **注意**
>
> 最大の実際のタスクの並列性は、alive BEノードの数または消費するパーティションの数のいずれかです。

したがって、消費するパーティションの数が7で、alive BEノードの数が5で、`max_routine_load_task_concurrent_num`がデフォルト値の5であるとします。実際のタスクの並列性を増やすために、他の2つのパラメータの値を増やすことができます。例えば、`desired_concurrent_number`の値を`5`（デフォルト値は`3`）に設定することができます。この場合、実際のタスクの並列性は`min(5,7,5,5)`で`5`に設定されます。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
"desired_concurrent_number" = "5" -- desired_concurrent_numberの値を5に設定する
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### 列のマッピングを設定する

CSV形式のデータの列の順序がターゲットテーブルの列と一致しない場合、CSV形式のデータとターゲットテーブルの列との列マッピングを`COLUMNS`パラメータを使用して指定する必要があります。

**ターゲットデータベースとテーブル**

CSV形式のデータの列に基づいて、ターゲットデータベース`example_db`内に`example_tbl2`という名前のテーブルを作成します。このシナリオでは、性別を格納するための5つの列を作成する必要があります。

```SQL
CREATE TABLE example_db.example_tbl2 ( 
    `order_id` bigint NOT NULL COMMENT "Order ID",
    `pay_dt` date NOT NULL COMMENT "Payment date", 
    `customer_name` varchar(26) NULL COMMENT "Customer name", 
    `nationality` varchar(26) NULL COMMENT "Nationality", 
    `price` double NULL COMMENT "Price"
) 
DUPLICATE KEY (order_id,pay_dt) 
DISTRIBUTED BY HASH(order_id); 
```

**ルーチンロードジョブ**

この例では、CSV形式のデータの5番目の列はターゲットテーブルにインポートする必要がないため、`COLUMNS`で列マッピングを指定する際に5番目の列を一時的に`temp_gender`として指定し、他の列は直接`example_tbl2`にマッピングします。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl2_ordertest1 ON example_tbl2
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, temp_gender, price)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### フィルタ条件を設定する

特定の条件を満たすデータのみをロードする場合、`WHERE`句でフィルタ条件を設定することができます。例えば、`price > 100`というフィルタ条件を設定します。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl2_ordertest1 ON example_tbl2
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price),
WHERE price > 100 -- フィルタ条件を設定する
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### NULL値を持つ行をフィルタリングするために厳密モードを有効にする

`PROPERTIES`で`"strict_mode" = "true"`を設定することで、ルーチンロードジョブが厳密モードで実行されるようにすることができます。ソースの列に`NULL`値が存在するが、対象のStarRocksテーブルの列がNULL値を許容しない場合、ソースの列にNULL値が含まれる行はフィルタリングされます。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
"strict_mode" = "true" -- 厳密モードを有効にする
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### エラートレランスを設定する

ビジネスシナリオにおいて、不適格なデータに対する許容度が低い場合、エラー検出ウィンドウとエラーデータ行の最大数を設定することで、エラートレランスを設定することができます。エラー検出ウィンドウ内のエラーデータ行の数が`max_error_number`の値を超えると、ルーチンロードジョブが一時停止します。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
"max_batch_rows" = "100000",-- max_batch_rowsの値に10を乗じたものがエラー検出ウィンドウになります。
"max_error_number" = "100" -- エラー検出ウィンドウ内で許容されるエラーデータ行の最大数です。
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### SSLと関連するパラメータを設定する

BEがKafkaにアクセスするためにSSLを使用するセキュリティプロトコルを指定する必要がある場合、`"property.security.protocol" = "ssl"`および関連するパラメータを設定する必要があります。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
-- セキュリティプロトコルをSSLに指定します。
"property.security.protocol" = "ssl",
-- CA証明書の場所。
"property.ssl.ca.location" = "FILE:ca-cert",
-- Kafkaクライアントの公開鍵の場所。
"property.ssl.certificate.location" = "FILE:client.pem",
-- Kafkaクライアントの秘密鍵の場所。
"property.ssl.key.location" = "FILE:client.key",
-- Kafkaクライアントの秘密鍵のパスワード。
"property.ssl.key.password" = "abcdefg"
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### trim_space、enclose、およびescapeを設定する

Kafkaトピック`test_csv`からCSV形式のデータをロードし、列区切り記号の前後のスペースを削除し、`enclose`を`"`に設定し、`escape`を`\`に設定する場合、次のコマンドを実行します。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_test_csv ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
    "trim_space"="true",
    "enclose"="\"",
    "escape"="\\",
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic"="test_csv",
    "property.kafka_default_offsets"="OFFSET_BEGINNING"
);
```

### JSON形式のデータをロードする

#### StarRocksテーブルの列名がJSONキーと一致している

**データセットの準備**

例として、Kafkaトピック`ordertest2`に次のようなJSON形式のデータが存在するとします。

```SQL
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}
```

> **注意** 各JSONオブジェクトは1つのKafkaメッセージ内にある必要があります。そうでない場合、JSON形式のデータの解析に失敗するエラーが発生します。

**ターゲットデータベースとテーブル**

StarRocksクラスタ内のデータベース`example_db`に`example_tbl3`という名前のテーブルを作成します。列名はJSONキー名と一致しています。

```SQL
CREATE TABLE example_db.example_tbl3 ( 
    commodity_id varchar(26) NULL, 
    customer_name varchar(26) NULL, 
    country varchar(26) NULL, 
    pay_time bigint(20) NULL, 
    price double SUM NULL COMMENT "Price") 
AGGREGATE KEY(commodity_id,customer_name,country,pay_time)
DISTRIBUTED BY HASH(commodity_id); 
```

**ルーチンロードジョブ**

ルーチンロードジョブではシンプルモードを使用することができます。つまり、ルーチンロードジョブを作成する際に`jsonpaths`および`COLUMNS`パラメータを指定する必要はありません。StarRocksは、Kafkaクラスタの`ordertest2`トピック内のJSON形式のデータのキーをターゲットテーブル`example_tbl3`の列名に基づいて抽出し、JSON形式のデータをターゲットテーブルにロードします。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl3_ordertest2 ON example_tbl3
PROPERTIES
(
    "format" = "json"
 )
FROM KAFKA
(
    "kafka_broker_list" = "<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest2"
);
```

> **注意**
>
> - JSONデータの最も外側のレイヤーが配列構造である場合、`"strip_outer_array"="true"`を設定して外側の配列構造を削除する必要があります。また、`jsonpaths`を指定する場合、JSONデータの最も外側の配列構造が削除されたため、全体のJSONオブジェクトのルート要素がフラット化されます。
> - `json_root`を使用して、ロードするJSON形式のデータのルート要素を指定することができます。

#### StarRocksテーブルには式を使用して生成された派生列が含まれている

**データセットの準備**

例として、Kafkaクラスタの`ordertest2`トピックに次のようなJSON形式のデータが存在するとします。

```SQL
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}
```

**ターゲットデータベースとテーブル**

StarRocksクラスタ内のデータベース`example_db`に`example_tbl4`という名前のテーブルを作成します。`pay_dt`列は、JSON形式のデータのキー`pay_time`の値を計算して生成される派生列です。

```SQL
CREATE TABLE example_db.example_tbl4 ( 
    `commodity_id` varchar(26) NULL, 
    `customer_name` varchar(26) NULL, 
    `country` varchar(26) NULL,
    `pay_time` bigint(20) NULL,  
    `pay_dt` date NULL, 
    `price` double SUM NULL) 
AGGREGATE KEY(`commodity_id`,`customer_name`,`country`,`pay_time`,`pay_dt`) 
DISTRIBUTED BY HASH(`commodity_id`); 
```

**ルーチンロードジョブ**

ルーチンロードジョブではマッチモードを使用することができます。つまり、ルーチンロードジョブを作成する際に`jsonpaths`および`COLUMNS`パラメータを指定する必要があります。

`jsonpaths`パラメータには、JSON形式のデータのキーをJSON形式のデータの配置順に指定する必要があります。

また、JSON形式のデータのキー`pay_time`の値を`pay_dt`列に格納する前にDATE型に変換する必要があるため、`COLUMNS`で`pay_dt=from_unixtime(pay_time,'%Y%m%d')`という計算を指定する必要があります。他のキーの値は、JSON形式のデータのキーとターゲットテーブルに直接マッピングすることができます。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl4_ordertest2 ON example_tbl4
COLUMNS(commodity_id, customer_name, country, pay_time, pay_dt=from_unixtime(pay_time, '%Y%m%d'), price)
PROPERTIES
(
    "format" = "json",
    "jsonpaths" = "[\"$.commodity_id\",\"$.customer_name\",\"$.country\",\"$.pay_time\",\"$.price\"]"
 )
FROM KAFKA
(
    "kafka_broker_list" = "<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest2"
);
```

> **注意**
>
> - JSONデータの最も外側のレイヤーが配列構造である場合、`"strip_outer_array"="true"`を設定して外側の配列構造を削除する必要があります。また、`jsonpaths`を指定する場合、JSONデータの最も外側の配列構造が削除されたため、全体のJSONオブジェクトのルート要素がフラット化されます。
> - `json_root`を使用して、ロードするJSON形式のデータのルート要素を指定することができます。

#### CASE式を使用して生成された派生列が含まれる

**データセットの準備**

例として、Kafkaトピック`topic-expr-test`に次のようなJSON形式のデータが存在するとします。

```JSON
{"key1":1, "key2": 21}
{"key1":12, "key2": 22}
{"key1":13, "key2": 23}
{"key1":14, "key2": 24}
```

**ターゲットデータベースとテーブル**

StarRocksクラスタ内のデータベース`example_db`に`tbl_expr_test`という名前のテーブルを作成します。ターゲットテーブル`tbl_expr_test`には2つの列が含まれており、`col2`列の値はJSONデータに対してCASE式を使用して生成されます。

```SQL
CREATE TABLE tbl_expr_test (
    col1 string, col2 string)
DISTRIBUTED BY HASH (col1);
```

**ルーチンロードジョブ**

ターゲットテーブルの`col2`列の値がCASE式を使用して生成されるため、ルーチンロードジョブの`COLUMNS`パラメータに対応する式を指定する必要があります。

```SQL
CREATE ROUTINE LOAD rl_expr_test ON tbl_expr_test
COLUMNS (
      key1,
      key2,
      col1 = key1,
      col2 = CASE WHEN key1 = "1" THEN "key1=1" 
                  WHEN key1 = "12" THEN "key1=12"
                  ELSE "nothing" END) 
PROPERTIES ("format" = "json")
FROM KAFKA
(
    "kafka_broker_list" = "<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "topic-expr-test"
);
```

**StarRocksテーブルのクエリ**

StarRocksテーブルをクエリします。結果は、`col2`列の値がCASE式の出力であることを示しています。

```SQL
MySQL [example_db]> SELECT * FROM tbl_expr_test;
+------+---------+
| col1 | col2    |
+------+---------+
| 1    | key1=1  |
| 12   | key1=12 |
| 13   | nothing |
| 14   | nothing |
+------+---------+
4 rows in set (0.015 sec)
```

#### ロードするJSON形式のデータのルート要素を指定する

`json_root`を使用して、ロードするJSON形式のデータのルート要素を指定する必要があります。値は有効なJsonPath式である必要があります。

**データセットの準備**

例として、Kafkaクラスタの`ordertest3`トピックに次のようなJSON形式のデータが存在するとします。また、ロードするJSON形式のデータのルート要素は`$.RECORDS`です。

```SQL
{"RECORDS":[{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875},{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895},{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}]}
```

**ターゲットデータベースとテーブル**

StarRocksクラスタ内のデータベース`example_db`に`example_tbl3`という名前のテーブルを作成します。

```SQL
CREATE TABLE example_db.example_tbl3 ( 
    commodity_id varchar(26) NULL, 
    customer_name varchar(26) NULL, 
    country varchar(26) NULL, 
    pay_time bigint(20) NULL, 
    price double SUM NULL) 
AGGREGATE KEY(commodity_id,customer_name,country,pay_time) 
ENGINE=OLAP
DISTRIBUTED BY HASH(commodity_id); 
```

**ルーチンロードジョブ**

`PROPERTIES`で`"json_root" = "$.RECORDS"`を設定して、ロードするJSON形式のデータのルート要素を指定します。また、ロードするJSON形式のデータが配列構造であるため、`"strip_outer_array" = "true"`を設定して、最も外側の配列構造を削除する必要があります。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl3_ordertest3 ON example_tbl3
PROPERTIES
(
    "format" = "json",
    "json_root" = "$.RECORDS",
    "strip_outer_array" = "true"
 )
FROM KAFKA
(
    "kafka_broker_list" = "<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest2"
);
```

### Avro形式のデータをロードする

v3.0.1以降、StarRocksはルーチンロードを使用してAvroデータをロードすることができます。
#### Avroスキーマはシンプルです

Avroスキーマが比較的シンプルであり、Avroデータのすべてのフィールドをロードする必要があります。

**データセットの準備**

- **Avroスキーマ**

    1. 以下のAvroスキーマファイル `avro_schema1.avsc` を作成します：

        ```json
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

    2. [スキーマレジストリ](https://docs.confluent.io/platform/current/schema-registry/index.html)にAvroスキーマを登録します。

- **Avroデータ**

Avroデータを準備し、それをKafkaトピック `topic_1` に送信します。

**ターゲットデータベースとテーブル**

Avroデータのフィールドに基づいて、StarRocksクラスタのターゲットデータベース `sensor` にテーブル `sensor_log1` を作成します。テーブルの列名はAvroデータのフィールド名と一致している必要があります。AvroデータをStarRocksにロードする際のデータ型のマッピングについては、[データ型のマッピング](#データ型のマッピング)を参照してください。

```SQL
CREATE TABLE sensor.sensor_log1 ( 
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

**ルーチンロードジョブ**

ルーチンロードジョブでは、シンプルモードを使用することができます。つまり、ルーチンロードジョブを作成する際にパラメータ `jsonpaths` を指定する必要はありません。以下のステートメントを実行して、AvroメッセージをKafkaトピック `topic_1` で消費し、データをデータベース `sensor` のテーブル `sensor_log1` にロードするためのルーチンロードジョブ `sensor_log_load_job1` を作成します。

```sql
CREATE ROUTINE LOAD sensor.sensor_log_load_job1 ON sensor_log1  
PROPERTIES  
(  
  "format" = "avro"  
)  
FROM KAFKA  
(  
  "kafka_broker_list" = "<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>,...",
  "confluent.schema.registry.url" = "http://172.xx.xxx.xxx:8081",  
  "kafka_topic"= "topic_1",  
  "kafka_partitions" = "0,1,2,3,4,5",  
  "property.kafka_default_offsets" = "OFFSET_BEGINNING"  
);
```

#### Avroスキーマにはネストされたレコード型のフィールドが含まれています

Avroスキーマにはネストされたレコード型のフィールドが含まれており、ネストされたレコード型のフィールドのサブフィールドをStarRocksにロードする必要があります。

**データセットの準備**

- **Avroスキーマ**

    1. 以下のAvroスキーマファイル `avro_schema2.avsc` を作成します。外部のAvroレコードには、`id`、`name`、`checked`、`sensor_type`、`data`の5つのフィールドが順番に含まれています。そして、フィールド `data` にはネストされたレコード `data_record` が含まれています。

        ```JSON
        {
            "type": "record",
            "name": "sensor_log",
            "fields" : [
                {"name": "id", "type": "long"},
                {"name": "name", "type": "string"},
                {"name": "checked", "type" : "boolean"},
                {"name": "sensor_type", "type": {"type": "enum", "name": "sensor_type_enum", "symbols" : ["TEMPERATURE", "HUMIDITY", "AIR-PRESSURE"]}},
                {"name": "data", "type": 
                    {
                        "type": "record",
                        "name": "data_record",
                        "fields" : [
                            {"name": "data_x", "type" : "boolean"},
                            {"name": "data_y", "type": "long"}
                        ]
                    }
                }
            ]
        }
        ```

    2. [スキーマレジストリ](https://docs.confluent.io/platform/current/schema-registry/index.html)にAvroスキーマを登録します。

- **Avroデータ**

Avroデータを準備し、それをKafkaトピック `topic_2` に送信します。

**ターゲットデータベースとテーブル**

Avroデータのフィールドに基づいて、StarRocksクラスタのターゲットデータベース `sensor` にテーブル `sensor_log2` を作成します。

外部レコードのフィールド `id`、`name`、`checked`、`sensor_type` の他に、ネストされたレコード `data_record` のサブフィールド `data_y` もロードする必要があるとします。

```sql
CREATE TABLE sensor.sensor_log2 ( 
    `id` bigint NOT NULL COMMENT "センサーID",
    `name` varchar(26) NOT NULL COMMENT "センサー名", 
    `checked` boolean NOT NULL COMMENT "チェック済み", 
    `sensor_type` varchar(26) NOT NULL COMMENT "センサータイプ",
    `data_y` long NULL COMMENT "センサーデータ" 
) 
ENGINE=OLAP 
DUPLICATE KEY (id) 
DISTRIBUTED BY HASH(`id`); 
```

**ルーチンロードジョブ**

ロードジョブを送信し、Avroデータのロードが必要なフィールドを `jsonpaths` で指定します。ネストされたレコードのサブフィールド `data_y` の場合、その `jsonpath` を `"$.data.data_y"` と指定する必要があります。

```sql
CREATE ROUTINE LOAD sensor.sensor_log_load_job2 ON sensor_log2  
PROPERTIES  
(  
  "format" = "avro",
  "jsonpaths" = "[\"$.id\",\"$.name\",\"$.checked\",\"$.sensor_type\",\"$.data.data_y\"]"
)  
FROM KAFKA  
(  
  "kafka_broker_list" = "<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>,...",
  "confluent.schema.registry.url" = "http://172.xx.xxx.xxx:8081",  
  "kafka_topic" = "topic_1",  
  "kafka_partitions" = "0,1,2,3,4,5",  
  "property.kafka_default_offsets" = "OFFSET_BEGINNING"  
);
```

#### AvroスキーマにはUnionフィールドが含まれています

**データセットの準備**

AvroスキーマにはUnionフィールドが含まれており、UnionフィールドをStarRocksにロードする必要があります。

- **Avroスキーマ**

    1. 以下のAvroスキーマファイル `avro_schema3.avsc` を作成します。外部のAvroレコードには、`id`、`name`、`checked`、`sensor_type`、`data`の5つのフィールドが順番に含まれています。そして、フィールド `data` はUnion型であり、`null` とネストされたレコード `data_record` の2つの要素を含んでいます。

        ```JSON
        {
            "type": "record",
            "name": "sensor_log",
            "fields" : [
                {"name": "id", "type": "long"},
                {"name": "name", "type": "string"},
                {"name": "checked", "type" : "boolean"},
                {"name": "sensor_type", "type": {"type": "enum", "name": "sensor_type_enum", "symbols" : ["TEMPERATURE", "HUMIDITY", "AIR-PRESSURE"]}},
                {"name": "data", "type": [null,
                        {
                            "type": "record",
                            "name": "data_record",
                            "fields" : [
                                {"name": "data_x", "type" : "boolean"},
                                {"name": "data_y", "type": "long"}
                            ]
                        }
                    ]
                }
            ]
        }
        ```

    2. [スキーマレジストリ](https://docs.confluent.io/platform/current/schema-registry/index.html)にAvroスキーマを登録します。

- **Avroデータ**

Avroデータを準備し、それをKafkaトピック `topic_3` に送信します。

**ターゲットデータベースとテーブル**

Avroデータのフィールドに基づいて、StarRocksクラスタのターゲットデータベース `sensor` にテーブル `sensor_log3` を作成します。

外部レコードのフィールド `id`、`name`、`checked`、`sensor_type` の他に、Union型フィールド `data` の要素 `data_record` のフィールド `data_y` もロードする必要があるとします。

```sql
CREATE TABLE sensor.sensor_log3 ( 
    `id` bigint NOT NULL COMMENT "センサーID",
    `name` varchar(26) NOT NULL COMMENT "センサー名", 
    `checked` boolean NOT NULL COMMENT "チェック済み", 
    `sensor_type` varchar(26) NOT NULL COMMENT "センサータイプ",
    `data_y` long NULL COMMENT "センサーデータ" 
) 
ENGINE=OLAP 
DUPLICATE KEY (id) 
DISTRIBUTED BY HASH(`id`); 
```

**ルーチンロードジョブ**

ロードジョブを送信し、Avroデータのロードが必要なフィールドを `jsonpaths` で指定します。Union型フィールド `data` のフィールド `data_y` の場合、その `jsonpath` を `"$.data.data_y"` と指定する必要があります。

```sql
CREATE ROUTINE LOAD sensor.sensor_log_load_job3 ON sensor_log3  
PROPERTIES  
(  
  "format" = "avro",
  "jsonpaths" = "[\"$.id\",\"$.name\",\"$.checked\",\"$.sensor_type\",\"$.data.data_y\"]"
)  
FROM KAFKA  
(  
  "kafka_broker_list" = "<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>,...",
  "confluent.schema.registry.url" = "http://172.xx.xxx.xxx:8081",  
  "kafka_topic" = "topic_1",  
  "kafka_partitions" = "0,1,2,3,4,5",  
  "property.kafka_default_offsets" = "OFFSET_BEGINNING"  
);
```

Union型フィールド `data` の値が `null` の場合、StarRocksテーブルの列 `data_y` にロードされる値は `null` です。Union型フィールド `data` の値がデータレコードの場合、列 `data_y` にロードされる値はLong型です。
