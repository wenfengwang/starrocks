---
displayed_sidebar: Chinese
---

# CREATE ROUTINE LOAD

## 機能

Routine LoadはApache Kafka®からのメッセージを継続的に消費し、StarRocksにインポートすることをサポートしています。Routine Loadは、Kafkaのメッセージ形式としてCSV、JSON、Avro（v3.0.1以降）をサポートし、Kafkaへのアクセス時には`plaintext`、`ssl`、`sasl_plaintext`、`sasl_ssl`などの複数のセキュリティプロトコルをサポートしています。

この文書では、CREATE ROUTINE LOADの構文、パラメータ説明、および例を紹介します。

> **説明**
>
> - Routine Loadの使用シナリオ、基本原理、基本操作については、[Apache Kafka®からの継続的なインポート](../../../loading/RoutineLoad.md)を参照してください。
> - Routine Load操作には対象テーブルのINSERT権限が必要です。ユーザーアカウントにINSERT権限がない場合は、[GRANT](../account-management/GRANT.md)を参照してユーザーに権限を付与してください。

## 構文

```SQL
CREATE ROUTINE LOAD <database_name>.<job_name> ON <table_name>
[load_properties]
[job_properties]
FROM data_source
[data_source_properties]
```

## パラメータ説明

### `database_name`、`job_name`、`table_name`

`database_name`

任意。対象データベースの名前です。

`job_name`

必須。インポートジョブの名前です。1つのテーブルに複数のインポートジョブが存在する可能性がありますので、Kafkaトピック名やインポートジョブを作成したおおよその時間など、識別可能な情報を使用して意味のあるインポートジョブ名を設定し、複数のインポートジョブを区別することをお勧めします。同一データベース内でインポートジョブ名は一意でなければなりません。

`table_name`

必須。対象テーブルの名前です。

### `load_properties`

任意。ソースデータの属性です。構文：

```SQL
[COLUMNS TERMINATED BY '<column_separator>'],
[ROWS TERMINATED BY '<row_separator>'],
[COLUMNS (<column1_name>[,<column2_name>,<column_assignment>,... ])],
[WHERE <expr>],
[PARTITION (<partition1_name>[,<partition2_name>,...])]
[TEMPORARY PARTITION (<temporary_partition1_name>[,<temporary_partition2_name>,...])]
```

CSV形式のデータをインポートする場合は、列の区切り文字を指定できます。デフォルトは`\t`、つまりタブです。例えば `COLUMNS TERMINATED BY ","` と入力して、列の区切り文字をカンマ(,)に指定できます。

> **説明**
>
> - ここで指定した列の区切り文字がソースデータの列の区切り文字と一致していることを確認する必要があります。
> - StarRocksは、最大50バイトのUTF-8エンコード文字列を列の区切り文字として設定することをサポートしており、カンマ(,)、タブ、パイプ(|)などの一般的な区切り文字が含まれます。
> - NULL値は`\N`で表されます。例えば、ソースデータに3列あり、ある行の第1列と第3列のデータがそれぞれ`a`と`b`で、第2列にデータがない場合、第2列は`\N`を使用してNULL値を表し、`a,\N,b`と書かれます。`a,,b`は第2列が空文字列であることを意味します。

`ROWS TERMINATED BY`

ソースデータ内の行の区切り文字を指定するために使用します。このパラメータを指定しない場合、デフォルトは`\n`です。

`COLUMNS`

