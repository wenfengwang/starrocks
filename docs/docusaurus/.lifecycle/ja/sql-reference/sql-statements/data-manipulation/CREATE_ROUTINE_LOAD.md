---
displayed_sidebar: "Japanese"
---

# ルーチンロードの作成

## 説明

Routine LoadはApache Kafka®からメッセージを継続的に消費し、StarRocksにデータをロードできます。Routine Loadは、KafkaクラスタからCSV、JSON、およびAvro (v3.0.1以降サポート)データを消費し、`plaintext`、`ssl`、`sasl_plaintext`、および`sasl_ssl`など複数のセキュリティプロトコルを介してKafkaにアクセスできます。

このトピックでは、CREATE ROUTINE LOADステートメントの構文、パラメータ、および例について説明します。

> **注記**
>
> - ルーチンロードのアプリケーションシナリオ、原則、および基本操作の詳細については、[Apache Kafka®からデータを継続的にロードする](../../../loading/RoutineLoad.md)を参照してください。
> - StarRocksテーブルにデータをロードできるのは、それらのStarRocksテーブルにINSERT権限があるユーザーのみです。INSERT権限がない場合は、[GRANT](../account-management/GRANT.md)でStarRocksクラスタに接続するユーザーにINSERT権限を付与する手順に従ってください。

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

オプション。StarRocksデータベースの名前。

`job_name`

必須。ルーチンロードジョブの名前。1つのテーブルは複数のルーチンロードジョブからデータを受け取ることができます。複数のルーチンロードジョブを区別するために、意味のあるルーチンロードジョブ名（例: Kafkaトピック名とおおよその作成時間）を設定することをお勧めします。ルーチンロードジョブ名は同じデータベース内で一意である必要があります。

`table_name`

必須。データがロードされるStarRocksテーブルの名前。

### `load_properties`

オプション。データのプロパティ。構文:

```SQL
[COLUMNS TERMINATED BY '<column_separator>'],
[ROWS TERMINATED BY '<row_separator>'],
[COLUMNS (<column1_name>[, <column2_name>, <column_assignment>, ... ])],
[WHERE <expr>],
[PARTITION (<partition1_name>[, <partition2_name>, ...])]
[TEMPORARY PARTITION (<temporary_partition1_name>[, <temporary_partition2_name>, ...])]
```

`COLUMNS TERMINATED BY`

CSV形式のデータの列セパレータ。デフォルトの列セパレータは`\t`（タブ）です。たとえば、`COLUMNS TERMINATED BY ","`を使用して列セパレータをカンマとして指定できます。

> **注記**
>
> - ここで指定された列セパレータが取り込まれるデータの列セパレータと同じであることを確認してください。
> - 50バイトを超えないカンマ（,）、タブ、またはパイプ(|)などのUTF-8文字列をテキストデリミタとして使用できます。
> - ヌル値は`\N`を使用して示します。たとえば、データレコードは3つの列で構成され、データレコードは最初と三番目の列にデータを保持していますが、二番目の列はデータを保持していません。このような場合、二番目の列に`NULL`値を示すために`\N`を使用する必要があります。これは、レコードを`a,\N,b`としてコンパイルする必要があることを意味します。情報なしでは`a,,b`が空の文字列を示す。

`ROWS TERMINATED BY`

CSV形式のデータの行セパレータ。デフォルトの行セパレータは`\n`です。

`COLUMNS`

