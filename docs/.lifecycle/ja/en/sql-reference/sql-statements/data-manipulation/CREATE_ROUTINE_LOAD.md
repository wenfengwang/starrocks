---
displayed_sidebar: English
---

# CREATE ROUTINE LOAD

## 説明

Routine Loadは、Apache Kafka®からのメッセージを継続的に消費し、StarRocksにデータをロードすることができます。Routine Loadは、KafkaクラスターからCSV、JSON、Avro（v3.0.1以降でサポート）データを消費し、`plaintext`、`ssl`、`sasl_plaintext`、`sasl_ssl`などの複数のセキュリティプロトコルを介してKafkaにアクセスできます。

このトピックでは、CREATE ROUTINE LOADステートメントの構文、パラメーター、および例について説明します。

> **注記**
>
> - Routine Loadのアプリケーションシナリオ、原則、および基本操作については、[Apache Kafka®からのデータの継続的なロード](../../../loading/RoutineLoad.md)を参照してください。
> - StarRocksテーブルにデータをロードできるのは、そのStarRocksテーブルにINSERT権限を持つユーザーのみです。INSERT権限がない場合は、[GRANT](../account-management/GRANT.md)の指示に従って、StarRocksクラスタに接続するためのユーザーにINSERT権限を付与してください。

## 構文

```SQL
CREATE ROUTINE LOAD <database_name>.<job_name> ON <table_name>
[load_properties]
[job_properties]
FROM data_source
[data_source_properties]
```

## パラメーター

### `database_name`、`job_name`、`table_name`

`database_name`

オプション。StarRocksデータベースの名前。

`job_name`

必須。Routine Loadジョブの名前。テーブルは複数のRoutine Loadジョブからデータを受け取ることができます。複数のRoutine Loadジョブを区別するために、Kafkaトピック名やジョブ作成のおおよその時間など、識別可能な情報を使用して意味のあるRoutine Loadジョブ名を設定することを推奨します。Routine Loadジョブの名前は、同じデータベース内で一意でなければなりません。

`table_name`

必須。データがロードされるStarRocksテーブルの名前。

### `load_properties`

オプション。データのプロパティ。構文：

```SQL
[COLUMNS TERMINATED BY '<column_separator>'],
[ROWS TERMINATED BY '<row_separator>'],
[COLUMNS (<column1_name>[, <column2_name>, <column_assignment>, ... ])],
[WHERE <expr>],
[PARTITION (<partition1_name>[, <partition2_name>, ...])]
[TEMPORARY PARTITION (<temporary_partition1_name>[, <temporary_partition2_name>, ...])]
```

`COLUMNS TERMINATED BY`

CSV形式のデータの列区切り文字。デフォルトの列区切り文字は`\t`（タブ）です。例えば、`COLUMNS TERMINATED BY ","`を使用して列区切り文字をコンマに指定することができます。

> **注記**
>
> - ここで指定する列区切り文字が、取り込むデータの列区切り文字と同じであることを確認してください。
> - コンマ（,）、タブ、パイプ（|）など、長さが50バイトを超えないUTF-8文字列をテキスト区切り文字として使用できます。
> - Null値は`\N`を使用して表されます。例えば、データレコードが3つの列で構成され、データレコードは1列目と3列目にデータを保持し、2列目にはデータを保持していない場合、2列目に`\N`を使用してnull値を示す必要があります。つまり、レコードは`a,\N,b`としてコンパイルする必要があります。`a,,b`は、レコードの2列目に空文字列が含まれていることを示します。

`ROWS TERMINATED BY`

CSV形式のデータの行区切り文字。デフォルトの行区切り文字は`\n`です。

`COLUMNS`

