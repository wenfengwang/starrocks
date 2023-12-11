---
displayed_sidebar: "Japanese"
---

# ルーチンロードの作成

## 説明

ルーチンロードは、Apache Kafka®からメッセージを連続して消費し、StarRocksにデータをロードすることができます。ルーチンロードは、KafkaクラスタからCSV、JSON、およびAvro（v3.0.1以降でサポート）データを消費し、`plaintext`、`ssl`、`sasl_plaintext`、および`sasl_ssl`を含む複数のセキュリティプロトコルを使用してKafkaにアクセスできます。

このトピックでは、CREATE ROUTINE LOADステートメントの構文、パラメータ、および例について説明します。

> **注意**
>
> - ルーチンロードのアプリケーションシナリオ、原則、および基本的な操作については、[Apache Kafka®からデータを連続してロードする](../../../loading/RoutineLoad.md)を参照してください。
> - StarRocksテーブルにデータをロードするには、StarRocksテーブルにINSERT権限があるユーザーのみが可能です。INSERT 権限がない場合は、 [GRANT](../account-management/GRANT.md)でStarRocksクラスタに接続するユーザーにINSERT権限を付与するように指示してください。

## 構文

```SQL
CREATE ROUTINE LOAD <database_name>.<job_name> ON <table_name>
[load_properties]
[job_properties]
FROM data_source
[data_source_properties]
```

## パラメータ

### `database_name`、 `job_name`、 `table_name`

`database_name`

任意。StarRocksデータベースの名前です。

`job_name`

必須。ルーチンロードジョブの名前です。1つのテーブルは、複数のルーチンロードジョブからデータを受信することができます。複数のルーチンロードジョブを区別するために、識別可能な情報を使用して意味のあるルーチンロードジョブ名を設定することをお勧めします。たとえば、Kafkaトピック名とおおよそのジョブ作成時間を使用して、ルーチンロードジョブの名前を設定してください。ルーチンロードジョブの名前は、同じデータベース内で一意である必要があります。

`table_name`

必須。データをロードするStarRocksテーブルの名前です。

### `load_properties`

任意。データのプロパティです。構文は次のとおりです。

```SQL
[COLUMNS TERMINATED BY '<column_separator>'],
[ROWS TERMINATED BY '<row_separator>'],
[COLUMNS (<column1_name>[, <column2_name>, <column_assignment>, ...])],
[WHERE <expr>],
[PARTITION (<partition1_name>[, <partition2_name>, ...])]
[TEMPORARY PARTITION (<temporary_partition1_name>[, <temporary_partition2_name>, ...])]
```

`COLUMNS TERMINATED BY`

CSV形式のデータの列区切り記号です。デフォルトの列区切り記号は`\t`（タブ）です。たとえば、列区切り記号をカンマに指定するには`COLUMNS TERMINATED BY ","`と指定できます。

> **注意**
>
> - ここで指定した列区切り記号が取り込まれるデータの列区切り記号と同じであることを確認してください。
> - カンマ（,）、タブ、またはパイプ（|）など、長さが50バイトを超えないUTF-8文字列をテキスト区切り記号として使用できます。
> - NULL値は`\N`を使用して示します。たとえば、データレコードは3つの列で構成され、データレコードが1番目と3番目の列にデータを保持していますが、2番目の列にはデータを保持していない場合、2番目の列に`\N`を使用してNULL値を示す必要があります。これはレコードを`a,\N,b`としてコンパイルする必要があります。`a,,b` とすると、レコードの2番目の列が空の文字列を保持していることを示します。

`ROWS TERMINATED BY`

CSV形式のデータの行区切り記号です。デフォルトの行区切り記号は`\n`です。

`COLUMNS`