ソースデータの列とStarRocksテーブルの列のマッピング。詳細については、このトピックの[Column mapping](#column-mapping)を参照してください。

- `column_name`: ソースデータの列が計算なしにStarRocksテーブルの列にマップできる場合、列名のみを指定すればよいです。これらの列はマップされた列と呼ばれます。
- `column_assignment`: ソースデータの列が直接StarRocksテーブルの列にマッピングできず、データのロード前に列の値を計算する必要がある場合は、`expr`で計算関数を指定する必要があります。これらの列は派生列と呼ばれます。
  StarRocksはまずマップされた列を解析しますので、派生列をマップされた列の後に配置することをお勧めします。

`WHERE`

フィルタ条件。フィルタ条件を満たすデータだけがStarRocksにロードできます。たとえば、`col1`の値が`100`より大きいかつ`col2`の値が`1000`に等しい行のみをインザットしたい場合は、`WHERE col1 > 100 and col2 = 1000`を使用できます。

> **注記**
>
> フィルタ条件で指定された列はソース列または派生列である場合があります。

`PARTITION`

StarRocksテーブルがパーティション`p0`、`p1`、`p2`、および`p3`に分散されており、StarRocksの`p1`、`p2`、および`p3`にのみデータをロードし、`p0`に保存されるデータをフィルタリングしたい場合は、`PARTITION(p1, p2, p3)`をフィルタ条件として指定できます。デフォルトでは、このパラメータを指定しないと、データはすべてのパーティションにロードされます。例:

```SQL
PARTITION (p1, p2, p3)
```

`TEMPORARY PARTITION`

データをロードしたい[一時パーティション](../../../table_design/Temporary_partition.md)の名前。カンマ（,）で区切られた複数の一時パーティションを指定できます。

### `job_properties`

必須。ロードジョブのプロパティ。構文:

```SQL
PROPERTIES ("<key1>" = "<value1>"[, "<key2>" = "<value2>" ...])
```

| **プロパティ**           | **必須** | **説明**                                                          |
| ------------------------ | ------- | --------------------------------------------------------------- |
| desired_concurrent_number | いいえ   | 単一のルーチンロードジョブの期待されるタスク並列性。デフォルト値: `3`。実際のタスク並列性は、複数のパラメータの最小値によって決定されます: `min(alive_be_number, partition_number, desired_concurrent_number, max_routine_load_task_concurrent_num)`。 <ul><li>`alive_be_number`: オンラインのBEノードの数。</li><li>`partition_number`: 消費されるパーティションの数。</li><li>`desired_concurrent_number`: 単一のルーチンロードジョブの期待されるタスク並列性。デフォルト値: `3`。</li><li>`max_routine_load_task_concurrent_num`: ルーチンロードジョブのデフォルトの最大タスク並列性。デフォルト値は`5`です。[FE dynamic parameter](../../../administration/Configuration.md#configure-fe-dynamic-parameters)を参照してください。</li></ul>最大の実際のタスク並列性は、オンラインのBEノードの数または消費されるパーティションの数によって決定されます。 |
| max_batch_interval         | いいえ   | タスクのスケジュール間隔、つまり、タスクが実行される頻度。単位: 秒。値の範囲: `5` ～ `60`。デフォルト値: `10`。`10`秒より短いスケジュールの場合、過度に高いロード頻度で多くのタブレットバージョンが生成されますので、値を`10`より大きく設定することをお勧めします。 |
| max_batch_rows             | いいえ   | このプロパティはエラー検出ウィンドウを定義するためだけに使用されます。ウィンドウは、単一のルーチンロードタスクによって消費されるデータの行数です。値は`10 * max_batch_rows`です。デフォルト値は`10 * 200000 = 200000`です。ルーチンロードタスクは、例えば無効なJSON形式のデータなど、StarRocksが解析できないエラーデータを検出します。 |
| max_error_number           | いいえ   | エラー検出ウィンドウ内で許容されるエラーデータ行の最大数。エラーデータ行の数がこの値を超えると、ロードジョブは一時停止します。 [SHOW ROUTINE LOAD](./SHOW_ROUTINE_LOAD.md)を実行し、`ErrorLogUrls`を使用してエラーログを表示できます。その後、エラーログに従ってKafkaのエラーを修正できます。デフォルト値は`0`で、これはエラー行が許可されていないことを意味します。<br />**注記** <br />エラーデータ行には、WHERE句で除外されたデータ行が含まれません。 |
| strict_mode                | いいえ   | [strict mode](../../../loading/load_concept/strict_mode.md)を有効にするかどうかを指定します。有効値: `true` および `false`。デフォルト値: `false`。厳密モードが有効になっている場合、ロードされたデータの列の値が`NULL`であり、かつ対象テーブルがこの列に対して`NULL`値を許可していない場合、データ行はフィルタされます。 |
| log_rejected_record_num | いいえ | ログに記録できる最大の不適格データ行数を指定します。このパラメータはv3.1以降でサポートされています。有効値: `0`、`-1`、およびゼロより大きい正の整数。デフォルト値: `0`。<ul><li>値が`0`の場合、除外されたデータ行はログに記録されません。</li><li>値が`-1`の場合、除外されたすべてのデータ行がログに記録されます。</li><li>`n`などのゼロより大きい正の整数は、各BEで最大`n`の不適格データ行を記録できます。</li></ul> |
| timezone                  | No           | ロードジョブで使用されるタイムゾーン。デフォルト値: `Asia/Shanghai`。このパラメータの値は、strftime()、alignment_timestamp()、from_unixtime() などの関数によって返される結果に影響します。このパラメータで指定されたタイムゾーンはセッションレベルのタイムゾーンです。詳細については、[タイムゾーンの設定](../../../administration/timezone.md)を参照してください。 |
| merge_condition           | No           | データを更新する条件として使用したい列の名前を指定します。データは、この列にロードされるデータの値がこの列の現在の値以上である場合にのみ更新されます。詳細については、[ロードを通じたデータの変更](../../../loading/Load_to_Primary_Key_tables.md)を参照してください。<br />**注意**<br />プライマリキー表のみ条件付き更新をサポートしています。指定した列はプライマリキー列にできません。 |
| format                    | No           | ロードするデータのフォーマット。有効な値: `CSV`、`JSON`、`Avro`(v3.0.1以降でサポート)。デフォルト値: `CSV`。 |
| trim_space                | No           | CSV形式のデータファイルで列の区切り文字の前や後ろにスペースがある場合にそれらを削除するかどうかを指定します。タイプ: BOOLEAN。デフォルト値: `false`。<br />一部のデータベースでは、CSV形式のデータファイルとしてデータをエクスポートする際に、列の区切り文字にスペースが追加されます。このようなスペースは、場所によってはleading spacesまたはtrailing spacesと呼ばれます。`trim_space`パラメータを設定することで、StarRocksによって不要なスペースをデータロード時に削除することができます。<br />なお、StarRocksは特定の文字で包まれた(leading spacesやtrailing spacesを含む)フィールド内のスペースを削除しません。例えば、区切り文字にpipe (`|`) を使用し、`enclose`指定文字としてダブルクォーテーション (`"`) を使用する場合、次のフィールド値があります:`| "Love StarRocks" |`。`trim_space`を`true`に設定すると、StarRocksは、前のフィールド値を次のように処理します:`|"Love StarRocks"|`。 |
| enclose                   | No           | CSV形式のデータファイルにおいて[RFC4180](https://www.rfc-editor.org/rfc/rfc4180)に従ってフィールド値を包むために使用される文字を指定します。タイプ: single-byte character。デフォルト値: `NONE`。最も一般的な文字は、シングルクォート (`'`) とダブルクォーテーション (`"`) です。<br />`enclose`指定文字で包まれたすべての特殊文字（行区切り文字や列区切り文字を含む）は、通常のシンボルと見なされます。StarRocksではRFC4180を越えて、`enclose`指定文字として任意のsingle-byte文字を指定することができます。<br />フィールド値に`enclose`指定文字を含む場合は、同じ文字を使用してその`enclose`指定文字をエスケープすることができます。たとえば、`enclose`を`"`に設定し、フィールド値が`a "quoted" c`であるとします。この場合、フィールド値を次のようにデータファイルに入力することができます: `"a ""quoted"" c"`。 |
| escape                    | No           | 行区切り文字、列区切り文字、エスケープ文字、`enclose`指定文字などのさまざまな特殊文字をエスケープするために使用される文字を指定します。タイプ: single-byte character。デフォルト値: `NONE`。最も一般的な文字はスラッシュ (`\`) で、SQLステートメント内ではダブルスラッシュ (`\\`) として書かれる必要があります。<br />**注意**<br />`escape`で指定された文字は、`enclose`指定文字のペアごとの内部および外部に適用されます。<br />次の2つの例を以下に示します：<ul><li>`enclose`を`"`、`escape`を`\`に設定した場合、StarRocksは`"say \"Hello world\""`を`say "Hello world"`にパースします。</li><li>列区切り文字がカンマ (`,`) である場合、`escape`を`\`に設定した場合、StarRocksは`a, b\, c`を`a`と`b, c`の2つの別々のフィールド値にパースします。</li></ul> |
| strip_outer_array         | No           | JSON形式のデータの最も外側にある配列構造を削除するかどうかを指定します。有効な値: `true`、`false`。デフォルト値: `false`。実際のビジネスシナリオでは、JSON形式のデータには外側に角かっこ `[]` で示される配列構造があります。このような状況では、このパラメータを`true`に設定することをお勧めします。その結果、StarRocksは外側の角かっこ `[]` を削除し、各内部配列を別々のデータレコードとしてロードします。`false`に設定した場合、StarRocksはJSON形式のデータ全体を1つの配列としてパースし、その配列を単一のデータレコードとしてロードします。JSON形式のデータ`[{"category" : 1, "author" : 2},  {"category" : 3, "author" : 4} ]`を例として使用します。このパラメータを`true`に設定すると、`{"category" : 1, "author" : 2}`と`{"category" : 3, "author" : 4}`は2つの別々のデータレコードとして解釈され、2つのStarRocksデータ行にロードされます。 |
| jsonpaths                 | No           | JSON形式のデータからロードしたいフィールドの名前。このパラメータの値は有効なJsonPath表現です。詳細については、このトピックの[StarRocksテーブルには式を使用して生成された値が含まれます](#starrocks-table-contains-derived-columns-whose-values-are-generated-by-using-expressions)を参照してください。 |
| json_root                 | No           | ロードするJSON形式のデータのルート要素。StarRocksは`json_root`を介してルートノードの要素を抽出してパースします。デフォルトで、このパラメータの値は空で、すべてのJSON形式のデータがロードされることを示します。詳細については、このトピックの[ロードするJSON形式データのルート要素を指定する](#specify-the-root-element-of-the-json-formatted-data-to-be-loaded)を参照してください。 |
| task_consume_second | No | 指定されたルーチンロードジョブ内の各ルーチンロードタスクがデータを消費する最大時間。単位: 秒。[FE動的パラメータ](../../../administration/Configuration.md)`routine_load_task_consume_second`(クラスタ内のすべてのルーチンロードジョブに適用される)とは異なり、このパラメータは個々のルーチンロードジョブに固有であり、より柔軟です。このパラメータはv3.1.0以降でサポートされています。<ul> <li>`task_consume_second`と`task_timeout_second`が構成されていない場合、StarRocksはロードの挙動を制御するためにFE動的パラメータ`routine_load_task_consume_second`と`routine_load_task_timeout_second`を使用します。</li> <li>`task_consume_second`が構成されている場合、`task_timeout_second`のデフォルト値は`task_consume_second` * 4として計算されます。</li> <li>`task_timeout_second`が構成されている場合、`task_consume_second`のデフォルト値は`task_timeout_second`/4として計算されます。</li> </ul> |
|task_timeout_second|No|指定されたルーチンロードジョブ内の各ルーチンロードタスクのタイムアウト期間。単位: 秒。[FE動的パラメータ](../../../administration/Configuration.md)`routine_load_task_timeout_second`(クラスタ内のすべてのルーチンロードジョブに適用される)とは異なり、このパラメータは個々のルーチンロードジョブに固有であり、より柔軟です。このパラメータはv3.1.0以降でサポートされています。<ul> <li>`task_consume_second`と`task_timeout_second`が構成されていない場合、StarRocksはロードの挙動を制御するためにFE動的パラメータ`routine_load_task_consume_second`と`routine_load_task_timeout_second`を使用します。</li> <li>`task_timeout_second`が構成されている場合、`task_consume_second`のデフォルト値は`task_timeout_second`/4として計算されます。</li> <li>`task_consume_second`が構成されている場合、`task_timeout_second`のデフォルト値は`task_consume_second` * 4として計算されます。</li> </ul> |

### `data_source`、`data_source_properties`

必須です。データソースと関連するプロパティ。

```sql
FROM <data_source>
 ("<key1>" = "<value1>"[, "<key2>" = "<value2>" ...])
```

`data_source`

必須です。ロードしたいデータのソース。有効な値: `KAFKA`。

`data_source_properties`

データソースのプロパティ。

| プロパティ          | 必須 | 説明                                                     |
| ----------------- | -------- | ------------------------------------------------------------ |
| kafka_broker_list | Yes      | Kafkaのブローカー接続情報。フォーマットは `<kafka_broker_ip>:<broker_ port>` です。複数のブローカーはカンマ (`,`) で区切られます。Kafkaブローカーが使用するデフォルトポートは `9092` です。例:`"kafka_broker_list" = ""xxx.xx.xxx.xx:9092,xxx.xx.xxx.xx:9092"`. |
| kafka_topic       | Yes      | 消費するKafkaのトピック。ルーチンロードジョブは1つのトピックからのメッセージのみを消費できます。 |
| kafka_partitions  | いいえ       | 購読するKafkaパーティション、例：`"kafka_partitions" = "0, 1, 2, 3"`。このプロパティが指定されていない場合、デフォルトですべてのパーティションが購読されます。|
| kafka_offsets     | いいえ       | Kafkaパーティションでデータを消費するための開始オフセット。`kafka_partitions`で指定されたKafkaパーティションの最新オフセットからRoutine Loadジョブがデータを消費します。有効な値: <ul><li>特定のオフセット：特定のオフセットからデータを消費します。</li><li>`OFFSET_BEGINNING`：可能な限り初期のオフセットからデータを消費します。</li><li>`OFFSET_END`：最新のオフセットからデータを消費します。</li></ul> 複数の開始オフセットはカンマ(,)で区切られます。例：`"kafka_offsets" = "1000, OFFSET_BEGINNING, OFFSET_END, 2000"`。|
| property.kafka_default_offsets| いいえ| すべてのコンシューマーパーティションのデフォルト開始オフセット。このプロパティのサポートされる値は、`kafka_offsets`プロパティと同じです。|
| confluent.schema.registry.url|いいえ |Avroスキーマが登録されているSchema RegistryのURL。StarRocksは、このURLを使用してAvroスキーマを取得します。 フォーマットは次のとおりです：<br />`confluent.schema.registry.url = http[s]://[<schema-registry-api-key>:<schema-registry-api-secret>@]<hostname or ip address>[:<port>]`|

#### その他のデータソース関連のプロパティ

Kafka関連のプロパティ（Kafka）を指定することができます。これらのプロパティは、Kafkaコマンドライン`--property`を使用するのと同等です。よりサポートされるプロパティについては、[librdkafka configuration properties](https://github.com/confluentinc/librdkafka/blob/master/CONFIGURATION.md)でKafkaコンシューマークライアントのプロパティを参照してください。

> **注意**
>
> プロパティの値がファイル名の場合は、ファイル名の前にキーワード `FILE:` を追加してください。ファイルの作成方法については、[CREATE FILE](../Administration/CREATE_FILE.md)を参照してください。

- **すべての購読されるパーティションのデフォルト初期オフセットを指定**

```SQL
"property.kafka_default_offsets" = "OFFSET_BEGINNING"
```

- **Routine Loadジョブで使用するコンシューマーグループのIDを指定**

```SQL
"property.group.id" = "group_id_0"
```

`property.group.id`が指定されていない場合、StarRocksはRoutine Loadジョブの名前に基づいてランダムな値を生成し、`{job_name}_{random uuid}`の形式で生成します。例：`simple_job_0a64fe25-3983-44b2-a4d8-f52d3af4c3e8`。

- **BEがKafkaへのアクセスに使用するセキュリティプロトコルと関連するパラメータを指定**

  セキュリティプロトコルは、`plaintext`（デフォルト）、`ssl`、`sasl_plaintext`、または`sasl_ssl`として指定することができます。指定したセキュリティプロトコルに応じて、関連するパラメータを構成する必要があります。

  セキュリティプロトコルが`sasl_plaintext`または`sasl_ssl`に設定されている場合、以下のSASL認証メカニズムがサポートされます。

  - PLAIN
  - SCRAM-SHA-256とSCRAM-SHA-512
  - OAUTHBEARER

  - SSLセキュリティプロトコルを使用してKafkaにアクセスする場合：

    ```SQL
    -- セキュリティプロトコルをSSLとして指定
    "property.security.protocol" = "ssl"
    -- Kafkaブローカーのキーを検証するためのCA証明書のファイルまたはディレクトリパス
    -- Kafkaサーバーがクライアント認証を有効にしている場合、次の3つのパラメータも必要です：
    -- 認証に使用されるクライアントの公開鍵へのパス
    "property.ssl.certificate.location" = "FILE:client.pem"
    -- 認証に使用されるクライアントの秘密鍵へのパス
    "property.ssl.key.location" = "FILE:client.key"
    -- クライアントの秘密鍵のパスワード
    "property.ssl.key.password" = "xxxxxx"
    ```

  - SASL_PLAINTEXTセキュリティプロトコルとSASL/PLAIN認証メカニズムを使用してKafkaにアクセスする場合：

    ```SQL
    -- セキュリティプロトコルをSASL_PLAINTEXTとして指定
    "property.security.protocol" = "SASL_PLAINTEXT"
    -- 使用するSASLメカニズムをPLAINとして指定
    "property.sasl.mechanism" = "PLAIN" 
    -- SASLユーザー名
    "property.sasl.username" = "admin"
    -- SASLパスワード
    "property.sasl.password" = "xxxxxx"
    ```

### FEおよびBEの構成項目

Routine Loadに関連するFEおよびBEの構成項目については、[構成項目](../../../administration/Configuration.md)を参照してください。

## カラムマッピング

### CSV形式のデータの読み込みのためのカラムマッピングを構成

CSV形式のデータのカラムがStarRocksテーブルのカラムと1対1でシーケンスにマッピングできる場合は、データとStarRocksテーブルのカラム間のカラムマッピングを構成する必要はありません。

CSV形式のデータのカラムがStarRocksテーブルのカラムと1対1でシーケンスにマッピングできない場合は、`columns`パラメータを使用してデータファイルとStarRocksテーブルのカラム間のカラムマッピングを構成する必要があります。以下に、そのうちの2つの使用ケースを示します。

- **同じ列数だが、異なる列順序。また、データはマッチングされたStarRocksテーブルの列に読み込まれる前に関数によって計算される必要がありません。**

  - `columns`パラメータでは、データファイルの列が配置されているとおりに、StarRocksテーブルの列の名前を指定する必要があります。

  - 例えば、StarRocksテーブルは`col1`、`col2`、`col3`の3つの列で構成されており、データファイルも3つの列で構成されていますが、これらの列がStarRocksテーブルの列 `col3`, `col2`, `col1` にマップできる場合は、`"columns: col3, col2, col1"`を指定する必要があります。

- **異なる列数および異なる列順序。また、データはマッチングされたStarRocksテーブルの列に読み込まれる前に関数によって計算される必要があります。**

  `columns`パラメータでは、データファイルの列が配置されているとおりに、StarRocksテーブルの列の名前を指定し、データを計算するために使用したい関数を指定する必要があります。2つの例を以下に示します。

  - StarRocksテーブルは`col1`、`col2`、`col3`の3つの列で構成されており、データファイルは4つの列で構成されている場合、最初の3つの列がStarRocksテーブルの列 `col1`, `col2`, `col3` にマップでき、4番目の列がStarRocksテーブルのいずれかの列にマップできない場合、データファイルの4番目の列に一時的な名前を指定する必要があります。たとえば、`"columns: col1, col2, col3, temp"`を指定します。この場合、データファイルの4番目の列は一時的な名前 `temp` になります。

  - StarRocksテーブルは`year`、`month`、`day`の3つの列で構成されており、データファイルは`yyyy-mm-dd hh:mm:ss`形式の日付と時刻の値を収納する1つの列のみを含む場合、`"columns: col, year = year(col), month=month(col), day=day(col)"`を指定します。この場合、`col`はデータファイル列の一時的な名前であり、`year = year(col)`, `month=month(col)`, `day=day(col)`という関数を使用して、データファイル列`col`からデータを抽出し、マッピングStarRocksテーブルの列にデータをロードします。たとえば、`year = year(col)`はデータファイル列`col`から`yyyy`データを抽出し、StarRocksテーブル列 `year` にデータをロードするために使用されます。

詳細な例については、[カラムマッピングの構成](#configure-column-mapping)を参照してください。

### JSON形式またはAvro形式のデータの読み込みのためのカラムマッピングを構成

> **注意**
>
> v3.0.1以降、StarRocksはRoutine Loadを使用してAvroデータの読み込みをサポートしています。JSONまたはAvroデータを読み込む場合、カラムマッピングおよび変換の構成は同じです。そのため、このセクションではカラムマッピングの構成について紹介するためにJSONデータを使用しています。

JSON形式のデータのキーがStarRocksテーブルの列と同じ名前を持っている場合、単純モードを使用してJSON形式のデータを読み込むことができます。単純モードでは、`jsonpaths`パラメータを指定する必要はありません。このモードでは、JSON形式のデータは、例えば`{"category": 1, "author": 2, "price": "3"}`のように波括弧 `{}` によってオブジェクトである必要があります。この場合、`category`、`author`、`price`はキー名であり、これらのキーはStarRocksテーブルの`category`、`author`、`price`に名前によって1対1でマッピングできる必要があります。詳細は、[simple mode](#Column names of the target table are consistent with JSON keys)を参照してください。
JSON形式のデータのキーがStarRocksテーブルの列と異なる名前を持つ場合は、一致モードを使用してJSON形式のデータをロードできます。一致モードでは、`jsonpaths`および`COLUMNS`パラメータを使用して、JSON形式のデータとStarRocksテーブルとの列マッピングを指定する必要があります。

- `jsonpaths`パラメータでは、JSON形式のデータ内でどのように配置されているかに応じて、JSONキーを順番に指定します。
- `COLUMNS`パラメータでは、JSONのキーとStarRocksテーブルの列とのマッピングを指定します:
  - `COLUMNS`パラメータで指定された列名は、JSON形式のデータへの1対1のマッピング順に配置されます。
  - `COLUMNS`パラメータで指定された列名は、StarRocksテーブルの列と名前による1対1のマッピングが行われます。

例については、[StarRocksテーブルには式を使用して生成された派生列が含まれています](#starrocks-table-contains-derived-columns-whose-values-are-generated-by-using-expressions)を参照してください。

## 例

### CSV形式のデータのロード

このセクションでは、さまざまなパラメータ設定や組み合わせを使用して、さまざまなロード要件に対応する方法を説明する例として、CSV形式のデータを使用します。

**データセットの準備**

データセットからCSV形式のデータを`ordertest1`というKafkaトピックからロードしたいとします。データセットの各メッセージには6つの列が含まれます: 注文ID、支払い日、顧客名、国籍、性別、価格。

```plaintext
2020050802,2020-05-08,Johann Georg Faust,Deutschland,male,895
2020050802,2020-05-08,Julien Sorel,France,male,893
2020050803,2020-05-08,Dorian Grey,UK,male,1262
2020050901,2020-05-09,Anna Karenina,Russia,female,175
2020051001,2020-05-10,Tess Durbeyfield,US,female,986
2020051101,2020-05-11,Edogawa Conan,japan,male,8924
```

**テーブルの作成**

CSV形式のデータの列に基づいて、データベース`example_db`に`example_tbl1`という名前のテーブルを作成します。

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

#### 指定されたパーティションおよびオフセットからデータの読み込みを開始する

ルーチンロードジョブが特定のパーティションとオフセットからデータの読み込みを開始する必要がある場合は、`kafka_partitions`および`kafka_offsets`パラメータを構成する必要があります。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1",
    "kafka_partitions" ="0,1,2,3,4", -- 読み込むパーティション
    "kafka_offsets" = "1000, OFFSET_BEGINNING, OFFSET_END, 2000" -- 対応する初期オフセット
);
```

#### タスク並列度を増やして読み込みパフォーマンスを向上させる

読み込みパフォーマンスを向上させ、蓄積された消費を回避するために、ルーチンロードジョブを作成する際に`desired_concurrent_number`の値を増やすことで、タスク並列度を増やすことができます。タスク並列度により、1つのルーチンロードジョブを可能な限り多くの並列タスクに分割することができます。

> **注記**
>
> 読み込みパフォーマンスを向上させる他の方法については、[Routine Load FAQ](../../../faq/loading/Routine_load_faq.md)を参照してください。

実際のタスク並列度は、次の複数のパラメータのうち最小の値によって決定されます。

```SQL
min(alive_be_number, partition_number, desired_concurrent_number, max_routine_load_task_concurrent_num)
```

> **注記**
>
> 実際のタスク並列度の最大値は、生きているBEノードの数または消費するパーティションの数のいずれかになります。

したがって、生きているBEノードの数および消費するパーティションの数が他の2つのパラメータ`max_routine_load_task_concurrent_num`および`desired_concurrent_number`の値よりも大きい場合、実際のタスク並列度を増やすために、他の2つのパラメータの値を増やすことができます。

消費するパーティションの数が7で生きているBEノードの数が5であり、`max_routine_load_task_concurrent_num`がデフォルト値である`5`であると仮定します。実際のタスク並列度を増やしたい場合は、`desired_concurrent_number`を`5` (デフォルト値は`3`)に設定します。この場合、実際のタスク並列度`min(5,7,5,5)`は`5`に設定されます。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
"desired_concurrent_number" = "5" -- desired_concurrent_numberの値を5に設定
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### 列のマッピングを構成する

CSV形式のデータの列の順序が対象テーブルの列と一致しない場合、CSV形式のデータと対象テーブルとの列マッピングを指定する必要があります。CSV形式のデータと対象テーブルとの列マッピングを`COLUMNS`パラメータを通じて指定する場合、CSV形式のデータの5番目の列を対象テーブルにインポートする必要がないと仮定します。

**対象のデータベースおよびテーブル**

CSV形式のデータの列に基づいて、対象のデータベース`example_db`に`example_tbl2`という名前のテーブルを作成します。このシナリオでは、性別を格納する5番目の列以外のCSV形式のデータの5つの列に対応する5つの列を作成する必要があります。

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

この例では、CSV形式のデータの5番目の列を対象テーブルにロードする必要がないため、5番目の列は一時的に`COLUMNS`で`temp_gender`として名前を付け、他の列は直接テーブル`example_tbl2`にマッピングされます。

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

#### フィルタ条件の設定

特定の条件を満たすデータのみをロードしたい場合は、`WHERE`句にフィルタ条件を設定することができます。たとえば、`price > 100`のような条件を設定できます。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl2_ordertest1 ON example_tbl2
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price),
WHERE price > 100 -- フィルタ条件を設定
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### NULL値を持つ行をフィルタリングするために厳密モードを有効にする

`PROPERTIES`で`"strict_mode" = "true"`を設定することで、ルーチンロードジョブを厳密モードにすることができます。ソース列に`NULL`値が存在するが、対象のStarRocksテーブル列が`NULL`値を許可していない場合、ソース列の`NULL`値を保持する行はフィルタリングされます。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
"strict_mode" = "true" -- strict_modeを有効にする
)
FROM KAFKA
(
```
    + "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    + "kafka_topic" = "ordertest1"
);

#### エラートレランスの設定

業務シナリオにおけるデータの不適合耐性が低い場合は、パラメータ `max_batch_rows` および `max_error_number` の設定によってエラー検出ウィンドウおよびエラー データ行の最大数を設定する必要があります。エラー検出ウィンドウ内のエラー データ行数が `max_error_number` の値を超えると、Routine Load ジョブが一時停止します。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
"max_batch_rows" = "100000",-- max_batch_rows の値に 10 を乗算したものがエラー検出ウィンドウとなります。
"max_error_number" = "100" -- エラー検出ウィンドウ内で許容されるエラー データ行数の最大数です。
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### SSL を使用したセキュリティ プロトコルの指定と関連するパラメータの構成

BE が Kafka にアクセスする際に SSL を使用したセキュリティ プロトコルを指定する必要がある場合は、`"property.security.protocol" = "ssl"` および関連するパラメータを構成する必要があります。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
-- SSL を使用したセキュリティ プロトコルの指定です。
"property.security.protocol" = "ssl",
-- CA 証明書の場所です。
"property.ssl.ca.location" = "FILE:ca-cert",
-- Kafka クライアントの認証が有効になっている場合、次のプロパティを構成する必要があります。
-- Kafka クライアントの公開キーの場所です。
"property.ssl.certificate.location" = "FILE:client.pem",
-- Kafka クライアントの秘密キーの場所です。
"property.ssl.key.location" = "FILE:client.key",
-- Kafka クライアントの秘密キーのパスワードです。
"property.ssl.key.password" = "abcdefg"
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### trim_space、enclose、escape の設定

`test_csv` という名前の Kafka トピックから CSV 形式のデータをロードしようとしています。データセット内の各メッセージには、次の 6 つの列が含まれています: オーダー ID、支払日、顧客名、国籍、性別、価格。

```Plaintext
 "2020050802" , "2020-05-08" , "Johann Georg Faust" , "Deutschland" , "male" , "895"
 "2020050802" , "2020-05-08" , "Julien Sorel" , "France" , "male" , "893"
 "2020050803" , "2020-05-08" , "Dorian Grey\,Lord Henry" , "UK" , "male" , "1262"
 "2020050901" , "2020-05-09" , "Anna Karenina" , "Russia" , "female" , "175"
 "2020051001" , "2020-05-10" , "Tess Durbeyfield" , "US" , "female" , "986"
 "2020051101" , "2020-05-11" , "Edogawa Conan" , "japan" , "male" , "8924"
```

`test_csv` トピックからすべてのデータを `example_tbl1` にロードし、列の区切り文字の前後の空白を削除し、`enclose` を `"` に設定し、`escape` を `\` に設定する場合、次のコマンドを実行します。

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
    "kafka_broker_list" = "<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic"="test_csv",
    "property.kafka_default_offsets"="OFFSET_BEGINNING"
);
```

### JSON 形式のデータをロード

#### StarRocks テーブルの列名が JSON キー名と一致している

**データセットの準備**

例えば、Kafka トピック `ordertest2` に次の JSON 形式のデータが存在するとします。

```SQL
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}
```

> **注記** 各 JSON オブジェクトは 1 つの Kafka メッセージ内にある必要があります。それ以外の場合、JSON 形式のデータの解析に失敗したことを示すエラーが発生します。

**対象データベースとテーブル**

StarRocks クラスターのターゲットデータベース `example_db` にテーブル `example_tbl3` を作成します。カラム名は JSON 形式のデータのキー名と一致しています。

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

**Routine Load ジョブ**

Routine Load ジョブをシンプル モードで使用できます。つまり、Routine Load ジョブを作成する際に `jsonpaths` および `COLUMNS` パラメータを指定する必要はありません。StarRocks は、Kafka クラスターのトピック `ordertest2` の JSON 形式のデータのキーを、対象テーブル `example_tbl3` の列名に従って抽出し、JSON 形式のデータをターゲット テーブルにロードします。

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

> **注記**
>
> - JSON 形式のデータの最も外側のレイヤーが配列構造の場合、`PROPERTIES` に `"strip_outer_array"="true"` を設定して外側の配列構造を削除する必要があります。さらに、`jsonpaths` を指定する必要がある場合、JSON 形式のデータの最も外側の配列構造が削除されているため、全体の JSON 形式のデータのルート要素はフラット化された JSON オブジェクトです。
> - `json_root` を使用して、JSON 形式のデータのルート要素を指定できます。

#### StarRocks テーブルには、式を使用して生成された値を持つ導出列が含まれています

**データセットの準備**

例えば、Kafka クラスターのトピック `ordertest2` に次の JSON 形式のデータが存在するとします。

```SQL
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}
```

**対象データベースとテーブル**

StarRocks クラスターのデータベース `example_db` に `example_tbl4` という名前のテーブルを作成します。`pay_dt` という列は、JSON 形式のデータのキー `pay_time` の値を計算して値が生成される導出列です。

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
```

**ルーチンロードジョブ**

ルーチンロードジョブでは、一致モードを使用できます。つまり、ルーチンロードジョブの作成時には、`jsonpaths`および`COLUMNS`パラメータを指定する必要があります。

`jsonpaths`パラメータで、JSON形式のデータのキーを指定し、シーケンスに沿って並べる必要があります。

また、JSON形式のデータ内のキー`pay_time`の値は、`pay_dt`列に格納される前にDATE型に変換する必要があります。これには、`COLUMNS`で`pay_dt=from_unixtime(pay_time,'%Y%m%d')`を使用して計算を指定する必要があります。JSON形式のデータ内の他のキーの値は、`example_tbl4`テーブルに直接マップできます。

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
> - JSONデータの最も外側のレイヤーが配列構造である場合は、`PROPERTIES`で`"strip_outer_array"="true"`を設定して、最も外側の配列構造を削除する必要があります。また、`jsonpaths`を指定する必要がある場合、JSONデータの最も外側の配列構造が削除されるため、全体のJSONデータのルート要素はフラット化されたJSONオブジェクトです。
> - `json_root`を使用して、JSON形式のデータのルート要素を指定できます。

#### StarRocksテーブルには、CASE式を使用して生成された派生列が含まれています

**データセットを準備**

例えば、以下のJSON形式のデータがKafkaトピック`topic-expr-test`に存在します。

```JSON
{"key1":1, "key2": 21}
{"key1":12, "key2": 22}
{"key1":13, "key2": 23}
{"key1":14, "key2": 24}
```

**ターゲットデータベースおよびテーブル**

StarRocksクラスターのデータベース`example_db`に`tbl_expr_test`という名前のテーブルを作成します。ターゲットテーブル`tbl_expr_test`には、`col2`列の値がJSONデータ上のCASE式を使用して計算される必要があります。

```SQL
CREATE TABLE tbl_expr_test (
    col1 string, col2 string)
DISTRIBUTED BY HASH (col1);
```

**ルーチンロードジョブ**

対象テーブルの`col2`列の値がCASE式を使用して生成されるため、ルーチンロードジョブの`COLUMNS`パラメータで対応する式を指定する必要があります。

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

**StarRocksテーブルをクエリ**

StarRocksテーブルをクエリします。その結果、`col2`列の値がCASE式の出力であることが示されます。

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

#### ロードするJSON形式のデータのルート要素を指定

`json_root`を使用して、ロードするJSON形式のデータのルート要素を指定する必要があります。また、その値は有効なJsonPath式である必要があります。

**データセットを準備**

例えば、以下のJSON形式のデータがKafkaクラスターのトピック`ordertest3`に存在します。そして、ロードするJSON形式のデータのルート要素は`$.RECORDS`です。

```SQL
{"RECORDS":[{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875},{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895},{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}]}
```

**ターゲットデータベースおよびテーブル**

StarRocksクラスターのデータベース`example_db`に`example_tbl3`という名前のテーブルを作成します。

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

`PROPERTIES`で`"json_root" = "$.RECORDS"`を設定して、ロードするJSON形式のデータのルート要素を指定できます。また、ロードするJSON形式のデータが配列構造であるため、最も外側の配列構造を削除するために`"strip_outer_array" = "true"`を設定する必要があります。

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

### Avro形式のデータをロード

v3.0.1以降、StarRocksはルーチンロードを使用してAvroデータをロードすることができます。

#### Avroスキーマが単純な場合

Avroスキーマが比較的単純な場合、Avroデータのすべてのフィールドをロードする必要があります。

**データセットを準備**

- **Avroスキーマ**

    1. 以下のAvroスキーマファイル`avro_schema1.avsc`を作成します。

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

    2. Avroスキーマを[スキーマレジストリ](https://docs.confluent.io/platform/current/schema-registry/index.html)に登録します。

- **Avroデータ**

Avroデータを準備し、それをKafkaトピック`topic_1`に送信します。

**ターゲットデータベースおよびテーブル**

Avroデータのフィールドに応じて、StarRocksクラスターのターゲットデータベース`sensor`に`sensor_log1`という名前のテーブルを作成します。テーブルの列名は、Avroデータのフィールド名に一致する必要があります。AvroデータをStarRocksにロードする際のデータ型のマッピングについては、[データ型のマッピング](#データ型のマッピング)を参照してください。

```SQL
CREATE TABLE sensor.sensor_log1 ( 
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

**ルーチンロードジョブ**
```
単純モードを使用して、Routine Load ジョブを行うことができます。つまり、Routine Load ジョブを作成する際に、パラメータ `jsonpaths` を指定する必要はありません。以下のステートメントを実行して、名前を `sensor_log_load_job1` とする Routine Load ジョブを提出し、Kafka トピック `topic_1` から Avro メッセージを処理してデータをデータベース `sensor` のテーブル `sensor_log1` にロードします。

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

#### Avroスキーマにはネストされたレコード型フィールドが含まれています

Avroスキーマにネストされたレコード型フィールドが含まれており、ネストされたレコード型フィールドのサブフィールドをStarRocksにロードする必要があるとします。

**データセットの準備**

- **Avroスキーマ**

    1. 次のAvroスキーマファイル `avro_schema2.avsc` を作成します。
    外部のAvroレコードには、順番に `id`、`name`、`checked`、`sensor_type`、および `data` の5つのフィールドが含まれています。そして、`data` フィールドにはネストされたレコード `data_record` が含まれています。

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

Avroデータを準備し、Kafkaトピック `topic_2` に送信します。

**ターゲットのデータベースおよびテーブル**

Avroデータのフィールドに基づいて、StarRocksクラスターのターゲットデータベース `sensor` にテーブル `sensor_log2` を作成します。

外部レコードのフィールド `id`、`name`、`checked`、`sensor_type` だけでなく、ネストされたレコード `data_record` のサブフィールド `data_y` もロードする必要があるとします。

```sql
CREATE TABLE sensor.sensor_log2 ( 
    `id` bigint NOT NULL COMMENT "sensor id",
    `name` varchar(26) NOT NULL COMMENT "sensor name", 
    `checked` boolean NOT NULL COMMENT "checked", 
    `sensor_type` varchar(26) NOT NULL COMMENT "sensor type",
    `data_y` long NULL COMMENT "sensor data" 
) 
ENGINE=OLAP 
DUPLICATE KEY (id) 
DISTRIBUTED BY HASH(`id`); 
```

**Routine Load ジョブ**

ロードジョブを提出し、Avroデータのロードが必要なフィールドを指定するために `jsonpaths` を使用します。ここで、ネストされたレコード `data_record` のサブフィールド `data_y` のために `jsonpath` を `"$.data.data_y"` と指定する必要があることに注意してください。

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

#### Avroスキーマに Union フィールドが含まれている

**データセットの準備**

Avroスキーマに Union フィールドが含まれており、Union フィールドをStarRocksにロードする必要があるとします。

- **Avroスキーマ**

    1. 次のAvroスキーマファイル `avro_schema3.avsc` を作成します。
    外部のAvroレコードには、順番に `id`、`name`、`checked`、`sensor_type`、および `data` の5つのフィールドが含まれています。そして、`data` フィールドは `null` とネストされたレコード `data_record` の2つの要素を含む Union 型であるとします。

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

Avroデータを準備し、Kafkaトピック `topic_3` に送信します。

**ターゲットのデータベースおよびテーブル**

Avroデータのフィールドに基づいて、StarRocksクラスターのターゲットデータベース `sensor` にテーブル `sensor_log3` を作成します。

外部レコードのフィールド `id`、`name`、`checked`、`sensor_type` だけでなく、Union 型フィールド `data` の要素 `data_record` のフィールド `data_y` もロードする必要があるとします。

```sql
CREATE TABLE sensor.sensor_log3 ( 
    `id` bigint NOT NULL COMMENT "sensor id",
    `name` varchar(26) NOT NULL COMMENT "sensor name", 
    `checked` boolean NOT NULL COMMENT "checked", 
    `sensor_type` varchar(26) NOT NULL COMMENT "sensor type",
    `data_y` long NULL COMMENT "sensor data" 
) 
ENGINE=OLAP 
DUPLICATE KEY (id) 
DISTRIBUTED BY HASH(`id`); 
```

**Routine Load ジョブ**

ロードジョブを提出し、Avroデータでロードするフィールドを指定するために `jsonpaths` を使用します。ここで、フィールド `data_y` に対して `jsonpath` を `"$.data.data_y"` と指定する必要があることに注意してください。

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

Union型フィールド `data` の値が `null` の場合、StarRocksテーブルのカラム `data_y` にロードされる値は `null` です。Union型フィールド `data` の値がデータレコードの場合、カラム `data_y` にロードされる値はLong型です。