ソースデータと対象テーブル間の列のマッピングと変換関係です。詳細は[列マッピングと変換関係](#列マッピングと変換関係)を参照してください。

- `column_name`：マッピング列。ソースデータのこの種類の列の値は、計算せずに直接対象テーブルの列に入れることができます。
- `column_assignment`：派生列。形式は`column_name = expr`で、ソースデータのこの種類の列の値は、式`expr`に基づいて計算された後に対象テーブルの列に入れる必要があります。派生列はマッピング列の後に配置することをお勧めします。なぜならStarRocksは先にマッピング列を解析し、その後で派生列を解析するからです。

> **説明**
>
> 以下の場合は`COLUMNS`パラメータを設定する必要はありません：
>
> - インポート予定のCSVデータの列が対象テーブルの列の数と順序と一致している場合。
> - インポート予定のJSONデータのキー名が対象テーブルの列名と一致している場合。

`WHERE`

フィルタ条件を設定し、条件を満たすデータのみがStarRocksにインポートされます。例えば、`col1`が`100`より大きく、`col2`が`1000`に等しいデータ行のみをインポートしたい場合は、`WHERE col1 > 100 and col2 = 1000`と入力できます。

> **説明**
>
> フィルタ条件で指定された列は、ソースデータに元々存在する列でも、ソースデータの列に基づいて生成された派生列でも構いません。

`PARTITION`

データを対象テーブルの指定されたパーティションにインポートします。パーティションを指定しない場合、データは自動的に対応するパーティションにインポートされます。例：

```SQL
PARTITION(p1, p2, p3)
```

### `job_properties`

必須。インポートジョブの属性です。構文：

```SQL
PROPERTIES ("<key1>" = "<value1>"[, "<key2>" = "<value2>" ...])
```

パラメータの説明は以下の通りです：

| **パラメータ**                  | **必須** | **説明**                                                     |
| ------------------------- | ------------ | ------------------------------------------------------------ |
| desired_concurrent_number | いいえ           | 単一のRoutine Loadインポートジョブの**希望**タスク並行度を表し、1つのインポートジョブが最大でいくつのタスクに分割されて並行実行されるかを示します。デフォルト値は`3`です。しかし、**実際の**タスク並行度は以下の複数のパラメータによって決定される式に基づいており、実際のタスク並行度の上限はBEノードの数または消費パーティションの数になります。`min(alive_be_number, partition_number, desired_concurrent_number, max_routine_load_task_concurrent_num)`。<ul> <li>`alive_be_number`：稼働中のBEノードの数。</li><li>`partition_number`：消費パーティションの数。</li><li>`desired_concurrent_number`：単一のRoutine Loadインポートジョブの希望タスク並行度。デフォルト値は`3`です。</li><li>`max_routine_load_task_concurrent_num`：Routine Loadインポートジョブのデフォルト最大タスク並行度。デフォルト値は`5`です。このパラメータは[FE動的パラメータ](../../../administration/FE_configuration.md)です。</li></ul> |
| max_batch_interval        | いいえ           | タスクのスケジュール間隔、つまりタスクが何秒ごとに実行されるかを指定します。単位は秒です。範囲は`5`～`60`です。デフォルト値は`10`です。インポート間隔は10秒以上を推奨します。それ以下だとインポート頻度が高すぎてバージョン数が多すぎるエラーが発生する可能性があります。 |
| max_batch_rows            | いいえ           | このパラメータはエラー検出ウィンドウの範囲を定義するためにのみ使用され、エラー検出ウィンドウは単一のRoutine Loadインポートタスクが消費する`10 * max-batch-rows`行のデータになります。デフォルトは`10 * 200000 = 2000000`です。インポートタスクでは、ウィンドウ内のデータにエラーがないかどうかをチェックします。エラーデータとは、StarRocksが解析できないデータ、例えば無効なJSONを指します。 |
| max_error_number          | いいえ           | エラー検出ウィンドウ内で許可されるエラーデータ行数の上限です。エラーデータ行数がこの値を超えると、インポートジョブは一時停止されます。この場合、[SHOW ROUTINE LOAD](../data-manipulation/SHOW_ROUTINE_LOAD.md)を実行して`ErrorLogUrls`に基づいてKafka内のメッセージをチェックし、エラーを修正する必要があります。デフォルト値は`0`で、エラー行が許可されていないことを意味します。エラー行には、WHERE句でフィルタリングされたデータは含まれません。|
| strict_mode               | いいえ           | 厳格モードを有効にするかどうかを指定します。値は`TRUE`または`FALSE`です。デフォルト値は`FALSE`です。有効にすると、ソースデータのある列の値が`NULL`であるが、対象テーブルのその列が`NULL`を許可していない場合、その行のデータはフィルタリングされます。<br />このモードの説明については、[厳格モード](../../../loading/load_concept/strict_mode.md)を参照してください。|
| log_rejected_record_num | いいえ | データ品質が不十分でフィルタリングされたデータ行を最大何行まで記録するかを指定します。このパラメータはバージョン3.1以降でサポートされています。値の範囲は`0`、`-1`、正の整数です。デフォルト値は`0`です。<ul><li>`0`の場合、フィルタリングされたデータ行は記録されません。</li><li>`-1`の場合、フィルタリングされたすべてのデータ行が記録されます。</li><li>正の整数（例えば`n`）の場合、各BEノードで最大`n`行のフィルタリングされたデータ行が記録されます。</li></ul> |
| timezone                  | いいえ           | このパラメータの値は、インポートに関連するすべての時差設定関連の関数の結果に影響します。時差に影響される関数には、strftime、alignment_timestamp、from_unixtimeなどがあります。詳細は[タイムゾーンの設定](../../../administration/timezone.md)を参照してください。インポートパラメータ`timezone`で設定されたタイムゾーンは、[タイムゾーンの設定](../../../administration/timezone.md)で説明されているセッションレベルのタイムゾーンに対応します。|
| merge_condition           | いいえ           | 更新が有効になる条件として指定された列名を指定します。このように、インポートされたデータのその列の値が現在の値以上である場合にのみ、更新が有効になります。[インポートによるデータ変更](../../../loading/Load_to_Primary_Key_tables.md)を参照してください。指定された列は非主キー列でなければならず、条件更新は主キーモデルのテーブルのみがサポートされています。 |
| format                    | いいえ           | ソースデータの形式で、値の範囲は`CSV`、`JSON`、または`Avro`（v3.0.1以降）。デフォルト値は`CSV`です。|
| trim_space                | いいえ           | CSVファイル内の列区切り文字の前後のスペースを削除するかどうかを指定します。値のタイプはBOOLEANです。デフォルト値は`false`です。<br />一部のデータベースは、CSVファイルとしてデータをエクスポートする際に、列区切り文字の前後にスペースを追加することがあります。位置によって、これらのスペースは「先行スペース」または「後行スペース」と呼ばれることがあります。このパラメータを設定することで、StarRocksはインポート時にこれらの不要なスペースを削除することができます。<br />ただし、`enclose`で指定された文字で囲まれたフィールド内のスペース（フィールドの先行スペースと後行スペースを含む）は削除されないことに注意してください。例えば、列区切り文字がパイプ（<code class="language-text">&#124;</code>）で、`enclose`で指定された文字がダブルクォート（`"`）の場合：<code class="language-text">&#124; "Love StarRocks" &#124;</code>。trim_spaceをtrueに設定すると、StarRocksが処理した後の結果データは<code class="language-text">&#124;"Love StarRocks"&#124;</code>になります。|

| enclose                   | 否           | 根据 [RFC4180](https://www.rfc-editor.org/rfc/rfc4180) 规定，用于指定将 CSV 文件中的字段括起来的字符。取值类型：单字节字符。默认值：`NONE`。最常用的 `enclose` 字符为单引号 (`'`) 或双引号 (`"`)。<br />被 `enclose` 指定的字符括起来的字段内的所有特殊字符（包括行分隔符、列分隔符等）都将被视为普通字符。StarRocks 提供的 `enclose` 属性比 RFC4180 标准更进一步，支持设置任意单个字节的字符。<br />如果一个字段内包含了 `enclose` 指定的字符，则可以使用相同的字符对 `enclose` 指定的字符进行转义。例如，在设置了 `enclose` 为双引号 (`"`) 时，字段值 `a "quoted" c` 在 CSV 文件中应该写作 `"a ""quoted"" c"`。 |
| escape                    | 否           | 指定用于转义的字符。用于转义各种特殊字符，如行分隔符、列分隔符、转义符、`enclose` 指定的字符等，使 StarRocks 将这些特殊字符解析为字段值的一部分。取值类型：单字节字符。默认值：`NONE`。最常用的 `escape` 字符为反斜杠 (`\`)，在 SQL 语句中应写作双反斜杠 (`\\`)。<br />`escape` 指定的字符同时作用于 `enclose` 指定的字符的内部和外部。<br />以下为两个示例：<br /><ul><li>当设置 `enclose` 为双引号 (`"`)、`escape` 为反斜杠 (`\`) 时，StarRocks 会将 `"say \"Hello world\""` 解析为字段值 `say "Hello world"`。</li><li>假设列分隔符为逗号 (`,`)，当设置 `escape` 为反斜杠 (`\`) 时，StarRocks 会将 `a, b\, c` 解析为 `a` 和 `b, c` 两个字段值。</li></ul> |
| strip_outer_array         | 否           | 是否剪裁 JSON 数据最外层的数组结构。取值范围：`TRUE` 或 `FALSE`。默认值：`FALSE`。在实际业务场景中，待导入的 JSON 数据可能在最外层有一对表示数组结构的中括号 `[]`。在这种情况下，通常建议将此参数设置为 `true`，这样 StarRocks 会剪除外层的中括号 `[]`，并将中括号 `[]` 内的每个内层数组作为单独的一行数据导入。如果此参数设置为 `false`，StarRocks 则会将整个 JSON 数据解析为一个数组，并作为单行数据导入。例如，待导入的 JSON 数据为 `[{"category" : 1, "author" : 2}, {"category" : 3, "author" : 4}]`，如果此参数设置为 `true`，则 StarRocks 会将 `{"category" : 1, "author" : 2}` 和 `{"category" : 3, "author" : 4}` 解析为两行数据，并导入到目标表中的相应行。 |
| jsonpaths                 | 否           | 用于指定待导入字段的名称。仅在使用匹配模式导入 JSON 数据时需要指定此参数。参数取值为 JSON 格式。参见[目标表存在衍生列，其列值通过表达式计算生成](#目标表存在衍生列其列值通过表达式计算生成)。|
| json_root                 | 否           | 如果不需要导入整个 JSON 数据，则指定实际待导入 JSON 数据的根节点。参数取值为合法的 JsonPath。默认值为空，表示将导入整个 JSON 数据。具体请参见本文提供的示例[指定实际待导入 JSON 数据的根节点](#指定实际待导入-json-数据的根节点)。 |
| task_consume_second       | 否           | 单个 Routine Load 导入作业中每个 Routine Load 导入任务消费数据的最大时长，单位为秒。与 [FE 动态参数](../../../administration/FE_configuration.md) `routine_load_task_consume_second`（作用于集群内所有 Routine Load 导入作业）相比，此参数仅针对单个 Routine Load 导入作业，更加灵活。该参数自 v3.1.0 版本起新增。<ul><li>当未配置 `task_consume_second` 和 `task_timeout_second` 时，StarRocks 将使用 FE 动态参数 `routine_load_task_consume_second` 和 `routine_load_task_timeout_second` 来控制导入行为。</li><li>当仅配置 `task_consume_second` 时，默认 `task_timeout_second` = `task_consume_second` * 4。</li><li>当仅配置 `task_timeout_second` 时，默认 `task_consume_second` = `task_timeout_second` / 4。</li></ul>|
| task_timeout_second       | 否           | Routine Load 导入作业中每个 Routine Load 导入任务的超时时间，单位为秒。与 [FE 动态参数](../../../administration/FE_configuration.md) `routine_load_task_timeout_second`（作用于集群内所有 Routine Load 导入作业）相比，此参数仅针对单个 Routine Load 导入作业，更加灵活。该参数自 v3.1.0 版本起新增。<ul><li>当未配置 `task_consume_second` 和 `task_timeout_second` 时，StarRocks 将使用 FE 动态参数 `routine_load_task_consume_second` 和 `routine_load_task_timeout_second` 来控制导入行为。</li><li>当仅配置 `task_timeout_second` 时，默认 `task_consume_second` = `task_timeout_second` / 4。</li><li>当仅配置 `task_consume_second` 时，默认 `task_timeout_second` = `task_consume_second` * 4。</li></ul>|

### `data_source`、`data_source_properties`

数据源及其属性。语法：

```Bash
FROM <data_source>
 ("<key1>" = "<value1>"[, "<key2>" = "<value2>" ...])
```

`data_source`

必填。指定数据源，目前仅支持 `KAFKA`。

`data_source_properties`

必填。数据源属性，参数及其说明如下：

| **参数**                  | **说明**                                                     |
| ------------------------- | ------------------------------------------------------------ |
| kafka_broker_list         | Kafka 的 Broker 连接信息。格式为 `<kafka_broker_ip>:<kafka_port>`，多个 Broker 之间用英文逗号 (,) 分隔。Kafka Broker 默认端口号为 `9092`。示例：`"kafka_broker_list" = "xxx.xx.xxx.xx:9092,xxx.xx.xxx.xx:9092"` |
| kafka_topic               | Kafka Topic 名称。一个导入作业仅支持消费一个 Topic 的消息。  |
| kafka_partitions          | 待消费的分区。示例：`"kafka_partitions" = "0, 1, 2, 3"`。如果不配置此参数，则默认消费所有分区。 |
| kafka_offsets             | 待消费分区的起始消费位点，必须与 `kafka_partitions` 中指定的每个分区一一对应。如果不配置此参数，则默认从分区末尾开始消费。支持的取值为：<ul><li>具体消费位点：从分区中该消费位点的数据开始消费。</li><li>`OFFSET_BEGINNING`：从分区中有数据的位置开始消费。</li><li>`OFFSET_END`：从分区末尾开始消费。</li></ul>多个起始消费位点之间用英文逗号 (,) 分隔。<br />示例：`"kafka_offsets" = "1000, OFFSET_BEGINNING, OFFSET_END, 2000"`。|
| property.kafka_default_offsets | 所有待消费分区的默认起始消费位点。支持的取值与 `kafka_offsets` 相同。 |
| confluent.schema.registry.url | 注册该 Avro schema 的 Schema Registry 的 URL，StarRocks 将从该 URL 获取 Avro schema。格式为 `confluent.schema.registry.url = http[s]://[<schema-registry-api-key>:<schema-registry-api-secret>@]<hostname or ip address>[:<port>]`。|

**更多数据源相关参数**

支持设置更多 Kafka 数据源相关参数，功能等同于 Kafka 命令行的 `--property`，支持的参数请参见 [librdkafka 配置项文档](https://github.com/edenhill/librdkafka/blob/master/CONFIGURATION.md)中适用于客户端的配置项。

> **说明**
>
> 当参数的取值是文件时，值前需加上关键词 `FILE:`。关于如何创建文件，请参见 [CREATE FILE](../Administration/CREATE_FILE.md) 命令文档。

**指定所有待消费分区的默认起始消费位点。**

```Plaintext
"property.kafka_default_offsets" = "OFFSET_BEGINNING"
```

`property.kafka_default_offsets` 的取值可以是具体的消费位点，或者：

- `OFFSET_BEGINNING`：从分区中有数据的位置开始消费。
- `OFFSET_END`：从分区末尾开始消费。

**指定导入任务消费 Kafka 时所基于的 Consumer Group 的 `group.id`**

```Plaintext
"property.group.id" = "group_id_0"
```

如果未指定 `group.id`，StarRocks 将根据 Routine Load 的导入作业名称生成一个随机值，格式为 `{job_name}_{random uuid}`，例如 `simple_job_0a64fe25-3983-44b2-a4d8-f52d3af4c3e8`。

**指定 BE 访问 Kafka 时的安全协议并配置相关参数**

支持的安全协议包括 `plaintext`（默认）、`ssl`、`sasl_plaintext` 和 `sasl_ssl`，并且需要根据安全协议配置相关参数。

当安全协议为 `sasl_plaintext` 或 `sasl_ssl` 时，支持以下 SASL 认证机制：

- PLAIN
- SCRAM-SHA-256 和 SCRAM-SHA-512
- OAUTHBEARER

- **使用 SSL 安全协议访问 Kafka 时**

```sql
"property.security.protocol" = "ssl", -- 指定安全协议为 SSL
"property.ssl.ca.location" = "FILE:ca-cert", -- CA 证书的位置
-- 如果 Kafka server 端开启了 client 认证，则还需设置以下三个参数：
"property.ssl.certificate.location" = "FILE:client.pem", -- Client 的公钥位置
"property.ssl.key.location" = "FILE:client.key", -- Client 的私钥位置
"property.ssl.key.password" = "abcdefg" -- Client 的私钥密码
```

- **使用 SASL_PLAINTEXT 安全协议和 SASL/PLAIN 认证机制访问 Kafka 时**

```sql
"property.security.protocol" = "SASL_PLAINTEXT", -- 指定安全协议为 SASL_PLAINTEXT
"property.sasl.mechanism" = "PLAIN", -- 指定 SASL 认证机制为 PLAIN
"property.sasl.username" = "admin", -- SASL 的用户名
"property.sasl.password" = "admin" -- SASL 的密码
```

### FE 和 BE 配置项

Routine Load 相关配置项，请参见[配置参数](../../../administration/FE_configuration.md)。

## 列映射和转换关系

### 导入 CSV 数据


**CSV形式のデータの列とターゲットテーブルの列の数や順序が一致しない場合**は、`COLUMNS`パラメータを使用して、ソースデータとターゲットテーブル間の列のマッピングと変換関係を指定する必要があります。一般的には以下の2つのシナリオが含まれます：

- **ソースデータの列とターゲットテーブルの列の数は一致していますが、順序が異なる**場合。また、データは関数計算を必要とせず、直接ターゲットテーブルの対応する列に入れることができます。
  `COLUMNS`パラメータでソースデータの列の順序に従って、ターゲットテーブルの対応する列名を使用して列のマッピングと変換関係を設定する必要があります。

  例えば、ターゲットテーブルには`col1`、`col2`、`col3`の順に3列あり、ソースデータにも`col3`、`col2`、`col1`の順に3列あり、それぞれターゲットテーブルの`col1`、`col2`、`col3`に対応しています。この場合、`COLUMNS(col3, col2, col1)`を指定する必要があります。
- **ソースデータの列とターゲットテーブルの列の数が一致しない場合、または一部の列のデータを変換（関数計算後）してターゲットテーブルの対応する列に入れる必要がある**場合。
  `COLUMNS`パラメータでソースデータの列の順序に従って、ターゲットテーブルの対応する列名を使用して列のマッピング関係を設定するだけでなく、データ計算に参加する関数も指定する必要があります。以下は2つの例です：

  - **ソースデータの列がターゲットテーブルの列より多い**場合。
    例えば、ターゲットテーブルには`col1`、`col2`、`col3`の順に3列あり、ソースデータには`col1`、`col2`、`col3`に対応する前3列と、ターゲットテーブルに対応する列がない第4列があります。この場合、`COLUMNS(col1, col2, col3, temp)`を指定する必要があり、最後の列は占有のために任意の名前（例えば`temp`）を指定できます。
  - **ターゲットテーブルにソースデータの列を計算して生成された派生列が存在する**場合。
    例えば、ソースデータには`yyyy-mm-dd hh:mm:ss`形式の時間データを含む1列のみがあります。ターゲットテーブルには`year`、`month`、`day`の順に3列あり、それぞれソースデータの時間データを含む列から計算して生成された派生列です。この場合、`COLUMNS(col, year = year(col), month=month(col), day=day(col))`を指定できます。ここで、`col`はソースデータに含まれる列の一時的な名前で、`year = year(col)`、`month=month(col)`、`day=day(col)`はソースデータの`col`列から対応するデータを抽出してターゲットテーブルの対応する派生列に入れるために使用されます。例えば、`year = year(col)`は`year`関数を使用してソースデータの`col`列から`yyyy`部分のデータを抽出してターゲットテーブルの`year`列に入れることを意味します。

  操作例については、[列のマッピングと変換関係の設定](#设置列的映射和转换关系)を参照してください。

### JSONまたはAvroデータのインポート

> **説明**
>
> バージョン3.0.1から、StarRocksはRoutine Loadを使用してAvroデータをインポートする機能をサポートしています。JSONまたはAvroデータをインポートする際、列のマッピングと変換関係の設定方法は同じです。したがって、このセクションではAvroデータのインポートを例に説明します。

**JSON形式のデータのKey名がターゲットテーブルの列名と一致しない場合**は、`jsonpaths`と`COLUMNS`の2つのパラメータを使用して、ソースデータとターゲットテーブル間の列のマッピングと変換関係を指定する必要があります：

- `jsonpaths`パラメータはインポートするJSONデータのKeyを指定し、ソートします（まるで新しいCSVデータを生成したかのように）。
- `COLUMNS`パラメータはインポートするJSONデータのKeyとターゲットテーブルの列のマッピング関係とデータ変換関係を指定します。
  - `jsonpaths`で指定されたKeyと順序を一致させます。
  - ターゲットテーブルの列と名前を一致させます。

詳細な例については、[派生列が存在し、その列値が式で計算されるターゲットテーブル](#目标表存在衍生列其列值通过表达式计算生成)を参照してください。

> **説明**
>
> インポートするJSONデータのKey名（Keyの順序と数は一致する必要はありません）がすべてターゲットテーブルの列名に対応している場合は、シンプルモードでJSONデータをインポートでき、`jsonpaths`と`COLUMNS`を設定する必要はありません。

## 例

### CSVデータのインポート

このセクションでは、CSV形式のデータを例にして、インポートジョブを作成する際に、さまざまなパラメータ設定を使用して異なるビジネスシナリオのさまざまなインポート要件を満たす方法について詳しく説明します。

**データセット**

KafkaクラスタのTopic `ordertest1`には、次のようなCSV形式のデータが存在するとします。CSVデータの列は、順に注文番号、支払日、顧客名、国籍、性別、支払金額を意味します。

```Plaintext
2020050802,2020-05-08,Johann Georg Faust,Deutschland,male,895
2020050802,2020-05-08,Julien Sorel,France,male,893
2020050803,2020-05-08,Dorian Grey,UK,male,1262
2020050901,2020-05-09,Anna Karenina,Russia,female,175
2020051001,2020-05-10,Tess Durbeyfield,US,female,986
2020051101,2020-05-11,Edogawa Conan,japan,male,8924
```

**ターゲットデータベースとテーブル**

CSVデータからインポートする必要がある列に基づいて、StarRocksクラスタのターゲットデータベース `example_db` に `example_tbl1` テーブルを作成します。テーブル作成のSQLは以下の通りです：

```SQL
CREATE TABLE example_db.example_tbl1 ( 
    `order_id` bigint NOT NULL COMMENT "注文番号",
    `pay_dt` date NOT NULL COMMENT "支払日", 
    `customer_name` varchar(26) NULL COMMENT "顧客名", 
    `nationality` varchar(26) NULL COMMENT "国籍", 
    `gender` varchar(26) NULL COMMENT "性別", 
    `price` double NULL COMMENT "支払金額") 
DUPLICATE KEY (order_id,pay_dt) 
DISTRIBUTED BY HASH(`order_id`);
```

#### Topicから特定のパーティションと開始オフセットで消費を開始

特定のパーティションと各パーティションの開始オフセットを指定する必要がある場合は、`kafka_partitions`、`kafka_offsets`のパラメータを設定する必要があります。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1",
    "kafka_partitions" ="0,1,2,3", -- パーティションを指定
    "kafka_offsets" = "1000, OFFSET_BEGINNING, OFFSET_END, 2000" -- 開始オフセットを指定
);
```

#### インポートパフォーマンスの調整

インポートパフォーマンスを向上させ、消費のバックログなどを避けるためには、単一のRoutine Loadインポートジョブの期待されるタスクの並行数`desired_concurrent_number`を設定し、インポートジョブをできるだけ多くのインポートタスクに分割して並行実行することができます。

> インポートパフォーマンスを向上させる他の方法については、[Routine Loadよくある質問](../../../faq/loading/Routine_load_faq.md)を参照してください。

実際のタスクの並行度は、以下の複数のパラメータによって決まる公式で決定され、上限はBEノードの数または消費パーティションの数です。

```SQL
min(alive_be_number, partition_number, desired_concurrent_number, max_routine_load_task_concurrent_num)
```

したがって、消費パーティションとBEノードの数が多く、他の2つのパラメータよりも大きい場合、実際のタスクの並行度を増やすためには、`max_routine_load_task_concurrent_num`、`desired_concurrent_number`の値を上げることができます。

消費パーティションの数が`7`で、生存しているBEの数が`5`で、`max_routine_load_task_concurrent_num`がデフォルト値の`5`であると仮定します。この場合、実際のタスクの並行度を上限まで増やすためには、`desired_concurrent_number`を`5`（デフォルト値は`3`）に設定する必要があり、実際のタスクの並行度`min(5,7,5,5)`は`5`になります。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
"desired_concurrent_number" = "5" -- 単一のRoutine Loadインポートジョブの期待されるタスクの並行数を設定
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### 列のマッピングと変換関係の設定

CSV形式のデータの列とターゲットテーブルの列の数や順序が一致しない場合、CSVデータの第5列をターゲットテーブルにインポートする必要がないと仮定すると、`COLUMNS`パラメータを使用してソースデータとターゲットテーブル間の列のマッピングと変換関係を指定する必要があります。

**ターゲットデータベースとテーブル**

CSVデータのうち、第5列の性別を除く残りの5列をStarRocksにインポートする必要がある場合、StarRocksクラスタのターゲットデータベース`example_db`に`example_tbl2`テーブルを作成します。

```SQL
CREATE TABLE example_db.example_tbl2 ( 
    `order_id` bigint NOT NULL COMMENT "注文番号",
    `pay_dt` date NOT NULL COMMENT "支払日", 
    `customer_name` varchar(26) NULL COMMENT "顧客名", 
    `nationality` varchar(26) NULL COMMENT "国籍", 
    `price` double NULL COMMENT "支払金額"
) 
DUPLICATE KEY (order_id,pay_dt) 
DISTRIBUTED BY HASH(`order_id`);
```

**インポートジョブ**

この例では、CSVデータの第5列をターゲットテーブルにインポートする必要がないため、`COLUMNS`で第5列を`temp_gender`と一時的に名付けてプレースホルダとして使用し、他の列は直接`example_tbl2`テーブルにマッピングします。

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

#### インポートデータのフィルタリング条件の設定

条件を満たすデータのみをインポートする場合は、WHERE句でフィルタリング条件を設定できます。例えば`price > 100`です。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price),
WHERE price > 100
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### インポートタスクを厳格モードに設定

`PROPERTIES`で`"strict_mode" = "true"`を設定すると、インポートジョブは厳格モードになります。ソースデータのある列の値がNULLで、ターゲットテーブルのその列がNULLを許可していない場合、その行のデータはフィルタリングされます。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
"strict_mode" = "true" -- インポートジョブを厳格モードに設定
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### インポートタスクの許容エラー率を設定

ビジネスシナリオがデータ品質を要求する場合、パラメータ`max_batch_rows`と`max_error_number`を設定して、エラー検出ウィンドウの範囲と許容されるエラーデータ行数の上限を設定する必要があります。エラーデータ行数がこの値を超えると、インポートジョブは一時停止します。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
"max_batch_rows" = "100000", -- エラー検出ウィンドウの範囲は、単一のRoutine Loadインポートタスクが消費する10 * max_batch_rows行数です。
"max_error_number" = "100" -- エラー検出ウィンドウ内で許容されるエラーデータ行数の上限
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1"
);
```

#### セキュリティプロトコルをSSLに指定し、関連パラメータを設定

BEがKafkaにアクセスする際に使用するセキュリティプロトコルをSSLに指定する必要がある場合は、`"property.security.protocol" = "ssl"`などのパラメータを設定する必要があります。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl1_ordertest1 ON example_tbl1
COLUMNS TERMINATED BY ",",
COLUMNS (order_id, pay_dt, customer_name, nationality, gender, price)
PROPERTIES
(
"desired_concurrent_number" = "5"
)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest1",
    "property.security.protocol" = "ssl", -- SSL暗号化を使用
    "property.ssl.ca.location" = "FILE:ca-cert", -- CA証明書の位置
    -- Kafkaサーバー側でクライアント認証が有効になっている場合は、以下の3つのパラメータも設定する必要があります：
    "property.ssl.certificate.location" = "FILE:client.pem", -- クライアントのPublic Keyの位置
    "property.ssl.key.location" = "FILE:client.key", -- クライアントのPrivate Keyの位置
    "property.ssl.key.password" = "abcdefg" -- クライアントのPrivate Keyのパスワード
);
```

#### `trim_space`、`enclose`、`escape`を設定

KafkaクラスタのTopic `test_csv`には、次のようなCSV形式のデータが存在するとします。CSVデータの列は、順に注文番号、支払日、顧客名、国籍、性別、支払金額を意味します。

```Plain
 "2020050802" , "2020-05-08" , "Johann Georg Faust" , "Deutschland" , "male" , "895"
 "2020050802" , "2020-05-08" , "Julien Sorel" , "France" , "male" , "893"
 "2020050803" , "2020-05-08" , "Dorian Grey\,Lord Henry" , "UK" , "male" , "1262"
 "2020050901" , "2020-05-09" , "Anna Karenina" , "Russia" , "female" , "175"
 "2020051001" , "2020-05-10" , "Tess Durbeyfield" , "US" , "female" , "986"
 "2020051101" , "2020-05-11" , "Edogawa Conan" , "japan" , "male" , "8924"
```

Topic `test_csv`のすべてのデータを`example_tbl1`にインポートし、`enclose`で指定された文字で囲まれたフィールドの前後のスペースを削除し、`enclose`文字をダブルクォート(")に指定し、`escape`文字をバックスラッシュ(`\`)に指定する場合は、以下のステートメントを実行します。

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

### JSON形式のデータをインポート

#### ターゲットテーブルの列名がJSONデータのKeyと一致している

`jsonpaths`や`COLUMNS`パラメータを使用せずに、シンプルモードでデータをインポートすることができます。StarRocksはターゲットテーブルの列名をJSONデータのKeyに対応させます。

**データセット**

KafkaクラスタのTopic `ordertest2`には、以下のようなJSONデータが存在するとします。

```JSON
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}
```

> 注意
>
> ここで、各JSONオブジェクトはKafkaメッセージ内に1行でなければなりません。そうでないと、「JSON解析エラー」が発生します。

**ターゲットデータベースとテーブル**

StarRocksクラスタのターゲットデータベース`example_db`に`example_tbl3`テーブルを作成し、列名がインポートする必要があるJSONデータのKeyと一致しています。

```SQL
CREATE TABLE example_db.example_tbl3 ( 
    `commodity_id` varchar(26) NULL COMMENT "商品ID", 
    `customer_name` varchar(26) NULL COMMENT "顧客名", 
    `country` varchar(26) NULL COMMENT "顧客の国籍", 
    `pay_time` bigint(20) NULL COMMENT "支払時間", 
    `price` double SUM NULL COMMENT "支払金額") 
AGGREGATE KEY(`commodity_id`,`customer_name`,`country`,`pay_time`) 
DISTRIBUTED BY HASH(`commodity_id`);
```

**インポートジョブ**

インポートジョブを提出する際には、`jsonpaths`や`COLUMNS`パラメータを使用せずに、シンプルモードを使用して、KafkaクラスタのTopic `ordertest2`のJSONデータをターゲットテーブル`example_tbl3`にインポートできます。

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

> **説明**
>
> - JSONデータの最外層が配列構造の場合は、`PROPERTIES`で`"strip_outer_array"="true"`を設定して、最外層の配列構造を削除する必要があります。また、`jsonpaths`を設定する際には、JSONデータのルートノードが最外層の配列構造を削除した後の**フラットなJSONオブジェクト**であることに注意してください。
> - 全てのJSONデータをインポートする必要がない場合は、`json_root`を使用して、実際にインポートする必要があるJSONデータのルートノードを指定する必要があります。

#### ターゲットテーブルに派生列があり、その列値は式で計算されます

マッチングモードを使用してデータをインポートする必要があります。つまり、`jsonpaths`と`COLUMNS`パラメータを使用して、インポートするJSONデータのKeyを指定し、`COLUMNS`パラメータでインポートするJSONデータのKeyとターゲットテーブルの列のマッピング関係とデータ変換関係を指定する必要があります。

**データセット**

KafkaクラスタのTopic `ordertest2`には、以下のようなJSON形式のデータが存在するとします。

```JSON
{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875}
{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895}
{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}
```

**ターゲットデータベースとテーブル**

StarRocksクラスタのターゲットデータベース`example_db`には、派生列`pay_dt`があり、JSONデータのKey `pay_time`に基づいて計算されたデータが存在するターゲットテーブル`example_tbl4`があります。その作成ステートメントは以下の通りです。

```SQL
CREATE TABLE example_db.example_tbl4 ( 
    `commodity_id` varchar(26) NULL COMMENT "商品ID", 
    `customer_name` varchar(26) NULL COMMENT "顧客名", 
    `country` varchar(26) NULL COMMENT "顧客の国籍",
    `pay_time` bigint(20) NULL COMMENT "支払時間",  
    `pay_dt` date NULL COMMENT "支払日", 
    `price` double SUM NULL COMMENT "支払金額") 
AGGREGATE KEY(`commodity_id`,`customer_name`,`country`,`pay_time`,`pay_dt`) 
DISTRIBUTED BY HASH(`commodity_id`);
```

**インポートジョブ**

インポートジョブを提出する際には、マッチングモードを使用します。`jsonpaths`を使用してインポートするJSONデータのKeyを指定します。また、JSONデータのKey `pay_time`をDATE型に変換してターゲットテーブルの列`pay_dt`にインポートする必要があるため、`COLUMNS`では`from_unixtime`関数を使用して変換する必要があります。JSONデータの他のKeyは、テーブル`example_tbl4`に直接マッピングできます。

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

> **説明**
>
> - JSONデータの最外層が配列構造の場合は、`PROPERTIES`で`"strip_outer_array"="true"`を設定して、最外層の配列構造を削除する必要があります。また、`jsonpaths`を設定する際には、JSONデータのルートノードが最外層の配列構造を削除した後の**フラットなJSONオブジェクト**であることに注意してください。
> - 全てのJSONデータをインポートする必要がない場合は、`json_root`を使用して、実際にインポートする必要があるJSONデータのルートノードを指定する必要があります。

#### ターゲットテーブルに派生列があり、その列値はCASE式で計算されます

**データセット**

KafkaクラスタのTopic `topic-expr-test`には、以下のようなJSON形式のデータが存在するとします。

```JSON
{"key1":1, "key2": 21}
{"key1":12, "key2": 22}
{"key1":13, "key2": 23}
{"key1":14, "key2": 24}
```

**目標データベースとテーブル**

StarRocks クラスターの目標データベース `example_db` には、目標テーブル `tbl_expr_test` が存在し、2つの列が含まれています。列 `col2` の値は JSON データに基づいて CASE 式を計算して得られます。テーブル作成の SQL は以下の通りです：

```SQL
CREATE TABLE tbl_expr_test (
    col1 string, col2 string)
DISTRIBUTED BY HASH (col1);
```

**インポートジョブ**

目標テーブルの列 `col2` の値は JSON データに基づいて CASE 式を計算して得られる必要があります。そのため、インポートジョブの `COLUMNS` パラメータに対応する CASE 式を設定する必要があります。

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

**データクエリ**

目標テーブルのデータをクエリし、結果として列 `col2` の値が CASE 式を計算した後の値であることを表示します。

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

#### 実際にインポートする JSON データのルートノードを指定

全ての JSON データをインポートする必要がない場合は、`json_root` を使用して実際に必要な JSON データのルートオブジェクトを指定する必要があります。パラメータの値は有効な JsonPath である必要があります。

**データセット**

Kafka クラスターのトピック `ordertest3` には、以下のような JSON 形式のデータが存在し、実際にインポートする際には `RECORDS` キーの値のみが必要です。

```JSON
{"RECORDS":[{"commodity_id": "1", "customer_name": "Mark Twain", "country": "US","pay_time": 1589191487,"price": 875},{"commodity_id": "2", "customer_name": "Oscar Wilde", "country": "UK","pay_time": 1589191487,"price": 895},{"commodity_id": "3", "customer_name": "Antoine de Saint-Exupéry","country": "France","pay_time": 1589191487,"price": 895}]}
```

**目標データベースとテーブル**

StarRocks クラスターの目標データベース `example_db` には、目標テーブル `example_tbl3` が存在し、テーブル作成の SQL は以下の通りです：

```SQL
CREATE TABLE example_db.example_tbl3 ( 
    `commodity_id` varchar(26) NULL COMMENT "商品ID", 
    `customer_name` varchar(26) NULL COMMENT "顧客名", 
    `country` varchar(26) NULL COMMENT "顧客国籍", 
    `pay_time` bigint(20) NULL COMMENT "支払時間", 
    `price` double SUM NULL COMMENT "支払金額") 
AGGREGATE KEY(`commodity_id`,`customer_name`,`country`,`pay_time`) 
DISTRIBUTED BY HASH(`commodity_id`);
```

**インポートジョブ**

インポートジョブを提出し、`"json_root" = "$.RECORDS"` を設定して実際にインポートする JSON データのルートノードを指定します。また、実際にインポートする JSON データが配列構造であるため、`"strip_outer_array" = "true"` を設定して外側の配列構造を削除する必要があります。

```SQL
CREATE ROUTINE LOAD example_db.example_tbl3_ordertest3 ON example_tbl3
PROPERTIES
(
    "format" = "json",
    "strip_outer_array" = "true",
    "json_root" = "$.RECORDS"
 )
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest2"
);
```

### Avro データのインポート

バージョン 3.0.1 から、StarRocks は Routine Load を使用して Avro データをインポートすることをサポートしています。

#### Avro スキーマが単純なデータ型のみを含む場合

Avro スキーマが単純なデータ型のみを含むと仮定し、Avro データの全てのフィールドをインポートする必要があります。

**データセット**

**Avro スキーマ**

1. 以下の Avro スキーマファイル `avro_schema1.avsc` を作成します：

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

2. この Avro スキーマを [Schema Registry](https://docs.confluent.io/platform/current/schema-registry/index.html) に登録します。

**Avro データ**

Avro データを構築し、Kafka クラスターのトピック `topic_1` に送信します。

**目標データベースとテーブル**

Avro データに含まれる必要なフィールドに基づいて、StarRocks クラスターの目標データベース `sensor` にテーブル `sensor_log1` を作成します。テーブルの列名は Avro データのフィールド名と一致しています。データ型のマッピングについてはxxxを参照してください。

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

**インポートジョブ**

インポートジョブを提出する際には、`jsonpaths` パラメータを使用せずに、シンプルモードで Kafka クラスターのトピック `topic_1` の Avro データをデータベース `sensor` のテーブル `sensor_log1` にインポートします。

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

#### Avro スキーマに Record 型のフィールドがネストされている場合

Avro スキーマに Record 型のフィールドがネストされており、ネストされた Record フィールドのサブフィールドをインポートする必要があると仮定します。

**データセット**

**Avro スキーマ**

1. 以下の Avro スキーマファイル `avro_schema2.avsc` を作成します。最外層の Record のフィールドは順に `id`、`name`、`checked`、`sensor_type`、`data` で、フィールド `data` にはネストされた Record `data_record` が含まれています。

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

2. この Avro スキーマを [Schema Registry](https://docs.confluent.io/platform/current/schema-registry/index.html) に登録します。

**Avro データ**

Avro データを構築し、Kafka クラスターのトピック `topic_2` に送信します。

**目標データベースとテーブル**

Avro データに含まれる必要なフィールドに基づいて、StarRocks クラスターの目標データベース `sensor` にテーブル `sensor_log2` を作成します。

最外層の Record の `id`、`name`、`checked`、`sensor_type` フィールドに加えて、ネストされた Record の `data_y` フィールドもインポートする必要があるとします。

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

**インポートジョブ**

インポートジョブを提出し、`jsonpaths` を使用して実際にインポートする Avro データのフィールドを指定します。ネストされた Record の `data_y` フィールドについては、`jsonpaths` を `"$.data.data_y"` として指定する必要があります。

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

#### Avro スキーマに Union 型のフィールドが含まれている場合

**データセット**

Avro データに Union 型のフィールドが含まれており、その Union フィールドをインポートする必要があると仮定します。

**Avro スキーマ**

1. 以下の Avro スキーマファイル `avro_schema3.avsc` を作成します。最外層の Record のフィールドは順に `id`、`name`、`checked`、`sensor_type`、`data` で、フィールド `data` は Union 型で、`null` とネストされた Record `data_record` の2つの要素が含まれています。

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

2. この Avro スキーマを [Schema Registry](https://docs.confluent.io/platform/current/schema-registry/index.html) に登録します。

**Avro データ**

Avro データを構築し、Kafka クラスターのトピック `topic_3` に送信します。

**目標データベースとテーブル**

Avro データに含まれる必要なフィールドに基づいて、StarRocks クラスターのターゲットデータベース `sensor` に `sensor_log3` テーブルを作成します。

最外層の Record の `id`、`name`、`checked`、`sensor_type` フィールドに加えて、Union 型のフィールド `data` の要素 `data_record` に含まれるフィールド `data_y` もインポートする必要があるとします。

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

**インポートジョブ**

インポートジョブを提出し、`jsonpaths` を使用して実際にインポートする Avro データのフィールドを指定します。ここで、フィールド `data_y` の `jsonpaths` を `"$.data.data_y"` と指定する必要があります。

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

Union 型のフィールド `data` の値が `null` の場合は、`data_y` 列の値も `null` としてインポートします。`data` の値が data record の場合は、`data_y` 列の値を Long 型としてインポートします。