ソースデータの列とStarRocksテーブルの列のマッピングです。詳細については、このトピックの[カラムマッピング](#column-mapping)を参照してください。

- `column_name`：ソースデータの列が計算なしでStarRocksテーブルの列にマッピングできる場合、列名のみを指定する必要があります。これらの列はマップされた列と呼ばれます。
- `column_assignment`：ソースデータの列が直接StarRocksテーブルの列にマッピングできない場合、およびデータロード前に列の値を計算する必要がある場合は、`expr`で計算関数を指定する必要があります。これらの列は導出列と呼ばれます。
  StarRocksは最初にマップされた列を解析するため、マップされた列の後に導出列を配置することをお勧めします。

`WHERE`

フィルタ条件です。フィルタ条件を満たすデータのみをStarRocksにロードできます。たとえば、`col1`の値が`100`を超え、`col2`の値が`1000`に等しい行のみを取り込みたい場合は、`WHERE col1 > 100 and col2 = 1000`を使用できます。

> **注意**
>
> フィルタ条件で指定された列は、ソース列または導出列であることができます。

`PARTITION`

StarRocksテーブルがパーティションp0、p1、p2、およびp3に分散されており、StarRocksにデータをp1、p2、およびp3のみにロードし、p0に保存されるデータをフィルタリングしたい場合は、`PARTITION(p1, p2, p3)`と指定できます。デフォルトでは、このパラメータを指定しないと、すべてのパーティションにデータがロードされます。例：

```SQL
PARTITION (p1, p2, p3)
```

`TEMPORARY PARTITION`

データをロードする[一時パーティション](../../../table_design/Temporary_partition.md)の名前です。複数の一時パーティションを指定できますが、それらはコンマ（,）で区切る必要があります。

### `job_properties`

必須。ロードジョブのプロパティです。構文は次のとおりです。

```SQL
PROPERTIES ("<key1>" = "<value1>"[, "<key2>" = "<value2>" ...])
```

| **プロパティ**              | **必須** | **説明**                                           |
| ------------------------- | -------- | ------------------------------------------------ |
| desired_concurrent_number | いいえ    | 単一のルーチンロードジョブの期待されるタスク並列性。デフォルト値：`3`。実際のタスク並列性は、複数のパラメータの最小値によって決定されます：`min(alive_be_number, partition_number, desired_concurrent_number, max_routine_load_task_concurrent_num)`。<ul><li>`alive_be_number`：生きているBEノードの数。</li><li>`partition_number`：消費するパーティションの数。</li><li>`desired_concurrent_number`：単一のルーチンロードジョブの期待されるタスク並列性。デフォルト値：`3`。</li><li>`max_routine_load_task_concurrent_num`：ルーチンロードジョブのデフォルトの最大タスク並列性、つまり`5`。[FE動的パラメータ](../../../administration/Configuration.md#configure-fe-dynamic-parameters)を参照してください。</li></ul>最大の実際のタスク並列性は、生きているBEノードの数または消費するパーティションの数によって決定されます。 |
| max_batch_interval        | いいえ    | タスクのスケジューリング間隔、つまり、タスクの実行頻度です。単位：秒。値の範囲：`5` ～ `60`。デフォルト値：`10`。`10` より短いスケジュールの場合、過剰なロード頻度によって多数のタブレットバージョンが生成されるため、`10` より大きい値を設定することをお勧めします。 |
| max_batch_rows            | いいえ    | このプロパティはエラー検出のウィンドウを定義するためにのみ使用されます。ウィンドウは単一のルーチンロードタスクによって消費されるデータ行の数です。値は`10 * max_batch_rows`です。デフォルト値は`10 * 200000 = 200000`です。ルーチンロードタスクは、無効なJSONフォーマットのデータなど、StarRocksが解析できないエラーデータを検出します。 |
| max_error_number          | いいえ    | エラー検出ウィンドウ内で許可されるエラーデータ行の最大数。エラーデータ行の数がこの値を超えると、ロードジョブが一時停止します。 [SHOW ROUTINE LOAD](./SHOW_ROUTINE_LOAD.md)を実行し、`ErrorLogUrls`を使用してエラーログを表示することができます。その後、エラーログに従ってKafkaでエラーを修正することができます。デフォルト値は`0`で、エラーデータ行は許可されていません。<br />**注意** <br />エラーデータ行には、WHERE句でフィルタリングされたデータ行は含まれません。 |
| strict_mode               | いいえ    | [厳密モード](../../../loading/load_concept/strict_mode.md)を有効にするかどうかを指定します。有効な値は`true` と`false`です。デフォルト値：`false`。厳密モードが有効になっている場合、ロードデータの列の値が`NULL`であるが、対象のテーブルがこの列の`NULL`値を許可していない場合、データ行はフィルタリングされます。 |
| log_rejected_record_num | いいえ | ログに記録できる不適格なデータ行の最大数を指定します。このパラメータはv3.1からサポートされています。有効な値：`0`、`-1`、0以外の正の整数。デフォルト値：`0`。<ul><li>`0`の値はフィルタリングされたデータ行をログに記録しません。</li><li>`-1`の値は、すべてのフィルタリングされたデータ行をログに記録します。</li><li>`n` のような0以外の正の整数は、各BEに最大で `n` 個のフィルタリングされたデータ行を記録できます。</li></ul> |

| タイムゾーン                  | いいえ           | ロードジョブで使用されるタイムゾーン。デフォルト値: `Asia/Shanghai`。このパラメータの値は、strftime()、alignment_timestamp()、およびfrom_unixtime()などの関数によって返される結果に影響します。このパラメータで指定されたタイムゾーンはセッションレベルのタイムゾーンです。詳細については、[タイムゾーンの構成](../../../administration/timezone.md)を参照してください。 |
| merge_condition           | いいえ           | データの更新を決定する条件として使用する列の名前を指定します。この列にロードされるデータの値が、この列の現在の値以上の場合のみデータが更新されます。詳細については、[ロードを介したデータの変更](../../../loading/Load_to_Primary_Key_tables.md)を参照してください。<br />**注意**<br />プライマリキー・テーブルのみ条件付き更新をサポートしています。指定した列はプライマリキー列にできません。 |
| フォーマット                    | いいえ           | ロードされるデータのフォーマット。有効な値: `CSV`、`JSON`、および`Avro` (v3.0.1 以降サポート)。デフォルト値: `CSV`。 |
| trim_space                | いいえ           | CSV形式のデータファイルに含まれる列区切り記号の前後の空白を削除するかどうかを指定します。タイプ: BOOLEAN。デフォルト値: `false`。<br />一部のデータベースでは、CSV形式のデータファイルとしてデータをエクスポートする際に、列区切り記号にスペースが追加されることがあります。このようなスペースは、場所に応じて先行スペースまたは後続スペースと呼ばれます。`trim_space` パラメータを設定することで、StarRocks にそのような不要なスペースをデータロード時に削除させることができます。<br />なお、StarRocks は、`enclose`指定文字で囲まれたフィールド内のスペース（先行スペースおよび後続スペースを含む）を削除しません。たとえば、パイプ(<code class="language-text">&#124;</code>)を列区切り記号とし、二重引用符（`"`）を `enclose` 指定文字として指定した以下のフィールド値を使用する場合: <code class="language-text">&#124; "Love StarRocks" &#124;</code>。`trim_space` を `true` に設定した場合、StarRocks は先行フィールド値を`<code class="language-text">&#124;"Love StarRocks"&#124;</code>` として処理します。 |
| enclose                   | いいえ           | CSV形式のデータファイルからフィールド値を囲むのに使用される文字を指定します。タイプ: 単バイト文字。デフォルト値: `NONE`。最も一般的な文字は単一引用符（`'`）と二重引用符（`"`）です。<br />`enclose`指定文字で囲まれたすべての特殊文字（行区切り記号および列区切り記号も含む）は通常のシンボルと見なされます。StarRocks は RFC4180 よりもさらに多くのことができ、`enclose`指定文字として任意の単バイト文字を指定することができます。<br />フィールド値が `enclose`指定文字を含む場合、同じ文字を使用してその `enclose`指定文字をエスケープできます。たとえば、`enclose` を `"` に設定し、フィールド値が `a "quoted" c` の場合、フィールド値をデータファイルに`"a ""quoted"" c"` と入力できます。 |
| escape                    | いいえ           | 行区切り記号、列区切り記号、エスケープ文字、および `enclose`指定文字などのさまざまな特殊文字をエスケープするために使用される文字を指定します。タイプ: 単バイト文字。デフォルト値: `NONE`。最も一般的な文字はスラッシュ（`\`）であり、SQL ステートメントでは二重スラッシュ（`\\`）として書かなければなりません。<br />**注意**<br />`escape` で指定された文字は、`enclose`指定文字の内外の両方に適用されます。<br />次の例をいくつか挙げます:<ul><li>`enclose` を `"` に設定し、`escape` を `\` に設定した場合、StarRocks は`"say \"Hello world\""` を `say "Hello world"` に解析します。</li><li>列区切り記号がカンマ(,)であるとします。`escape` を `\` に設定した場合、StarRocks は `a, b\, c` を `a` と `b, c` の二つの別々のフィールド値に解析します。</li></ul> |
| strip_outer_array         | いいえ           | JSON形式のデータの最外部配列構造を削除するかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `false`。実際のビジネスシナリオでは、JSON形式のデータには外部の角かっこのペア `[]` が示す最外部配列構造が含まれることがあります。このような状況では、このパラメータを `true` に設定することをお勧めします。これにより、StarRocks は外部の角かっこ `[]` を除去し、各内部配列を個別のデータレコードとしてロードします。`false` に設定した場合、StarRocks はJSON形式全体を1つの配列と解析し、その配列を1つのデータレコードとしてロードします。 `{"category" : 1, "author" : 2}` および `{"category" : 3, "author" : 4}` を使用した JSON形式のデータを例に取り上げます。このパラメータを `true` に設定した場合、それぞれのデータが2つの別々のデータレコードとして解析され、2つの StarRocks データ行にロードされます。 |
| jsonpaths                 | いいえ           | JSON形式のデータからロードしたいフィールドの名前。このパラメータの値は有効なJsonPath式です。詳細については、このトピックの[StarRocksテーブルに、式を使用して生成した値を持つ派生列が含まれています](#starrocks-table-contains-derived-columns-whose-values-are-generated-by-using-expressions)を参照してください。 |
| json_root                 | いいえ           | ロードする JSON形式データのルート要素。StarRocks はパーシングするためにルートノードの要素を`json_root`を介して抽出します。デフォルトでは、このパラメータの値は空であり、すべての JSON形式データがロードされることを示します。詳細については、このトピックの[ロードするJSON形式データのルート要素を指定する](#specify-the-root-element-of-the-json-formatted-data-to-be-loaded)を参照してください。 |
| task_consume_second | いいえ | 指定されたルーチンロードジョブ内の各ルーチンロードタスクがデータを消費する最大時間。単位: 秒。クラスタ内のすべてのルーチンロードジョブに適用される[FEDynamicParameters](../../../administration/Configuration.md)`routine_load_task_consume_second`（すべてのルーチンロードジョブに適用される）とは異なり、このパラメータは個々のルーチンロードジョブに固有であり、より柔軟です。このパラメータは、v3.1.0 以降でサポートされています。<ul> <li>`task_consume_second` と `task_timeout_second` が構成されていない場合、StarRocks はロード動作を制御するために、FEDynamicParameters `routine_load_task_consume_second` および `routine_load_task_timeout_second` を使用します。</li> <li>`task_consume_second` のみが構成されている場合、`task_timeout_second` のデフォルト値は `task_consume_second` * 4 として計算されます。</li> <li>`task_timeout_second` のみが構成されている場合、`task_consume_second` のデフォルト値は `task_timeout_second`/4 として計算されます。</li> </ul> |
| task_timeout_second|いいえ|指定されたルーチンロードジョブ内の各ルーチンロードタスクのタイムアウト継続時間。単位: 秒。クラスタ内のすべてのルーチンロードジョブに適用される[FEDynamicParameter](../../../administration/Configuration.md)`routine_load_task_timeout_second`（すべてのルーチンロードジョブに適用される）とは異なり、このパラメータは個々のルーチンロードジョブに固有であり、より柔軟です。このパラメータは、v3.1.0 以降でサポートされています。<ul> <li>`task_consume_second` と `task_timeout_second` が構成されていない場合、StarRocks はロード動作を制御するために、FEDynamicParameter `routine_load_task_consume_second` および `routine_load_task_timeout_second` を使用します。</li> <li>`task_timeout_second` のみが構成されている場合、`task_consume_second` のデフォルト値は `task_timeout_second`/4 として計算されます。</li> <li>`task_consume_second` のみが構成されている場合、`task_timeout_second` のデフォルト値は `task_consume_second` * 4 として計算されます。</li> </ul> |

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

| プロパティ          | 必須 | 説明                                                  |
| ----------------- | -------- | ------------------------------------------------------------ |
| kafka_broker_list | はい      | Kafka のブローカー接続情報。フォーマットは `<kafka_broker_ip>:<broker_port>` です。複数のブローカーはカンマ (,) で区切ります。Kafka ブローカーが使用するデフォルトのポートは `9092` です。例: `"kafka_broker_list" = ""xxx.xx.xxx.xx:9092,xxx.xx.xxx.xx:9092"`. |
| kafka_topic       | はい      | 消費する Kafka トピック。ルーチンロードジョブで1つのトピックからメッセージのみを消費できます。 |
| kafka_partitions  | いいえ   | 消費するKafkaパーティション、たとえば、`"kafka_partitions" = "0, 1, 2, 3"`。このプロパティが指定されていない場合、デフォルトですべてのパーティションが消費されます。|
| kafka_offsets     | いいえ   | `kafka_partitions`で指定したKafkaパーティションでデータを消費する起点オフセット。このプロパティが指定されていない場合、Routine Loadジョブは`kafka_partitions`の最新のオフセットからデータを消費します。有効な値:<ul><li>特定のオフセット：特定のオフセットからデータを消費します。</li><li>`OFFSET_BEGINNING`：可能な最も初期のオフセットからデータを消費します。</li><li>`OFFSET_END`：最新のオフセットからデータを消費します。</li></ul> 複数の起点オフセットはカンマ（,）で区切られます。たとえば、`"kafka_offsets" = "1000, OFFSET_BEGINNING, OFFSET_END, 2000"`。|
| property.kafka_default_offsets| いいえ| すべての消費者パーティションのデフォルトの起点オフセット。このプロパティのサポートされている値は`kafka_offsets`プロパティと同じです。|
| confluent.schema.registry.url|いいえ | Avroスキーマが登録されているSchema RegistryのURL。StarRocksはこのURLを使用してAvroスキーマを取得します。形式は次のとおりです:<br />`confluent.schema.registry.url = http[s]://[<schema-registry-api-key>:<schema-registry-api-secret>@]<hostname or ip address>[:<port>]`|

#### その他のデータソース関連のプロパティ

Kafkaに関連する追加のデータソース（Kafka）関連プロパティを指定できます。これはKafkaコマンドラインの`--property`を使用するのと同等です。サポートされているプロパティについては、[librdkafka configuration properties](https://github.com/confluentinc/librdkafka/blob/master/CONFIGURATION.md)でKafkaコンシューマクライアントのプロパティを参照してください。

> **注意**
>
> プロパティの値がファイル名の場合は、ファイル名の前にキーワード`FILE:`を追加します。ファイルの作成方法については、[CREATE FILE](../Administration/CREATE_FILE.md)を参照してください。

- **消費されるすべてのパーティションのデフォルトの初期オフセットを指定する**

```SQL
"property.kafka_default_offsets" = "OFFSET_BEGINNING"
```

- **Routine Loadジョブで使用されるコンシューマーグループのIDを指定する**

```SQL
"property.group.id" = "group_id_0"
```

`property.group.id`が指定されていない場合、StarRocksはRoutine Loadジョブの名前に基づいてランダムな値を生成し、`{job_name}_{random uuid}`形式で生成します。たとえば、`simple_job_0a64fe25-3983-44b2-a4d8-f52d3af4c3e8`のようになります。

- **BEがKafkaにアクセスするために使用するセキュリティプロトコルと関連するパラメーターを指定する**

  セキュリティプロトコルは`plaintext`（デフォルト）、`ssl`、`sasl_plaintext`、または`sasl_ssl`として指定できます。そして、指定したセキュリティプロトコルに応じて関連するパラメーターを構成する必要があります。

  セキュリティプロトコルを`ssl`として指定してKafkaにアクセスする場合:

    ```SQL
    -- セキュリティプロトコルをSSLとして指定します。
    "property.security.protocol" = "ssl"
    -- kafkaブローカーのキーを検証するためのCA証明書のファイルまたはディレクトリのパス
    -- Kafkaサーバーがクライアント認証を有効にしている場合、次の3つのパラメーターも必要です:
    -- 認証に使用されるクライアントの公開鍵のパス
    "property.ssl.certificate.location" = "FILE:client.pem"
    -- 認証に使用されるクライアントの秘密鍵のパス
    "property.ssl.key.location" = "FILE:client.key"
    -- クライアントの秘密鍵のパスワード
    "property.ssl.key.password" = "xxxxxx"
    ```

  `sasl_plaintext`または`sasl_ssl`を使用してKafkaにアクセスする場合:

    ```SQL
    -- セキュリティプロトコルをSASL_PLAINTEXTとして指定します
    "property.security.protocol" = "SASL_PLAINTEXT"
    -- PLAINをSASLメカニズムとして指定します。これはシンプルなユーザー名/パスワード認証メカニズムです
    "property.sasl.mechanism" = "PLAIN" 
    -- SASLユーザー名
    "property.sasl.username" = "admin"
    -- SASLパスワード
    "property.sasl.password" = "xxxxxx"
    ```

### FEおよびBE構成項目

Routine Loadに関連するFEおよびBE構成項目については、[構成項目](../../../administration/Configuration.md)を参照してください。

## カラムのマッピング

### CSV形式のデータの読み込みのためのカラムのマッピングを構成する

CSV形式のデータのカラムがStarRocksテーブルのカラムと1対1の順序でマッピングできる場合、データとStarRocksテーブルのカラム間のカラムマッピングを構成する必要はありません。

CSV形式のデータのカラムがStarRocksテーブルのカラムと1対1の順序でマッピングできない場合は、`columns`パラメータを使用してデータファイルとStarRocksテーブルのカラム間のカラムマッピングを構成する必要があります。次の2つのユースケースに該当します:

- **同じ数のカラムがあり、異なるカラムシーケンス。また、データはマッチングStarRocksテーブルのカラムにロードされる前に関数によって計算する必要がありません。**

  - `columns`パラメータでは、データファイルのカラムが配置されている順序と同じになるように、StarRocksテーブルのカラムの名前を指定する必要があります。

  - たとえば、StarRocksテーブルは`col1`、`col2`、`col3`の3つのカラムで構成されており、データファイルも3つのカラムで構成されており、これらのカラムをStarRocksテーブルのカラム`col3`、`col2`、`col1`に対応させる必要がある場合は、`"columns: col3, col2, col1"`を指定する必要があります。

- **異なる数のカラムや異なるカラムシーケンス。また、データはマッチングStarRocksテーブルのカラムにロードされる前に関数によって計算する必要があります。**

  データファイルのカラムが配置されている順序と同じになるように、`columns`パラメータではStarRocksテーブルのカラムの名前を指定し、データを計算するために使用する関数を指定する必要があります。2つの例は次のとおりです:

  - StarRocksテーブルは`col1`、`col2`、`col3`の3つのカラムで構成されており、データファイルは4つのカラムで構成されており、最初の3つのカラムをStarRocksテーブルのカラム`col1`、`col2`、`col3`に対応させることができ、4番目のカラムはどのStarRocksテーブルのカラムにも対応させることができない場合は、データファイルの4番目のカラムの名前を一時的に指定する必要があります。また、一時的な名前はStarRocksテーブルの任意のカラム名とは異なる必要があります。たとえば、`"columns: col1, col2, col3, temp"`を指定する必要があります。

  - StarRocksテーブルは`year`、`month`、`day`の3つのカラムで構成されており、データファイルは`yyyy-mm-dd hh:mm:ss`形式の日付と時刻値を含む1つのカラムのみで構成されています。この場合、`"columns: col, year = year(col), month=month(col), day=day(col)"`を指定します。ここで、`col`はデータファイルのカラムの一時的な名前であり、`year = year(col)`、`month=month(col)`、`day=day(col)`の関数は、データファイルのカラム`col`からデータを抽出し、マッピングされるStarRocksテーブルのカラムにデータをロードするために使用されます。たとえば、`year = year(col)`は、データファイルのカラム`col`から`yyyy`データを抽出し、StarRocksテーブルのカラム`year`にデータをロードするために使用されます。

その他の例については、[カラムのマッピングを構成する](#configure-column-mapping)を参照してください。

### JSON形式またはAvro形式のデータの読み込みのためのカラムのマッピングを構成する

>**注意**
>
>v3.0.1以降、StarRocksはRoutine Loadを使用してAvroデータを読み込むことができます。JSONまたはAvroデータを読み込む際は、カラムのマッピングと変換の構成が同じです。したがって、このセクションでは、カラムのマッピングと変換の構成について説明するためにJSONデータを使用します。
JSON形式のデータのキーがStarRocksテーブルの列とは異なる場合、一致モードを使用してJSON形式のデータを読み込むことができます。一致モードでは、`jsonpaths`および`COLUMNS`パラメータを使用して、JSON形式のデータとStarRocksテーブルの列の間の列マッピングを指定する必要があります。

- `jsonpaths`パラメータでは、JSON形式のデータ内で配置されている順序通りにJSONのキーを指定します。
- `COLUMNS`パラメータでは、JSONのキーとStarRocksテーブルの列とのマッピングを指定します:
  - `COLUMNS`パラメータで指定された列名は、JSON形式のデータと一対一にマッピングされます。
  - `COLUMNS`パラメータで指定された列名は、StarRocksテーブルの列と名前で一対一にマッピングされます。

例については、[StarRocks table contains derived columns whose values are generated by using expressions](#starrocks-table-contains-derived-columns-whose-values-are-generated-by-using-expressions)を参照してください。

## 例

### CSV形式のデータを読み込む

このセクションでは、さまざまなパラメータ設定と組み合わせを使用して、多様な読み込み要件を満たす方法を説明するために、CSV形式のデータを使用しています。

**データセットを準備する**

データセットからCSV形式のデータを`ordertest1`というKafkaトピックから読み込みたいとします。データセットのメッセージごとには、注文ID、支払日、顧客名、国籍、性別、価格の6つの列が含まれています。

```plaintext
2020050802,2020-05-08,Johann Georg Faust,Deutschland,male,895
2020050802,2020-05-08,Julien Sorel,France,male,893
2020050803,2020-05-08,Dorian Grey,UK,male,1262
2020050901,2020-05-09,Anna Karenina,Russia,female,175
2020051001,2020-05-10,Tess Durbeyfield,US,female,986
2020051101,2020-05-11,Edogawa Conan,japan,male,8924
```

**テーブルを作成する**

CSV形式のデータの列に従って、データベース`example_db`に`example_tbl1`という名前のテーブルを作成します。

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

####指定したパーティションとオフセットからデータを消費する

Routine Loadジョブが特定のパーティションとオフセットからデータを消費する必要がある場合、`kafka_partitions`および`kafka_offsets`パラメータを構成する必要があります。

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

#### タスク並列性を増やして読み込みパフォーマンスを向上させる

読み込みパフォーマンスを向上させ、累積消費を避けるために、Routine Loadジョブを作成する際に`desired_concurrent_number`の値を増やすことでタスク並列性を増やすことができます。タスク並列性により、1つのRoutine Loadジョブを可能な限り多くの並列タスクに分割することができます。

> **注意**
>
> 読み込みパフォーマンスを向上させるためのさらなる方法については、[Routine Load FAQ](../../../faq/loading/Routine_load_faq.md)を参照してください。

実際のタスク並列性は、次の複数のパラメータのうち最小の値によって決定されます。
```SQL
min(alive_be_number, partition_number, desired_concurrent_number, max_routine_load_task_concurrent_num)
```

> **注意**
>
> 実際のタスク並列性の最大値は、生きているBEノードの数または消費するパーティションの数のいずれかです。

したがって、生きているBEノードの数と消費するパーティションの数が他の2つのパラメータ`max_routine_load_task_concurrent_num`および`desired_concurrent_number`の値よりも大きい場合、実際のタスク並列性を増やすために他の2つのパラメータの値を増やすことができます。

消費するパーティションの数が7で、生きているBEノードの数が5であり、`max_routine_load_task_concurrent_num`がデフォルト値`5`の場合、実際のタスク並列性を増やしたい場合は、`desired_concurrent_number`を`5`（デフォルト値は`3`）に設定できます。この場合、実際のタスク並列性`min(5,7,5,5)`は`5`に構成されます。

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

#### 列マッピングを構成する

CSV形式のデータの列のシーケンスが対象テーブルの列と一貫していない場合、CSV形式のデータと対象テーブルの列の間で列マッピングを指定する必要があります。CSV形式のデータと対象テーブルの間で列マッピングを指定するには、`COLUMNS`パラメータを使用します。この例では、CSV形式のデータの5番目の列がターゲットテーブルにインポートされる必要がないと仮定し、CSV形式のデータと対象テーブル`example_tbl2`の間で列マッピングを指定します。


**ターゲットデータベースとテーブル**

CSV形式のデータの列に応じて、ターゲットデータベース`example_db`に`example_tbl2`という名前のテーブルを作成します。この場合、CSV形式のデータの5つの列に対応する5つの列を作成し、性別を格納する5番目の列は除外します。

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

**Routine Loadジョブ**

この例では、CSV形式のデータの5番目の列がターゲットテーブルにロードされる必要がないため、5番目の列名を一時的に`temp_gender`として`COLUMNS`に指定し、他の列は直接テーブル`example_tbl2`にマッピングします。

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

特定の条件を満たすデータのみを読み込みたい場合は、`WHERE`句でフィルタ条件を設定することができます。たとえば、`price > 100`とします。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl2_ordertest1 ON example_tbl2
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price),
WHERE price > 100 -- フィルタ条件を設定します
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### NULL値を含む行をフィルタリングするために厳密モードを有効にする

`PROPERTIES`で`"strict_mode" = "true"`を設定することで、Routine Loadジョブが厳密モードで実行されるようにすることができます。ソースの列に`NULL`値がある場合、目的のStarRocksテーブルの列が`NULL`値を許可しない場合、ソース列に`NULL`値が含まれる行はフィルタリングされます。

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
      "kafka_broker_list"="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
      "kafka_topic"="ordertest1"
    );

#### エラー許容度の設定

ビジネスシナリオで未加工データに対する許容度が低い場合、パラメータ `max_batch_rows` および `max_error_number` を構成してエラー検出ウィンドウとエラー行の最大数を設定する必要があります。エラー検出ウィンドウ内のエラー行数が `max_error_number` の値を超えると、Routine Load ジョブが一時停止します。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
"max_batch_rows" = "100000",-- max_batch_rows の値に 10 を掛けたものがエラー検出ウィンドウに相当します。
"max_error_number" = "100" -- エラー検出ウィンドウ内で許容されるエラー行数の最大数
)
FROM KAFKA
(
    "kafka_broker_list"="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic"="ordertest1"
);
```

#### SSL をセキュリティプロトコルとして指定し、関連パラメータを構成する

BE が Kafka にアクセスする際に SSL をセキュリティプロトコルとして指定する必要がある場合は、`"property.security.protocol" = "ssl"` および関連パラメータを構成する必要があります。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
-- SSL をセキュリティプロトコルとして指定する。
"property.security.protocol" = "ssl",
-- CA 証明書の場所。
"property.ssl.ca.location" = "FILE:ca-cert",
-- Kafka クライアントの認証が有効になっている場合、次のプロパティを構成する必要があります:
-- Kafka クライアントの公開キーの場所。
"property.ssl.certificate.location" = "FILE:client.pem",
-- Kafka クライアントのプライベートキーの場所。
"property.ssl.key.location" = "FILE:client.key",
-- Kafka クライアントのプライベートキーのパスワード。
"property.ssl.key.password" = "abcdefg"
)
FROM KAFKA
(
    "kafka_broker_list"="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic"="ordertest1"
);
```

#### trim_space、enclose、および escape を設定する

Kafka トピック `test_csv` から CSV 形式のデータをロードしようと思っています。データセットのすべてのメッセージには、注文 ID、支払日、顧客名、国籍、性別、価格の 6 つの列が含まれます。

```Plaintext
 "2020050802" , "2020-05-08" , "Johann Georg Faust" , "Deutschland" , "male" , "895"
 "2020050802" , "2020-05-08" , "Julien Sorel" , "France" , "male" , "893"
 "2020050803" , "2020-05-08" , "Dorian Grey\,Lord Henry" , "UK" , "male" , "1262"
 "2020050901" , "2020-05-09" , "Anna Karenina" , "Russia" , "female" , "175"
 "2020051001" , "2020-05-10" , "Tess Durbeyfield" , "US" , "female" , "986"
 "2020051101" , "2020-05-11" , "Edogawa Conan" , "japan" , "male" , "8924"
```

Kafka トピック `test_csv` のすべてのデータを `example_tbl1` にロードし、列区切り記号の前後の空白を削除し、`enclose` を `"`、`escape` を `\` に設定する場合は、次のコマンドを実行します。

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
    "kafka_broker_list"="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic"="test_csv",
    "property.kafka_default_offsets"="OFFSET_BEGINNING"
);
```

### JSON 形式のデータをロードする

#### StarRocks テーブル列名が JSON キー名と一致する

**データセットの準備**

たとえば、Kafka トピック `ordertest2` に次の JSON 形式のデータが存在するとします。

```SQL
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}
```

> **注** 各 JSON オブジェクトは 1 つの Kafka メッセージに存在する必要があります。さもないと、JSON 形式のデータの解析に失敗するエラーが発生します。

**ターゲットデータベースとテーブル**

StarRocks クラスタのターゲットデータベース `example_db` に `example_tbl3` という名前のテーブルを作成します。列名は JSON 形式のデータのキー名と一致しています。

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

ターゲットテーブル `example_tbl3` の列名に従って、`PROPERTIES` 内に `format` = `json` を指定せずに Routine Load ジョブのシンプルモードを使用できます。この場合、`ordertest2` トピック内の JSON 形式のデータのキーが抽出され、対象のテーブルにロードされます。

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

> **注**
>
> - JSON 形式のデータの最外層が配列構造の場合、`PROPERTIES` 内で `"strip_outer_array"="true"` を設定して最外層の配列構造を取り除く必要があります。また、`jsonpaths` を指定する必要がある場合、JSON 形式のデータの最外層の配列構造が取り除かれるため、JSON 形式のデータ全体のルート要素はフラット化された JSON オブジェクトになります。
> - `json_root` を使用して JSON 形式のデータのルート要素を指定できます。

#### StarRocks テーブルに、式を使用して値が生成される派生列が含まれる

**データセットの準備**

たとえば、Kafka クラスタのトピック `ordertest2` に次の JSON 形式のデータが存在するとします。

```SQL
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}
```

**ターゲットデータベースとテーブル**

StarRocks クラスタのターゲットデータベース `example_db` に `example_tbl4` という名前のテーブルを作成します。ここでは `pay_time` の値を計算して生成する `pay_dt` という派生列が含まれています。

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
**Routine Load job**

マッチングモードを使用して、Routine Load ジョブを実行できます。つまり、Routine Load ジョブを作成する際に `jsonpaths` および `COLUMNS` パラメータを指定する必要があります。

JSON形式のデータのキーを指定し、`jsonpaths` パラメータ内でシーケンスに並べる必要があります。

また、JSON形式のデータのキー `pay_time` の値を、`pay_time` の値が `example_tbl4` テーブルの `pay_dt` 列に保存される前に DATE 型に変換する必要があります。そのため、`COLUMNS` 内で `pay_dt=from_unixtime(pay_time,'%Y%m%d')` を使用して計算を指定する必要があります。JSON形式のデータの他のキーの値は、`example_tbl4` テーブルに直接マッピングできます。

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

> **注記**
>
> - JSONデータの最上位レベルが配列構造である場合は、`PROPERTIES` 内で `"strip_outer_array"="true"` を設定して外側の配列構造を取り除く必要があります。また、`jsonpaths` を指定する必要がある場合、JSONデータの最上位の配列構造が取り除かれるため、JSONデータ全体のルート要素はフラット化されたJSONオブジェクトです。
> - `json_root` を使用して、JSON形式のデータのルート要素を指定できます。
シンプルモードを使用してRoutine Loadジョブを実行できます。つまり、Routine Loadジョブの作成時に`jsonpaths`パラメータを指定する必要はありません。以下のステートメントを実行して、Routine Loadジョブ`sensor_log_load_job1`を作成し、Kafkaトピック`topic_1`からAvroメッセージを取得してデータをデータベース`sensor`のテーブル`sensor_log1`にロードしてください。

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

Avroスキーマにネストされたレコード型のフィールドが含まれており、ネストされたレコード型のフィールドのサブフィールドをStarRocksにロードする必要があるとします。

**データセットを準備**

- **Avroスキーマ**

    1. 次のAvroスキーマファイル`avro_schema2.avsc`を作成します。外部のAvroレコードには、順番に`id`、`name`、`checked`、`sensor_type`、`data`という5つのフィールドが含まれています。そして、フィールド`data`にはネストされたレコード`data_record`が含まれています。

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

    2. Avroスキーマを[Schema Registry](https://docs.confluent.io/platform/current/schema-registry/index.html)に登録します。

- **Avroデータ**

Avroデータを準備し、それをKafkaトピック`topic_2`に送信してください。

**ターゲットのデータベースとテーブル**

Avroデータのフィールドに従って、StarRocksクラスター内のターゲットデータベース`sensor`にテーブル`sensor_log2`を作成してください。

外部レコードのフィールド`id`、`name`、`checked`、`sensor_type`だけでなく、ネストされたレコード`data_record`内のサブフィールド`data_y`もロードする必要があるとします。

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

**Routine Loadジョブ**

ロードジョブを送信する際に、Avroデータのロードが必要なフィールドを`jsonpaths`で指定します。ネストされたレコード内のサブフィールド`data_y`については、その`jsonpath`を`"$.data.data_y"`と指定する必要があります。

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

#### AvroスキーマにUnionフィールドが含まれている場合

**データセットの準備**

AvroスキーマにUnionフィールドが含まれており、そのUnionフィールドをStarRocksにロードする必要があるとします。

- **Avroスキーマ**

    1. 次のAvroスキーマファイル`avro_schema3.avsc`を作成します。外部のAvroレコードには、順番に`id`、`name`、`checked`、`sensor_type`、`data`という5つのフィールドが含まれています。そして、フィールド`data`は`null`およびネストされたレコード`data_record`の2つの要素からなるUnion型である。

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

    2. Avroスキーマを[Schema Registry](https://docs.confluent.io/platform/current/schema-registry/index.html)に登録します。

- **Avroデータ**

Avroデータを準備し、それをKafkaトピック`topic_3`に送信してください。

**ターゲットのデータベースとテーブル**

Avroデータのフィールドに従って、StarRocksクラスター内のターゲットデータベース`sensor`にテーブル`sensor_log3`を作成してください。

外部レコードのフィールド`id`、`name`、`checked`、`sensor_type`だけでなく、ユニオン型フィールド`data`内の要素`data_record`のフィールド`data_y`もロードする必要があるとします。

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

**Routine Loadジョブ**

ロードジョブを送信する際に、Avroデータのロードが必要なフィールドを`jsonpaths`で指定します。要素`data_y`については、その`jsonpath`を`"$.data.data_y"`と指定する必要があります。

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

ユニオン型フィールド`data`の値が`null`の場合、StarRocksテーブルの`data_y`列にロードされる値は`null`となります。ユニオン型フィールド`data`の値がデータレコードの場合、`data_y`にロードされる値はLong型です。