ソースデータの列とStarRocksテーブルの列の間のマッピング。詳細については、このトピックの[列マッピング](#column-mapping)を参照してください。

- `column_name`: ソースデータの列が計算なしでStarRocksテーブルの列にマッピングできる場合、列名のみを指定します。これらの列はマップされた列と呼ばれます。
- `column_assignment`: ソースデータの列がStarRocksテーブルの列に直接マッピングできず、データロード前に関数を使用して列の値を計算する必要がある場合、`expr`で計算関数を指定する必要があります。これらの列は派生列と呼ばれます。
  StarRocksは最初にマップされた列を解析するため、派生列をマップされた列の後に配置することを推奨します。

`WHERE`

フィルター条件。フィルター条件を満たすデータのみがStarRocksにロードされます。例えば、`col1`の値が`100`より大きく、`col2`の値が`1000`に等しい行のみを取り込みたい場合、`WHERE col1 > 100 AND col2 = 1000`を使用できます。

> **注記**
>
> フィルター条件で指定される列は、ソース列または派生列です。

`PARTITION`

StarRocksテーブルがパーティションp0、p1、p2、p3に分散されており、StarRocksのp1、p2、p3にのみデータをロードし、p0に格納されるデータを除外したい場合、フィルター条件として`PARTITION(p1, p2, p3)`を指定できます。デフォルトでは、このパラメータを指定しない場合、データはすべてのパーティションにロードされます。例：

```SQL
PARTITION (p1, p2, p3)
```

`TEMPORARY PARTITION`

データをロードする[一時パーティション](../../../table_design/Temporary_partition.md)の名前。複数の一時パーティションを指定する場合は、コンマ（,）で区切る必要があります。

### `job_properties`

必須。ロードジョブのプロパティ。構文：

```SQL
PROPERTIES ("<key1>" = "<value1>"[, "<key2>" = "<value2>" ...])
```

| **プロパティ**              | **必須** | **説明**                                              |
| ------------------------- | ------------ | ------------------------------------------------------------ |
| desired_concurrent_number | いいえ           | 単一のRoutine Loadジョブの期待されるタスク並列処理数。デフォルト値は`3`です。実際のタスク並列処理数は、複数のパラメータの最小値によって決定されます：`min(alive_be_number, partition_number, desired_concurrent_number, max_routine_load_task_concurrent_num)`。<ul><li>`alive_be_number`：稼働中のBEノードの数。</li><li>`partition_number`：消費されるパーティションの数。</li><li>`desired_concurrent_number`：単一のRoutine Loadジョブの期待されるタスク並列処理数。デフォルト値は`3`です。</li><li>`max_routine_load_task_concurrent_num`：Routine Loadジョブのデフォルト最大タスク並列処理数は`5`です。[FE動的パラメータ](../../../administration/FE_configuration.md#configure-fe-dynamic-parameters)を参照してください。</li></ul>実際の最大タスク並列処理数は、稼働中のBEノードの数または消費されるパーティションの数のいずれか小さい方によって決定されます。|
| max_batch_interval        | いいえ           | タスクのスケジューリング間隔、つまりタスクが実行される頻度。単位は秒です。値の範囲は`5`～`60`です。デフォルト値は`10`です。`10`秒以上の値を設定することを推奨します。スケジューリングが10秒未満の場合、読み込み頻度が高すぎてタブレットバージョンが過剰に生成されます。|
| max_batch_rows            | いいえ           | このプロパティは、エラー検出のウィンドウを定義するためにのみ使用されます。ウィンドウは、単一のRoutine Loadタスクによって消費されるデータの行数です。値は`10 * max_batch_rows`です。デフォルト値は`10 * 200000 = 2000000`です。Routine Loadタスクは、エラー検出ウィンドウ内でエラーデータを検出します。エラーデータとは、StarRocksが解析できないデータ、例えば無効なJSON形式のデータを指します。 |

| max_error_number          | いいえ           | エラー検出ウィンドウ内で許容されるエラーデータ行の最大数です。エラーデータ行の数がこの値を超えると、ロードジョブは一時停止します。[SHOW ROUTINE LOAD](./SHOW_ROUTINE_LOAD.md) を実行し、`ErrorLogUrls`を使用してエラーログを表示します。その後、エラーログに従ってKafkaでエラーを修正できます。デフォルト値は `0` で、エラー行は許可されません。<br />**注**<br />エラーデータ行には、WHERE句によって除外されたデータ行は含まれません。 |
| strict_mode               | いいえ           | [strictモード](../../../loading/load_concept/strict_mode.md)を有効にするかどうかを指定します。有効な値: `true` と `false`。デフォルト値: `false`。strictモードが有効な場合、ロードされたデータの列の値が`NULL`であり、ターゲットテーブルがこの列の`NULL`値を許可しない場合、データ行はフィルターで除外されます。 |
| log_rejected_record_num | いいえ | ログに記録できる不適格データ行の最大数を指定します。このパラメーターはv3.1以降でサポートされています。有効な値: `0`、`-1`、および0以外の正の整数。デフォルト値: `0`。<ul><li>`0`はフィルターで除外されたデータ行がログに記録されないことを指定します。</li><li>`-1`はフィルターで除外されたすべてのデータ行がログに記録されることを指定します。</li><li>0以外の正の整数（例えば`n`）は、各BEでフィルターで除外された最大`n`データ行がログに記録されることを指定します。</li></ul> |
| timezone                  | いいえ           | ロードジョブで使用されるタイムゾーン。デフォルト値: `Asia/Shanghai`。このパラメーターの値は、strftime()、alignment_timestamp()、およびfrom_unixtime()などの関数によって返される結果に影響します。このパラメーターで指定されるタイムゾーンは、セッションレベルのタイムゾーンです。詳細については、[タイムゾーンの設定](../../../administration/timezone.md)を参照してください。 |
| merge_condition           | いいえ           | データを更新するかどうかを決定する条件として使用する列の名前を指定します。データは、この列にロードされるデータの値がこの列の現在の値以上の場合にのみ更新されます。詳細については、[ロードを通じてデータを変更する](../../../loading/Load_to_Primary_Key_tables.md)を参照してください。<br />**注**<br />条件付き更新はプライマリキーテーブルのみがサポートします。指定する列はプライマリキーカラムであってはなりません。|
| format                    | いいえ           | ロードするデータの形式。有効な値: `CSV`、`JSON`、および`Avro`（v3.0.1以降でサポート）。デフォルト値: `CSV`。 |
| trim_space                | いいえ           | データファイルがCSV形式の場合、列区切り文字の前後にあるスペースを削除するかどうかを指定します。タイプ: BOOLEAN。デフォルト値: `false`。<br />一部のデータベースでは、データをCSV形式のデータファイルとしてエクスポートする際に、列区切り文字の前後にスペースが追加されます。このようなスペースは、その位置に応じて先頭スペースまたは末尾スペースと呼ばれます。`trim_space`パラメータを設定することで、StarRocksはデータロード中にこれらの不要なスペースを削除することができます。<br />StarRocksは、`enclose`で指定された文字のペアで囲まれたフィールド内のスペース（先頭スペースと末尾スペースを含む）を削除しないことに注意してください。例えば、次のフィールド値ではパイプ（<code class="language-text">&#124;</code>）を列区切り文字として、ダブルクォーテーション（`"`）を`enclose`で指定された文字として使用します：<code class="language-text">&#124; "Love StarRocks" &#124;</code>。`trim_space`を`true`に設定すると、StarRocksは上記のフィールド値を<code class="language-text">&#124;"Love StarRocks"&#124;</code>として処理します。 |
| enclose                   | いいえ           | データファイルがCSV形式の場合、[RFC4180](https://www.rfc-editor.org/rfc/rfc4180)に従ってデータファイルのフィールド値を囲むために使用する文字を指定します。タイプ: 単一バイト文字。デフォルト値: `NONE`。最も一般的な文字はシングルクォート（`'`）とダブルクォート（`"`）です。<br />`enclose`で指定された文字で囲まれたすべての特殊文字（行区切り文字と列区切り文字を含む）は、通常の記号と見なされます。StarRocksはRFC4180を超えて、任意の単一バイト文字を`enclose`で指定された文字として指定できます。<br />フィールド値に`enclose`で指定された文字が含まれている場合、その`enclose`で指定された文字をエスケープするために同じ文字を使用できます。例えば、`enclose`を`"`に設定し、フィールド値が`a "quoted" c`である場合、データファイルには`"a ""quoted"" c"`としてフィールド値を入力できます。 |
| escape                    | いいえ           | 行区切り文字、列区切り文字、エスケープ文字、`enclose`で指定された文字など、さまざまな特殊文字をエスケープするために使用される文字を指定します。これらの文字はStarRocksによって通常の文字と見なされ、それらが存在するフィールド値の一部として解析されます。タイプ: 単一バイト文字。デフォルト値: `NONE`。最も一般的な文字はバックスラッシュ（`\`）で、SQLステートメントではダブルバックスラッシュ（`\\`）として記述する必要があります。<br />**注**<br />`escape`で指定された文字は、`enclose`で指定された文字のペアの内側と外側の両方に適用されます。<br />例えば、`enclose`を`"`に設定し、`escape`を`\`に設定すると、StarRocksは`"say \"Hello world\""`を`say "Hello world"`として解析します。列区切り文字がコンマ（`,`）である場合、`escape`を`\`に設定すると、StarRocksは`a, b\, c`を`a`と`b, c`の2つのフィールド値に解析します。 |
| strip_outer_array         | いいえ           | JSON形式のデータから最も外側の配列構造を取り除くかどうかを指定します。有効な値: `true`と`false`。デフォルト値: `false`。実際のビジネスシナリオでは、JSON形式のデータは、角括弧`[]`で示されるように最も外側の配列構造を持つ場合があります。この状況では、このパラメーターを`true`に設定してStarRocksが最も外側の角括弧`[]`を削除し、内部の各配列を個別のデータレコードとしてロードすることをお勧めします。このパラメーターを`false`に設定すると、StarRocksはJSON形式のデータ全体を1つの配列に解析し、その配列を1つのデータレコードとしてロードします。例として、JSON形式のデータ`[{"category" : 1, "author" : 2}, {"category" : 3, "author" : 4}]`を使用します。このパラメーターを`true`に設定すると、`{"category" : 1, "author" : 2}`と`{"category" : 3, "author" : 4}`は2つの別個のデータレコードとして解析され、2つのStarRocksデータ行にロードされます。 |
| jsonpaths                 | いいえ           | JSON形式のデータからロードするフィールドの名前です。このパラメーターの値は有効なJsonPath式です。詳細については、このトピックの[式を使用して値が生成されるStarRocksテーブルに含まれる派生列](#starrocks-table-contains-derived-columns-whose-values-are-generated-by-using-expressions)を参照してください。|
| json_root                 | いいえ           | JSON形式のデータのルート要素。StarRocksは`json_root`を通じてルートノードの要素を抽出し、解析します。デフォルトでは、このパラメーターの値は空で、すべてのJSON形式のデータが読み込まれることを意味します。詳細については、このトピック内の[JSON形式のデータのルート要素を指定する](#specify-the-root-element-of-the-json-formatted-data-to-be-loaded)を参照してください。|
| task_consume_second | いいえ | 指定されたRoutine Loadジョブ内の各Routine Loadタスクがデータを消費する最大時間。単位は秒です。[FE動的パラメーター](../../../administration/FE_configuration.md)の`routine_load_task_consume_second`（クラスター内のすべてのRoutine Loadジョブに適用される）とは異なり、このパラメーターは個々のRoutine Loadジョブに特有で、より柔軟です。このパラメーターはv3.1.0以降でサポートされています。<ul> <li>`task_consume_second`と`task_timeout_second`が設定されていない場合、StarRocksはFE動的パラメーターの`routine_load_task_consume_second`と`routine_load_task_timeout_second`を使用してロード動作を制御します。</li> <li>`task_consume_second`のみが設定されている場合、`task_timeout_second`のデフォルト値は`task_consume_second` * 4として計算されます。</li> <li>`task_timeout_second`のみが設定されている場合、`task_consume_second`のデフォルト値は`task_timeout_second` / 4として計算されます。</li> </ul> |
|task_timeout_second|いいえ|指定されたRoutine Loadジョブ内の各Routine Loadタスクのタイムアウト期間。単位は秒です。[FE動的パラメーター](../../../administration/FE_configuration.md)の`routine_load_task_timeout_second`（クラスター内のすべてのRoutine Loadジョブに適用される）とは異なり、このパラメーターは個々のRoutine Loadジョブに特有で、より柔軟です。このパラメーターはv3.1.0以降でサポートされています。<ul> <li>`task_consume_second`と`task_timeout_second`が設定されていない場合、StarRocksはFE動的パラメーターの`routine_load_task_consume_second`と`routine_load_task_timeout_second`を使用してロード動作を制御します。</li> <li>`task_timeout_second`のみが設定されている場合、`task_consume_second`のデフォルト値は`task_timeout_second` / 4として計算されます。</li> <li>`task_consume_second`のみが設定されている場合、`task_timeout_second`のデフォルト値は`task_consume_second` * 4として計算されます。</li> </ul>|

### `data_source`, `data_source_properties`

必須。データソースと関連プロパティ。

```sql
FROM <data_source>
 ("<key1>" = "<value1>"[, "<key2>" = "<value2>" ...])
```

`data_source`

必須。ロードしたいデータのソース。有効な値は`KAFKA`です。

`data_source_properties`

データソースのプロパティ。

| プロパティ          | 必須 | 説明                                                  |
| ----------------- | -------- | ------------------------------------------------------------ |
| kafka_broker_list | はい      | Kafkaブローカーの接続情報。形式は`<kafka_broker_ip>:<broker_port>`です。複数のブローカーはコンマ(,)で区切ります。Kafkaブローカーが使用するデフォルトポートは`9092`です。例：`"kafka_broker_list" = "xxx.xx.xxx.xx:9092,xxx.xx.xxx.xx:9092"`。 |
| kafka_topic       | はい      | 消費するKafkaトピック。Routine Loadジョブは一つのトピックからのメッセージのみを消費できます。 |
| kafka_partitions  | いいえ       | 消費するKafkaパーティション。例：`"kafka_partitions" = "0, 1, 2, 3"`。このプロパティが指定されていない場合、デフォルトで全てのパーティションが消費されます。 |
| kafka_offsets     | いいえ       | `kafka_partitions`で指定されたKafkaパーティション内でデータを消費する開始オフセット。このプロパティが指定されていない場合、Routine Loadジョブは`kafka_partitions`の最新のオフセットからデータを消費します。有効な値には以下があります：<ul><li>特定のオフセット：特定のオフセットからデータを消費します。</li><li>`OFFSET_BEGINNING`：可能な限り最初のオフセットからデータを消費します。</li><li>`OFFSET_END`：最新のオフセットからデータを消費します。</li></ul>複数の開始オフセットはコンマ(,)で区切られます。例：`"kafka_offsets" = "1000, OFFSET_BEGINNING, OFFSET_END, 2000"`。|
| property.kafka_default_offsets| いいえ| すべてのコンシューマーパーティションのデフォルト開始オフセット。このプロパティでサポートされる値は`kafka_offsets`プロパティの値と同じです。|
| confluent.schema.registry.url|いいえ |Avroスキーマが登録されているSchema RegistryのURL。StarRocksはこのURLを使用してAvroスキーマを取得します。形式は以下の通りです：<br />`confluent.schema.registry.url = http[s]://[<schema-registry-api-key>:<schema-registry-api-secret>@]<hostname or ip address>[:<port>]`|

#### その他のデータソース関連プロパティ

追加のデータソース（Kafka）関連プロパティを指定できます。これはKafkaコマンドラインの`--property`を使用するのと同等です。サポートされるプロパティの詳細については、[librdkafka設定プロパティ](https://github.com/confluentinc/librdkafka/blob/master/CONFIGURATION.md)のKafkaコンシューマクライアントのプロパティを参照してください。

> **注記**
>
> プロパティの値がファイル名の場合、ファイル名の前にキーワード`FILE:`を追加してください。ファイルの作成方法については、[CREATE FILE](../Administration/CREATE_FILE.md)を参照してください。

- **消費するすべてのパーティションのデフォルト初期オフセットを指定する**

```SQL
"property.kafka_default_offsets" = "OFFSET_BEGINNING"
```

- **Routine Loadジョブによって使用されるコンシューマグループのIDを指定する**

```SQL
"property.group.id" = "group_id_0"
```

`property.group.id`が指定されていない場合、StarRocksはルーチンロードジョブの名前に基づいてランダムな値を生成します。形式は`{job_name}_{random uuid}`、例えば`simple_job_0a64fe25-3983-44b2-a4d8-f52d3af4c3e8`です。

- **BEがKafkaにアクセスするために使用するセキュリティプロトコルと関連パラメータを指定する**

  セキュリティプロトコルは`plaintext`（デフォルト）、`ssl`、`sasl_plaintext`、`sasl_ssl`として指定できます。指定されたセキュリティプロトコルに応じて関連パラメータを設定する必要があります。

  セキュリティプロトコルが`sasl_plaintext`または`sasl_ssl`に設定されている場合、以下のSASL認証メカニズムがサポートされています：

  - PLAIN
  - SCRAM-SHA-256およびSCRAM-SHA-512
  - OAUTHBEARER

  - SSLセキュリティプロトコルを使用してKafkaにアクセスする：

    ```SQL
    -- セキュリティプロトコルをSSLとして指定します。
    "property.security.protocol" = "ssl"
    -- Kafkaブローカーの鍵を検証するためのCA証明書のファイルまたはディレクトリパス。
    -- Kafkaサーバーがクライアント認証を有効にしている場合、以下の3つのパラメータも必要です：
    -- 認証に使用されるクライアントの公開鍵のパス。
    "property.ssl.certificate.location" = "FILE:client.pem"
    -- 認証に使用されるクライアントの秘密鍵のパス。
    "property.ssl.key.location" = "FILE:client.key"
    -- クライアントの秘密鍵のパスワード。
    "property.ssl.key.password" = "xxxxxx"
    ```

  - SASL_PLAINTEXTセキュリティプロトコルとSASL/PLAIN認証メカニズムを使用してKafkaにアクセスする：

    ```SQL
    -- セキュリティプロトコルをSASL_PLAINTEXTとして指定します。
    "property.security.protocol" = "SASL_PLAINTEXT"
    -- SASLメカニズムを単純なユーザー名/パスワード認証メカニズムであるPLAINとして指定します。
    "property.sasl.mechanism" = "PLAIN" 
    -- SASLユーザー名
    "property.sasl.username" = "admin"
    -- SASLパスワード
    "property.sasl.password" = "xxxxxx"
    ```

### FEおよびBEの設定項目

Routine Loadに関連するFEおよびBEの設定項目については、[設定項目](../../../administration/FE_configuration.md)を参照してください。

## カラムマッピング

### CSV形式のデータのロードにカラムマッピングを設定する


CSV形式のデータの列がStarRocksテーブルの列に1対1で順番にマッピングできる場合、データとStarRocksテーブル間の列マッピングを設定する必要はありません。

CSV形式のデータの列がStarRocksテーブルの列に1対1で順番にマッピングできない場合、`columns`パラメータを使用してデータファイルとStarRocksテーブル間の列マッピングを設定する必要があります。これには以下の2つのユースケースが含まれます：

- **列数は同じですが、列の順序が異なります。また、データファイルからのデータは、対応するStarRocksテーブルの列にロードされる前に関数によって計算する必要はありません。**

  - `columns`パラメータでは、データファイルの列の配置と同じ順序でStarRocksテーブルの列の名前を指定する必要があります。

  - 例えば、StarRocksテーブルは`col1`、`col2`、`col3`の順に3つの列で構成され、データファイルも3つの列で構成されており、それらはStarRocksテーブルの`col3`、`col2`、`col1`の列に順番にマッピングできます。この場合、`"columns: col3, col2, col1"`と指定する必要があります。

- **列数と列の順序が異なります。また、データファイルからのデータは、対応するStarRocksテーブルの列にロードされる前に関数によって計算される必要があります。**

  `columns`パラメータでは、データファイルの列の配置と同じ順序でStarRocksテーブルの列の名前を指定し、データを計算するために使用する関数を指定する必要があります。以下に2つの例を示します：

  - StarRocksテーブルは`col1`、`col2`、`col3`の列で構成され、データファイルは4つの列で構成されており、最初の3つの列はStarRocksテーブルの`col1`、`col2`、`col3`の列に順番にマッピングできますが、4番目の列はStarRocksテーブルのいずれの列にもマッピングできません。この場合、データファイルの4番目の列に一時的な名前を指定する必要があり、その一時的な名前はStarRocksテーブルの列名とは異なるものでなければなりません。例えば、`"columns: col1, col2, col3, temp"`と指定することで、データファイルの4番目の列に一時的な名前`temp`を付けることができます。
  - StarRocksテーブルは`year`、`month`、`day`の列で構成され、データファイルは`yyyy-mm-dd hh:mm:ss`形式の日付と時刻の値を格納する1つの列のみで構成されています。この場合、`"columns: col, year = year(col), month=month(col), day=day(col)"`と指定することができ、`col`はデータファイル列の一時的な名前であり、関数`year = year(col)`、`month=month(col)`、`day=day(col)`はデータファイル列`col`からデータを抽出し、それをマッピングされたStarRocksテーブルの列にロードするために使用されます。例えば、`year = year(col)`はデータファイル列`col`から`yyyy`のデータを抽出し、それをStarRocksテーブルの`year`列にロードするために使用されます。

その他の例については、[列マッピングの設定](#configure-column-mapping)を参照してください。

### JSON形式またはAvro形式のデータの読み込みにおける列マッピングの設定

> **注記**
>
> v3.0.1以降、StarRocksはRoutine Loadを使用してAvroデータの読み込みをサポートしています。JSONまたはAvroデータを読み込む場合、列マッピングと変換の設定は同じです。したがって、このセクションではJSONデータを例にして設定を紹介します。

JSON形式のデータのキーがStarRocksテーブルの列名と同じ場合、シンプルモードを使用してJSON形式のデータを読み込むことができます。シンプルモードでは、`jsonpaths`パラメータを指定する必要はありません。このモードでは、JSON形式のデータは中括弧`{}`で示されるオブジェクトである必要があります。例えば`{"category": 1, "author": 2, "price": "3"}`のように、`category`、`author`、`price`はキー名であり、これらのキーはStarRocksテーブルの`category`、`author`、`price`の列に名前で1対1にマッピングできます。例については、[シンプルモード](#Column-names-of-the-target-table-are-consistent-with-JSON-keys)を参照してください。

JSON形式のデータのキー名がStarRocksテーブルの列名と異なる場合、マッチモードを使用してJSON形式のデータを読み込むことができます。マッチモードでは、`jsonpaths`と`COLUMNS`パラメータを使用して、JSON形式のデータとStarRocksテーブル間の列マッピングを指定する必要があります：

- `jsonpaths`パラメータでは、JSON形式のデータ内での配置と同じ順序でJSONキーを指定します。
- `COLUMNS`パラメータでは、JSONキーとStarRocksテーブルの列間のマッピングを指定します：
  - `COLUMNS`パラメータで指定された列名は、JSON形式のデータに1対1で順番にマッピングされます。
  - `COLUMNS`パラメータで指定された列名は、StarRocksテーブルの列に名前で1対1にマッピングされます。

例については、[式を使用して値が生成される派生列を含むStarRocksテーブル](#starrocks-table-contains-derived-columns-whose-values-are-generated-by-using-expressions)を参照してください。

## 例

### CSV形式のデータの読み込み

このセクションではCSV形式のデータを例に、さまざまなパラメータ設定と組み合わせを使用して、多様なロード要件に対応する方法について説明します。

**データセットの準備**

`ordertest1`という名前のKafkaトピックからCSV形式のデータをロードしたいとします。データセット内の各メッセージには、注文ID、支払日、顧客名、国籍、性別、価格の6つの列が含まれています。

```plaintext
2020050802,2020-05-08,Johann Georg Faust,Deutschland,male,895
2020050802,2020-05-08,Julien Sorel,France,male,893
2020050803,2020-05-08,Dorian Grey,UK,male,1262
2020050901,2020-05-09,Anna Karenina,Russia,female,175
2020051001,2020-05-10,Tess Durbeyfield,US,female,986
2020051101,2020-05-11,Edogawa Conan,Japan,male,8924
```

**テーブルの作成**

CSV形式のデータの列に基づいて、`example_db`データベース内に`example_tbl1`という名前のテーブルを作成します。

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

#### 指定されたパーティションとオフセットからデータの消費を開始する

Routine Loadジョブが指定されたパーティションとオフセットからデータの消費を開始する必要がある場合、`kafka_partitions`と`kafka_offsets`パラメータを設定する必要があります。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1",
    "kafka_partitions" ="0,1,2,3,4", -- 消費されるパーティション
    "kafka_offsets" = "1000, OFFSET_BEGINNING, OFFSET_END, 2000" -- 対応する初期オフセット
);
```

#### タスクの並列性を増やしてロードパフォーマンスを向上させる

ロードパフォーマンスを向上させ、累積的な消費を避けるために、`desired_concurrent_number` の値を増やしてタスクの並列性を高めることができます。これは、ルーチンロードジョブをできるだけ多くの並列タスクに分割することを可能にします。

> **注**
>
> ロードパフォーマンスを向上させる他の方法については、[ルーチンロードFAQ](../../../faq/loading/Routine_load_faq.md)を参照してください。

実際のタスク並列性は、以下の複数のパラメーターの中で最小の値によって決定されます：

```SQL
min(alive_be_number, partition_number, desired_concurrent_number, max_routine_load_task_concurrent_num)
```

> **注**
>
> 最大の実際のタスク並列性は、生存しているBEノードの数または消費されるパーティションの数のいずれかです。

したがって、生存しているBEノードの数と消費されるパーティションの数が、他の2つのパラメーター `max_routine_load_task_concurrent_num` と `desired_concurrent_number` の値よりも大きい場合、これら2つのパラメーターの値を増やして実際のタスク並列性を高めることができます。

消費されるパーティションの数が7、生存しているBEノードの数が5、`max_routine_load_task_concurrent_num` のデフォルト値が `5` であると仮定します。実際のタスク並列性を高めたい場合は、`desired_concurrent_number` を `5` に設定します（デフォルト値は `3`）。この場合、実際のタスク並列性 `min(5,7,5,5)` は `5` として設定されます。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
"desired_concurrent_number" = "5" -- desired_concurrent_number の値を5に設定
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### 列マッピングの設定

CSV形式のデータの列の順序がターゲットテーブルの列と一致しない場合、CSV形式のデータの5番目の列がターゲットテーブルにインポートされる必要がないと仮定して、`COLUMNS` パラメーターを使用してCSV形式のデータとターゲットテーブル間の列マッピングを指定する必要があります。

**ターゲットデータベースとテーブル**

CSV形式のデータの列に基づいて、ターゲットデータベース `example_db` にターゲットテーブル `example_tbl2` を作成します。このシナリオでは、性別を格納する5番目の列を除いて、CSV形式のデータの5つの列に対応する5つの列を作成する必要があります。

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

この例では、CSV形式のデータの5番目の列をターゲットテーブルにロードする必要がないため、`COLUMNS` で5番目の列を一時的に `temp_gender` として名付け、他の列は直接 `example_tbl2` テーブルにマッピングします。

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

特定の条件に合致するデータのみをロードしたい場合は、`WHERE` 句でフィルタ条件を設定できます。例えば、`price > 100.` のように。

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

#### 厳密モードを有効にしてNULL値を持つ行を除外する

`PROPERTIES` で `"strict_mode" = "true"` を設定することにより、ルーチンロードジョブを厳密モードにすることができます。ソース列に `NULL` 値があり、StarRocksテーブルの対象列がNULL値を許容しない場合、ソース列にNULL値を持つ行は除外されます。

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

#### エラー許容度の設定

ビジネスシナリオが不適格なデータに対して低い許容度を持つ場合は、`max_batch_rows` と `max_error_number` パラメータを設定して、エラー検出ウィンドウとエラーデータ行の最大数を設定する必要があります。エラー検出ウィンドウ内のエラーデータ行の数が `max_error_number` の値を超えた場合、ルーチンロードジョブは一時停止します。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
"max_batch_rows" = "100000", -- max_batch_rows の値に10を乗じたものがエラー検出ウィンドウです。
"max_error_number" = "100" -- エラー検出ウィンドウ内で許容されるエラーデータ行の最大数。
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### セキュリティプロトコルをSSLに指定し、関連するパラメータを設定する

BEがKafkaにアクセスする際に使用するセキュリティプロトコルをSSLとして指定する必要がある場合、`"property.security.protocol" = "ssl"` および関連するパラメータを設定する必要があります。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
-- セキュリティプロトコルをSSLとして指定する。
"property.security.protocol" = "ssl",
-- CA証明書の場所。
"property.ssl.ca.location" = "FILE:ca-cert",
-- Kafkaクライアントに認証が有効な場合、以下のプロパティを設定する必要があります：
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

#### trim_space、enclose、escapeの設定

`test_csv` という名前のKafkaトピックからCSV形式のデータをロードしたいとします。データセット内のすべてのメッセージには、注文ID、支払い日、顧客名、国籍、性別、価格の6つの列が含まれています。

```Plaintext
 "2020050802" , "2020-05-08" , "Johann Georg Faust" , "Deutschland" , "male" , "895"
 "2020050802" , "2020-05-08" , "Julien Sorel" , "France" , "male" , "893"
 "2020050803" , "2020-05-08" , "Dorian Grey\,Lord Henry" , "UK" , "male" , "1262"
 "2020050901" , "2020-05-09" , "Anna Karenina" , "Russia" , "female" , "175"
 "2020051001" , "2020-05-10" , "Tess Durbeyfield" , "US" , "female" , "986"
 "2020051101" , "2020-05-11" , "Edogawa Conan" , "Japan" , "male" , "8924"
```

Kafkaトピック `test_csv` からのすべてのデータを `example_tbl1` にロードし、列区切り文字の前後のスペースを削除し、`enclose` を `"` に、`escape` を `\` に設定する場合、次のコマンドを実行します。

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

#### StarRocksテーブルの列名とJSONキー名を一致させる

**データセットを準備する**

例えば、以下のJSON形式のデータがKafkaトピック`ordertest2`に存在します。

```SQL
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US", "pay_time": 1589191487, "price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK", "pay_time": 1589191487, "price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry", "country": "France", "pay_time": 1589191487, "price": 895}
```

> **注記**: 各JSONオブジェクトは1つのKafkaメッセージに含まれている必要があります。そうでない場合、JSON形式のデータの解析に失敗するエラーが発生します。

**ターゲットデータベースとテーブル**

StarRocksクラスタのターゲットデータベース`example_db`にテーブル`example_tbl3`を作成します。列名はJSON形式のデータのキー名と一致しています。

```SQL
CREATE TABLE example_db.example_tbl3 ( 
    commodity_id VARCHAR(26) NULL, 
    customer_name VARCHAR(26) NULL, 
    country VARCHAR(26) NULL, 
    pay_time BIGINT(20) NULL, 
    price DOUBLE SUM NULL COMMENT "Price") 
AGGREGATE KEY(commodity_id, customer_name, country, pay_time)
DISTRIBUTED BY HASH(commodity_id); 
```

**ルーチンロードジョブ**

ルーチンロードジョブではシンプルモードを使用できます。つまり、`jsonpaths`や`COLUMNS`パラメータを指定する必要はありません。StarRocksは、Kafkaクラスタのトピック`ordertest2`のJSON形式のデータのキーをターゲットテーブル`example_tbl3`の列名に基づいて抽出し、JSON形式のデータをターゲットテーブルにロードします。

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

> **注記**:
>
> - JSON形式のデータの最外層が配列構造である場合、`PROPERTIES`で`"strip_outer_array"="true"`を設定して最外層の配列構造を取り除く必要があります。また、`jsonpaths`を指定する必要がある場合、JSON形式のデータの最外層の配列構造が取り除かれるため、JSONデータ全体のルート要素はフラット化されたJSONオブジェクトになります。
> - `json_root`を使用してJSON形式のデータのルート要素を指定できます。

#### StarRocksテーブルには式を使用して値が生成される派生列が含まれている

**データセットを準備する**

例えば、以下のJSON形式のデータがKafkaクラスタのトピック`ordertest2`に存在します。

```SQL
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US", "pay_time": 1589191487, "price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK", "pay_time": 1589191487, "price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry", "country": "France", "pay_time": 1589191487, "price": 895}
```

**ターゲットデータベースとテーブル**

StarRocksクラスタのデータベース`example_db`に`example_tbl4`という名前のテーブルを作成します。列`pay_dt`は派生列で、JSON形式のデータ内のキー`pay_time`の値を計算して生成されます。

```SQL
CREATE TABLE example_db.example_tbl4 ( 
    `commodity_id` VARCHAR(26) NULL, 
    `customer_name` VARCHAR(26) NULL, 
    `country` VARCHAR(26) NULL,
    `pay_time` BIGINT(20) NULL,  
    `pay_dt` DATE NULL, 
    `price` DOUBLE SUM NULL) 
AGGREGATE KEY(`commodity_id`, `customer_name`, `country`, `pay_time`, `pay_dt`) 
DISTRIBUTED BY HASH(`commodity_id`); 
```

**ルーチンロードジョブ**

ルーチンロードジョブではマッチモードを使用する必要があります。つまり、`jsonpaths`と`COLUMNS`パラメータを指定する必要があります。

JSON形式のデータのキーを`jsonpaths`パラメータに指定し、順番に配置します。

JSON形式のデータのキー`pay_time`の値は、テーブル`example_tbl4`の列`pay_dt`に格納する前にDATE型に変換する必要があるため、`COLUMNS`で`pay_dt=from_unixtime(pay_time, '%Y%m%d')`として計算を指定する必要があります。JSONデータの他のキーの値は、テーブル`example_tbl4`に直接マッピングできます。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl4_ordertest2 ON example_tbl4
COLUMNS(commodity_id, customer_name, country, pay_time, pay_dt=from_unixtime(pay_time, '%Y%m%d'), price)
PROPERTIES
(
    "format" = "json",
    "jsonpaths" = "[\"$.commodity_id\", \"$.customer_name\", \"$.country\", \"$.pay_time\", \"$.price\"]"
)
FROM KAFKA
(
    "kafka_broker_list" = "<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest2"
);
```

> **注記**:
>
> - JSONデータの最外層が配列構造である場合、`PROPERTIES`で`"strip_outer_array"="true"`を設定して最外層の配列構造を取り除く必要があります。また、`jsonpaths`を指定する必要がある場合、JSONデータの最外層の配列構造が取り除かれるため、JSONデータ全体のルート要素はフラット化されたJSONオブジェクトになります。
> - `json_root`を使用してJSON形式のデータのルート要素を指定できます。

#### StarRocksテーブルにはCASE式を使用して値が生成される派生列が含まれています

**データセットを準備する**

例えば、以下のJSON形式のデータがKafkaトピック`topic-expr-test`に存在します。

```JSON
{"key1": 1, "key2": 21}
{"key1": 12, "key2": 22}
{"key1": 13, "key2": 23}
{"key1": 14, "key2": 24}
```

**ターゲットデータベースとテーブル**

StarRocksクラスタのデータベース`example_db`に`tbl_expr_test`という名前のテーブルを作成します。ターゲットテーブル`tbl_expr_test`には2つの列が含まれており、列`col2`の値はJSONデータでCASE式を使用して計算する必要があります。

```SQL
CREATE TABLE tbl_expr_test (
    col1 STRING, col2 STRING)
DISTRIBUTED BY HASH(col1);
```

**ルーチンロードジョブ**

ターゲットテーブルの列`col2`の値はCASE式を使用して生成されるため、ルーチンロードジョブの`COLUMNS`パラメータで対応する式を指定する必要があります。

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

StarRocksテーブルをクエリします。結果は、列`col2`の値がCASE式の出力であることを示しています。

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

`json_root`を使用してロードするJSON形式のデータのルート要素を指定する必要があり、値は有効なJsonPath式である必要があります。

**データセットを準備する**

例えば、以下のJSON形式のデータがKafkaクラスタのトピック`ordertest3`に存在します。読み込むJSON形式のデータのルート要素は`$.RECORDS`です。

```SQL
{"RECORDS":[{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875},{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895},{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}]}
```

**ターゲットデータベースおよびテーブル**

StarRocksクラスターの`example_db`データベースに`example_tbl3`という名前のテーブルを作成します。

```SQL
CREATE TABLE example_db.example_tbl3 ( 
    commodity_id varchar(26) NULL, 
    customer_name varchar(26) NULL, 
    country varchar(26) NULL, 
    pay_time bigint(20) NULL, 
    price double SUM NULL) 
AGGREGATE KEY(commodity_id, customer_name, country, pay_time) 
ENGINE=OLAP
DISTRIBUTED BY HASH(commodity_id); 
```

**ルーチンロードジョブ**

`PROPERTIES`に`"json_root" = "$.RECORDS"`を設定して、ロードするJSON形式のデータのルート要素を指定できます。また、ロードするJSON形式のデータが配列構造であるため、`"strip_outer_array" = "true"`を設定して最も外側の配列構造を剥がすようにも設定する必要があります。

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

### Avro形式のデータのロード

v3.0.1以降、StarRocksはルーチンロードを使用してAvroデータをロードすることをサポートしています。

#### Avroスキーマがシンプルな場合

Avroスキーマが比較的シンプルで、Avroデータのすべてのフィールドをロードする必要がある場合を想定します。

**データセットの準備**

- **Avroスキーマ**

    1. 次のAvroスキーマファイル`avro_schema1.avsc`を作成します。

        ```json
        {
            "type": "record",
            "name": "sensor_log",
            "fields" : [
                {"name": "id", "type": "long"},
                {"name": "name", "type": "string"},
                {"name": "checked", "type" : "boolean"},
                {"name": "data", "type": "double"},
                {"name": "sensor_type", "type": {"type": "enum", "name": "sensor_type_enum", "symbols" : ["TEMPERATURE", "HUMIDITY", "AIR_PRESSURE"]}}  
            ]
        }
        ```

    2. [スキーマレジストリ](https://docs.confluent.io/platform/current/schema-registry/index.html)にAvroスキーマを登録します。

- **Avroデータ**

Kafkaトピック`topic_1`にAvroデータを準備して送信します。

**ターゲットデータベースおよびテーブル**

Avroデータのフィールドに基づいて、StarRocksクラスターの`sensor`ターゲットデータベースに`sensor_log1`テーブルを作成します。テーブルのカラム名はAvroデータのフィールド名と一致する必要があります。AvroデータがStarRocksにロードされる際のデータタイプのマッピングについては、[データタイプのマッピング](#Data-types-mapping)を参照してください。

```SQL
CREATE TABLE sensor.sensor_log1 ( 
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

**ルーチンロードジョブ**

ルーチンロードジョブにはシンプルモードを使用できます。つまり、ルーチンロードジョブを作成する際に`jsonpaths`パラメータを指定する必要はありません。以下のステートメントを実行して、Kafkaトピック`topic_1`のAvroメッセージを消費し、データベース`sensor`のテーブル`sensor_log1`にデータをロードする`sensor_log_load_job1`という名前のルーチンロードジョブを送信します。

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
  "kafka_topic" = "topic_1",  
  "kafka_partitions" = "0,1,2,3,4,5",  
  "property.kafka_default_offsets" = "OFFSET_BEGINNING"  
);
```

#### Avroスキーマにネストされたレコード型フィールドが含まれる場合

Avroスキーマにネストされたレコード型フィールドが含まれ、そのネストされたレコード型フィールドのサブフィールドをStarRocksにロードする必要がある場合を想定します。

**データセットの準備**

- **Avroスキーマ**

    1. 次のAvroスキーマファイル`avro_schema2.avsc`を作成します。外側のAvroレコードには`id`、`name`、`checked`、`sensor_type`、および`data`のフィールドがあり、フィールド`data`にはネストされたレコード`data_record`があります。

        ```JSON
        {
            "type": "record",
            "name": "sensor_log",
            "fields" : [
                {"name": "id", "type": "long"},
                {"name": "name", "type": "string"},
                {"name": "checked", "type" : "boolean"},
                {"name": "sensor_type", "type": {"type": "enum", "name": "sensor_type_enum", "symbols" : ["TEMPERATURE", "HUMIDITY", "AIR_PRESSURE"]}},
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

Kafkaトピック`topic_2`にAvroデータを準備して送信します。

**ターゲットデータベースおよびテーブル**

Avroデータのフィールドに基づいて、StarRocksクラスターの`sensor`ターゲットデータベースに`sensor_log2`テーブルを作成します。

外部レコードのフィールド`id`、`name`、`checked`、および`sensor_type`をロードするだけでなく、ネストされたレコード`data_record`のサブフィールド`data_y`もロードする必要があるとします。

```sql
CREATE TABLE sensor.sensor_log2 ( 
    `id` bigint NOT NULL COMMENT "sensor id",
    `name` varchar(26) NOT NULL COMMENT "sensor name", 
    `checked` boolean NOT NULL COMMENT "checked", 
    `sensor_type` varchar(26) NOT NULL COMMENT "sensor type",
    `data_y` long NULL COMMENT "sensor data" 
) 
ENGINE=OLAP 
DUPLICATE KEY (`id`) 
DISTRIBUTED BY HASH(`id`); 
```

**ルーチンロードジョブ**

ロードジョブを送信し、`jsonpaths`を使用してロードする必要があるAvroデータのフィールドを指定します。ネストされたレコードのサブフィールド`data_y`については、その`jsonpath`を`"$.data.data_y"`として指定する必要があります。

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
  "kafka_topic" = "topic_2",  
  "kafka_partitions" = "0,1,2,3,4,5",  
  "property.kafka_default_offsets" = "OFFSET_BEGINNING"  
);
```

#### Avroスキーマにユニオンフィールドが含まれる場合

**データセットの準備**

Avroスキーマにユニオンフィールドが含まれ、そのユニオンフィールドをStarRocksにロードする必要がある場合を想定します。

- **Avroスキーマ**

    1. 次のAvroスキーマファイル`avro_schema3.avsc`を作成します。外側のAvroレコードには`id`、`name`、`checked`、`sensor_type`、および`data`のフィールドがあり、フィールド`data`はユニオン型で、`null`とネストされたレコード`data_record`の2つの要素が含まれます。

        ```JSON
        {
            "type": "record",
            "name": "sensor_log",
            "fields" : [
                {"name": "id", "type": "long"},
                {"name": "name", "type": "string"},
                {"name": "checked", "type" : "boolean"},
                {"name": "sensor_type", "type": {"type": "enum", "name": "sensor_type_enum", "symbols" : ["TEMPERATURE", "HUMIDITY", "AIR_PRESSURE"]}},
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
    2. Avro スキーマを [Schema Registry](https://docs.confluent.io/platform/current/schema-registry/index.html)に登録します。

- **Avro データ**

Avro データを準備し、Kafka トピック `topic_3` に送信します。

**ターゲットデータベースとテーブル**

Avro データのフィールドに基づいて、StarRocks クラスター内のターゲットデータベース `sensor` にテーブル `sensor_log3` を作成します。

外部レコードのフィールド `id`、`name`、`checked`、`sensor_type` をロードするだけでなく、ユニオン型フィールド `data` の要素 `data_record` 内のフィールド `data_y` もロードする必要があります。

```sql
CREATE TABLE sensor.sensor_log3 ( 
    `id` bigint NOT NULL COMMENT "sensor id",
    `name` varchar(26) NOT NULL COMMENT "sensor name", 
    `checked` boolean NOT NULL COMMENT "checked", 
    `sensor_type` varchar(26) NOT NULL COMMENT "sensor type",
    `data_y` long NULL COMMENT "sensor data" 
) 
ENGINE=OLAP 
DUPLICATE KEY (`id`) 
DISTRIBUTED BY HASH(`id`); 
```

**Routine Load ジョブ**

ロードジョブをサブミットし、`jsonpaths` を使用して Avro データ内でロードする必要があるフィールドを指定します。フィールド `data_y` については、その `jsonpath` を `"$.data.data_y"` として指定する必要があります。

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
  "kafka_topic" = "topic_3",  
  "kafka_partitions" = "0,1,2,3,4,5",  
  "property.kafka_default_offsets" = "OFFSET_BEGINNING"  
);
```

ユニオン型フィールド `data` の値が `null` の場合、StarRocks テーブルの `data_y` 列にロードされる値は `null` です。ユニオン型フィールド `data` の値がデータレコードである場合、`data_y` 列にロードされる値は Long 型になります。
