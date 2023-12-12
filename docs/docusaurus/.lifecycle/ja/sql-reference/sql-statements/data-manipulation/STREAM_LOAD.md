---
displayed_sidebar: "Japanese"
---

# ストリームロード

## 説明

StarRocksは、HTTPベースのストリームロード方法を提供し、地域のファイルシステムまたはストリーミングデータソースからデータをロードするのを支援します。 ロードジョブを提出すると、StarRocksはジョブを同期的に実行し、ジョブの結果を返します。ジョブが終了した後、その結果に基づいてジョブが成功したかどうかを判断できます。 ストリームロードのアプリケーションシナリオ、制限、原則、サポートされているデータファイルフォーマットの詳細については、[ローカルファイルシステムからストリームロードを使用してデータをロード](../../../loading/StreamLoad.md#loading-from-a-local-file-system-via-stream-load)を参照してください。

> **注意**
>
> - Stream Loadを使用してStarRocksテーブルにデータをロードした後、そのテーブルで作成されたマテリアライズドビューのデータも更新されます。
> - Stream Loadを使用してデータをStarRocksテーブルにロードできるのは、それらのStarRocksテーブルにINSERT権限を持つユーザーのみです。 INSERT権限がない場合は、[GRANT](../account-management/GRANT.md)で接続するユーザーにINSERT権限を付与するよう指示されます。

## 構文

```Bash
curl --location-trusted -u <username>:<password> -XPUT <url>
(
    data_desc
)
[opt_properties]        
```

このトピックでは、curlを使用してストリームロードを実行する方法を説明する例を示します。curl以外にも、他のHTTP互換ツールや言語を使用してストリームロードを実行することもできます。 ロード関連のパラメーターは、HTTPリクエストヘッダーフィールドに含まれています。 これらのパラメーターを入力する際には、次の点に注意してください。

- このトピックで示すように、チャンクされた転送エンコーディングを使用できます。チャンクされた転送エンコーディングを選択しない場合は、データの長さを示す`Content-Length`ヘッダーフィールドを入力して、データの整合性を確保する必要があります。

  > **注意**
  >
  > curlを使用してストリームロードを実行する場合、StarRocksは自動的に`Content-Length`ヘッダーフィールドを追加するため、手動で入力する必要はありません。

- `Expect`ヘッダーフィールドを追加し、その値を`100-continue`と指定し、「"Expect:100-continue"」のようにします。 これにより、ジョブリクエストが拒否された場合に不要なデータ転送を防止し、リソースオーバーヘッドを削減できます。

StarRocksでは、一部のリテラルがSQL言語による予約キーワードとして使用されます。 SQLステートメントでこれらのキーワードを直接使用しないでください。 SQLステートメントでそのようなキーワードを使用する場合は、バッククォート（``）で囲んでください。 詳細は[Keywords](../../../sql-reference/sql-statements/keywords.md)を参照してください。

## パラメーター

### ユーザー名とパスワード

StarRocksクラスタに接続するために使用するアカウントのユーザー名とパスワードを指定します。 これは必須パラメーターです。 パスワードが設定されていないアカウントを使用する場合は、`<username>:`のみを入力する必要があります。

### XPUT

HTTPリクエストメソッドを指定します。 これは必須パラメーターです。 ストリームロードはPUTメソッドのみをサポートしています。

### url

StarRocksテーブルのURLを指定します。 構文:

```Plain
http://<fe_host>:<fe_http_port>/api/<database_name>/<table_name>/_stream_load
```

次の表に、URLのパラメーターについて説明します。

| パラメーター     | 必須     | 説明                                                       |
| ------------- | -------- | ------------------------------------------------------------ |
| fe_host       | Yes      | StarRocksクラスタのFEノードのIPアドレス。<br/>**注意**<br/>ロードジョブを特定のBEノードに提出する場合は、BEノードのIPアドレスを入力する必要があります。 |
| fe_http_port  | Yes      | StarRocksクラスタのFEノードのHTTPポート番号。デフォルトのポート番号は`8030`です。<br/>**注意**<br/>ロードジョブを特定のBEノードに提出する場合は、BEノードのHTTPポート番号を入力する必要があります。デフォルトのポート番号は`8030`です。 |
| database_name | Yes      | StarRocksテーブルが属するデータベースの名前。                  |
| table_name    | Yes      | StarRocksテーブルの名前。                                         |

> **注意**
>
> [SHOW FRONTENDS](../Administration/SHOW_FRONTENDS.md)を使用して、FEノードのIPアドレスとHTTPポートを表示できます。

### data_desc

ロードするデータファイルについての記述です。 `data_desc`記述子には、データファイルの名前、形式、列セパレーター、行セパレーター、宛先パーティション、およびStarRocksテーブルに対する列マッピングが含まれる場合があります。 構文:

```Bash
-T <file_path>
-H "format: CSV | JSON"
-H "column_separator: <column_separator>"
-H "row_delimiter: <row_delimiter>"
-H "columns: <column1_name>[, <column2_name>, ... ]"
-H "partitions: <partition1_name>[, <partition2_name>, ...]"
-H "temporary_partitions: <temporary_partition1_name>[, <temporary_partition2_name>, ...]"
-H "jsonpaths: [ \"<json_path1>\"[, \"<json_path2>\", ...] ]"
-H "strip_outer_array:  true | false"
-H "json_root: <json_path>"
```

`data_desc`記述子のパラメータは、共通パラメータ、CSVパラメータ、JSONパラメータに分類できます。

#### 共通パラメータ

| パラメータ  | 必須     | 説明                                                       |
| ------------ | -------- | ------------------------------------------------------------ |
| file_path    | Yes      | データファイルの保存パス。ファイル名の拡張子を省略することもできます。 |
| format       | No       | データファイルの形式。有効な値: `CSV`および`JSON`。デフォルト値: `CSV`。 |
| partitions   | No       | データファイルをロードするパーティション。デフォルトでは、このパラメータを指定しない場合、StarRocksはデータファイルをStarRocksテーブルのすべてのパーティションにロードします。 |
| temporary_partitions|  No       | データファイルをロードする[一時的パーティション](../../../table_design/Temporary_partition.md)の名前を指定します。 複数の一時的パーティションを指定できますが、それらはカンマ(,)で区切られている必要があります。|
| columns      | No       | データファイルとStarRocksテーブルの間の列マッピング。<br />データファイルのフィールドがStarRocksテーブルの列に連続してマッピングできる場合、このパラメータを指定する必要はありません。 代わりに、このパラメータを使用してデータ変換を実装できます。 たとえば、CSVデータファイルをロードし、ファイルが`id`および`city`の2つの列で構成されており、それらがStarRocksテーブルの2つの列`city`と`id`に連続してマッピングできる場合、`"columns: city,tmp_id, id = tmp_id * 100"`を指定できます。 詳細については、本トピックの「[列マッピング](#column-mapping)」セクションを参照してください。 |

#### CSVパラメータ

| パラメータ      | 必須     | 説明                                                       |
| ---------------- | -------- | ------------------------------------------------------------ |
| column_separator | No       | データファイルでフィールドを区切るために使用される文字。 このパラメータを指定しない場合は、デフォルトで`\t`が使用されます。 <br />指定する列セパレーターは、ロードコマンドで指定した行セパレーターと同じであることを確認してください。<br/>**注意**<br/>CSVデータの場合、テキスト区切り子として、カンマ（,）、タブ、またはパイプ（\|）など、長さが50バイトを超えないUTF-8文字列を使用できます。 |
| row_delimiter    | No       | データファイルで行を区切るために使用される文字。 このパラメータを指定しない場合、デフォルトで`\n`が使用されます。 |
| skip_header      | No       | データファイルがCSV形式の場合、データファイルの最初の行数をスキップするかどうかを指定します。 タイプ: INTEGER。 デフォルト値: `0`。<br />CSV形式のデータファイルの場合、最初の数行は列名や列データ型などのメタデータを定義するために使用されることがあります。 `skip_header`パラメーターを設定することで、データロード時にデータファイルの最初の数行をStarRocksがスキップできます。 たとえば、このパラメータを`1`に設定すると、StarRocksはデータロード時にデータファイルの最初の行をスキップします。<br />データファイルの冒頭の数行は、ロードコマンドで指定した行セパレーターで区切られている必要があります。|
| trim_space       | いいえ       | データファイルがCSV形式の場合、列の区切り文字の直前および直後にあるスペースを削除するかどうかを指定します。タイプ：BOOLEAN。デフォルト値： `false`。<br />一部のデータベースでは、CSV形式のデータファイルとしてエクスポートする際に、列の区切り文字にスペースが追加されます。こうしたスペースは、その位置に応じてその前のスペースまたは後のスペースと呼ばれます。 `trim_space` パラメータを設定することで、不要なスペースをStarRocksがデータロード中に削除することができます。<br />なお、StarRocksは、一対の `enclose`-指定された文字で囲まれたフィールド内のスペース（前後のスペースを含む）を削除しません。たとえば、次のフィールド値は、パイプ（<code class="language-text">&#124;</code>）を列の区切り文字とし、二重引用符（`"`）を `enclose`-指定された文字として使用します：<br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> <br /><code class="language-text">&#124;" Love StarRocks "&#124;</code> <br /><code class="language-text">&#124; "Love StarRocks" &#124;</code> <br />`trim_space` を `true` に設定すると、StarRocksは次のフィールド値を次のように処理します：<br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> <br /><code class="language-text">&#124;" Love StarRocks "&#124;</code> <br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> |
| enclose          | いいえ       | データファイルがCSV形式の場合、[RFC4180](https://www.rfc-editor.org/rfc/rfc4180) に従ってフィールド値を囲む文字を指定します。タイプ：シングルバイト文字。デフォルト値： `NONE`。最も一般的な文字は、シングルクォーテーションマーク（`'`）、ダブルクォーテーションマーク（`"`）です。<br />`enclose`-指定された文字で囲まれたすべての特殊文字（行区切り文字および列区切り文字を含む）は、通常のシンボルとして処理されます。StarRocks は RFC4180 以上の機能があり、任意のシングルバイト文字を `enclose`-指定された文字として指定できます。<br />フィールド値に `enclose`-指定された文字を含む場合は、同じ文字を使用してその `enclose`-指定された文字をエスケープできます。たとえば、`enclose` を `"` に設定し、フィールド値が `a "quoted" c` の場合、フィールド値を `"a ""quoted"" c"` としてデータファイルに入力できます。 |
| escape           | いいえ       | 行区切り文字、列区切り文字、エスケープ文字、および `enclose`-指定された文字など、様々な特殊文字をエスケープするために使用される文字を指定します。タイプ：シングルバイト文字。デフォルト値： `NONE`。最も一般的な文字はスラッシュ（`\\`）です。SQLステートメントでは、スラッシュはダブルスラッシュ（`\\\\`）として書かれる必要があります。<br />**注記**<br />`escape` で指定された文字は、一対の `enclose`-指定された文字の内側および外側の両方に適用されます。<br />次の2つの例を参照してください：<ul><li>`enclose` を `"` に、`escape` を `\` に設定した場合、StarRocks は `"say \"Hello world\""` を `say "Hello world"` に解析します。</li><li>列区切り文字がコンマ（`,`）の場合、`escape` を `\` に設定したとき、StarRocks は `a, b\\, c` を2つの別々のフィールド値 `a` および `b, c` に解析します。</li></ul> |

> **注記**
>
> - CSVデータでは、50バイトを超えない UTF-8 文字列（コンマ（,）やタブ、パイプ（|）など）をテキストの区切り文字として使用できます。
> - ヌル値は `\N` を使用して示します。たとえば、データファイルが3つの列で構成されており、そのデータファイルのレコードが最初と三番目の列にはデータがありますが、二番目の列にはデータがない場合は、二番目の列に null 値を表す `\N` を使用する必要があります。この場合、レコードは `a,\N,b` として編集する必要があります。 `a,,b` はレコードの二番目の列に空の文字列のデータがあることを示します。
> - `skip_header`、`trim_space`、`enclose`、`escape`を含むフォーマットオプションは、v3.0以降でサポートされています。

#### JSONパラメータ

| パラメータ         | 必須 | 説明                                                  |
| ----------------- | ---- | ----------------------------------------------------- |
| jsonpaths         | いいえ | JSONデータファイルから読み込むキーの名前。マッチングモードを使用してJSONデータをロードする場合にのみ、このパラメータを指定する必要があります。このパラメータの値は JSON 形式です。[JSONデータロードの列マッピングの構成](#configure-column-mapping-for-json-data-loading)を参照してください。           |
| strip_outer_array | いいえ | 最外部の配列構造を削除するかどうかを指定します。有効な値: `true` および `false`。デフォルト値：`false`。<br />実務上のシナリオでは、JSONデータに外部の配列構造がある場合があります。これは、角括弧 `[]` で示されます。このような場合、StarRocks は `true` にこのパラメータを設定することをお勧めします。これにより、StarRocks は外部の角括弧 `[]` を削除し、内部の各配列を別々のデータレコードとしてロードします。`false` に設定すると、StarRocks はJSONデータファイル全体を1つの配列として解析し、その配列を1つのデータレコードとして読み込みます。<br />たとえば、次のJSONデータがあるとします：`[ {"category" : 1, "author" : 2}, {"category" : 3, "author" : 4} ]`。このパラメータを `true` に設定すると、 `{"category" : 1, "author" : 2}` および `{"category" : 3, "author" : 4}` は別々のデータレコードとして解析され、別々の StarRocks テーブル行に読み込まれます。 |
| json_root         | いいえ | JSONデータファイルから読み込みたいJSONデータのルート要素。マッチングモードを使用してJSONデータをロードする場合にのみ、このパラメータを指定する必要があります。このパラメータの値は有効な JsonPath 文字列です。デフォルトでは、このパラメータの値が空であるため、JSONデータファイルのすべてのデータが読み込まれます。詳細は、本トピックの「[ルート要素が指定されたマッチングモードを使用してJSONデータをロード](#load-json-data-using-matched-mode-with-root-element-specified)」セクションを参照してください。 |
| ignore_json_size  | いいえ | HTTPリクエストのJSON本文のサイズをチェックするかどうかを指定します。<br/>**注記**<br/>デフォルトでは、HTTPリクエストのJSON本文のサイズは100 MBを超えることはできません。JSON本文のサイズが100 MBを超える場合は、エラー "The size of this batch exceed the max size [104857600] of json type data data [8617627793]. Set ignore_json_size to skip check, although it may lead huge memory consuming." が報告されます。このエラーを防ぐために、HTTPリクエストヘッダーに `"ignore_json_size:true"` を追加して、StarRocks にJSON本文のサイズをチェックしないように指示できます。 |

JSONデータをロードする場合は、1つのJSONオブジェクトあたりのサイズが4 GBを超えてはいけないことにも注意してください。JSONデータファイル内の個々のJSONオブジェクトが4 GBを超える場合、エラー "This parser can't support a document that big." が報告されます。

### opt_properties

ロードジョブ全体に適用されるいくつかのオプションパラメータを指定します。構文：

```Bash
-H "label: <label_name>"
-H "where: <condition1>[, <condition2>, ...]"
-H "max_filter_ratio: <num>"
-H "timeout: <num>"
-H "strict_mode: true | false"
-H "timezone: <string>"
-H "load_mem_limit: <num>"
-H "merge_condition: <column_name>"
```

次の表は、オプションパラメータの説明です。

| パラメータ        | 必須 | 説明                                                  |
| ---------------- | ---- | ----------------------------------------------------- |
| label            | いいえ | ロードジョブのラベル。このパラメータを指定しない場合、StarRocksは自動でロードジョブのラベルを生成します。<br/>StarRocks では、1つのラベルを使用してデータバッチを複数回ロードすることはできません。そのため、同じデータを繰り返しロードすることを防ぎます。ラベルの命名規則については、「[システム上の制限](../../../reference/System_limit.md)」を参照してください。<br/>デフォルトでは、StarRocksは直近3日間で正常に完了したロードジョブのラベルを保持します。ラベルの保持期間を変更する場合は、[FEパラメータ](../../../administration/Configuration.md) `label_keep_max_second` を使用できます。 |
| where            | いいえ | StarRocks が前処理されたデータをフィルタリングする条件。WHERE句で指定されたフィルター条件を満たす前処理されたデータのみが読み込まれます。 |
| max_filter_ratio | No       | ロードジョブの最大エラートレランス。エラートレランスは、ロードジョブでリクエストされたすべてのデータレコードの中で、データ品質が不十分なためにフィルタリングされることができる最大データレコードのパーセンテージです。有効な値: `0` から `1`。デフォルト値: `0`。<br/>デフォルト値 `0` を維持することをお勧めします。これにより、不適格なデータレコードが検出された場合、ロードジョブが失敗し、データの正確性が保証されます。<br/>不適格なデータレコードを無視する場合は、このパラメータを `0` より大きい値に設定できます。これにより、データファイルに不適格なデータレコードが含まれていても、ロードジョブが成功することがあります。<br/>**注記**<br/>不適格なデータレコードには、WHERE句によってフィルタリングされたデータレコードは含まれません。 |
| log_rejected_record_num | No           | ログに記録できる不適格なデータ行の最大数を指定します。このパラメータは v3.1 以降でサポートされています。有効な値: `0`、`-1`、および正の整数。デフォルト値: `0`。<ul><li>値 `0` はフィルタリングされたデータ行を記録しないことを指定します。</li><li>値 `-1` はフィルタリングされたすべてのデータ行を記録することを指定します。</li><li>`n` などの正の整数は、各 BE ごとに記録できる最大データ行数を `n` まで指定します。</li></ul> |
| timeout          | No       | ロードジョブのタイムアウト期間。有効な値: `1` から `259200`。単位: 秒。デフォルト値: `600`。<br/>**注記**`timeout` パラメータに加えて、StarRocks クラスタ内のすべてのストリームロードジョブのタイムアウト期間を一元管理するために、[FEパラメータ](../../../administration/Configuration.md) `stream_load_default_timeout_second` を使用することもできます。`timeout` パラメータを指定すると、`timeout` パラメータで指定されたタイムアウト期間が優先されます。`timeout` パラメータを指定しない場合、`stream_load_default_timeout_second` パラメータで指定されたタイムアウト期間が優先されます。 |
| strict_mode      | No       | [厳密モード](../../../loading/load_concept/strict_mode.md) を有効にするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `false`。`true` は厳密モードを有効にし、`false` は厳密モードを無効にします。 |
| timezone         | No       | ロードジョブで使用するタイムゾーン。デフォルト値: `Asia/Shanghai`。このパラメータの値は、strftime、alignment_timestamp、from_unixtime などの関数によって返される結果に影響を与えます。このパラメータで指定されたタイムゾーンは、セッションレベルのタイムゾーンです。詳細については、[タイムゾーンの構成](../../../administration/timezone.md)を参照してください。 |
| load_mem_limit   | No       | ロードジョブに割り当てることができる最大メモリ量。単位: バイト。デフォルトでは、ロードジョブの最大メモリサイズは 2 GB です。このパラメータの値は、各 BE に割り当てることができる最大メモリ量を超えることはできません。 |
| merge_condition  | No       | 更新が有効になる条件として使用する列の名前を指定します。ソースレコードからターゲットレコードへの更新は、指定された列の値がターゲットデータレコードの値以上である場合にのみ有効になります。StarRocks は v2.5 以降、条件付き更新をサポートしています。詳細については、[ローディングを通じたデータの変更](../../../loading/Load_to_Primary_Key_tables.md)を参照してください。<br/>**注記**<br/>指定した列はプライマリ キー列ではない必要があります。また、条件付き更新はプライマリ キー テーブルを使用するテーブルのみがサポートしています。 |

## カラム マッピング

### CSV データロード用のカラム マッピングを構成する

データファイルの列を StarRocks テーブルの列に1対1でシーケンスにマッピングできる場合、データファイルと StarRocks テーブルの列の間のカラム マッピングを構成する必要はありません。

データファイルの列を StarRocks テーブルの列に1対1でシーケンスにマッピングできない場合、`columns` パラメータを使用して、データファイルと StarRocks テーブルの列間のカラム マッピングを構成する必要があります。これには次の2つのユースケースが含まれます。

- **列の数は同じですが、列のシーケンスが異なります。** **さらに、データファイルのデータは一致する StarRocks テーブルの列にロードされる前に関数によって計算する必要はありません。**

`columns` パラメータでは、データファイルの列が配置されている方法と同じシーケンスで、StarRocks テーブルの列の名前を指定する必要があります。

例えば、StarRocks テーブルは `col1`、`col2`、`col3` の3つの列で構成され、データファイルも3つの列で構成されており、これらは StarRocks テーブルの列 `col3`、`col2`、`col1` にそれぞれマッピングされます。この場合、`"columns: col3, col2, col1"` を指定する必要があります。

- **列の数が異なり、列のシーケンスも異なります。また、データファイルのデータは一致する StarRocks テーブルの列にロードされる前に関数によって計算する必要があります。**

`columns` パラメータでは、データファイルの列が配置されている方法と同じシーケンスで、StarRocks テーブルの列の名前を指定し、データを計算するために使用する関数を指定する必要があります。次の2つの例があります。

- StarRocks テーブルは `col1`、`col2`、`col3` の3つの列で構成されており、データファイルは4つの列で構成されていて、最初の3つの列がそれぞれ StarRocks テーブルの列 `col1`、`col2`、`col3`にマッピングされ、4番目の列は StarRocks テーブルの列にマッピングできない場合、一時的な名前をデータファイルの4番目の列のために指定する必要があります。一時的な名前は、StarRocks テーブルの列名とは異なる必要があります。例えば、`"columns: col1, col2, col3, temp"` を指定します。ここで、データファイルの4番目の列は一時的に `temp` という名前が付けられます。
- StarRocks テーブルは `year`、`month`、`day` の3つの列で構成されており、データファイルには `yyyy-mm-dd hh:mm:ss` 形式の日時値を収容できる列のみが含まれています。この場合、`"columns: col, year = year(col), month=month(col), day=day(col)"` を指定します。ここで、`col` はデータファイルの列の一時的な名前であり、`year = year(col)`、`month=month(col)`、`day=day(col)` という関数は、データファイルの列 `col` からデータを抽出し、対応する StarRocks テーブルの列に データをロードするために使用されます。例えば、`year = year(col)` はデータファイルの列 `col` から `yyyy` のデータを抽出し、StarRocks テーブルの列 `year` にデータをロードするために使用されます。

詳細な例については、[カラム マッピングの構成](#カラム-マッピングの構成)を参照してください。

### JSON データロード用のカラム マッピングを構成する

JSON ドキュメントのキーが StarRocks テーブルの列の名前と同じ名前である場合は、シンプルモードを使用して JSON 形式のデータをロードできます。シンプルモードでは、`jsonpaths` パラメータを指定する必要はありません。このモードでは、JSON 形式のデータは、カーリブラケット `{}` によって示されるオブジェクトである必要があります。 例: `{"category": 1, "author": 2, "price": "3"}`。この例では、`category`、`author`、`price` はキー名であり、これらのキーは StarRocks テーブルの `category`、`author`、`price` と一致する列に名前ごとにマッピングされます。

JSON ドキュメントのキーが StarRocks テーブルの列の名前と異なる名前を持つ場合は、マッチング モードを使用して JSON 形式のデータをロードする必要があります。マッチング モードでは、`jsonpaths` および `COLUMNS` パラメータを使用して、JSON ドキュメントと StarRocks テーブルの列の間のカラム マッピングを指定する必要があります。

- `jsonpaths` パラメータでは、JSON のキーを、JSON ドキュメント内で配置されている方法と同じシーケンスで指定する必要があります。
- `COLUMNS` パラメータでは、JSON のキーと StarRocks テーブルの列との間のマッピングを指定する必要があります:
  - `COLUMNS` パラメータで指定した列名は、JSON のキー名とシーケンスごとに1対1にマッピングされます。
  - `COLUMNS` パラメータで指定した列名は、StarRocks テーブルの列名と一致する名前ごとに1対1にマッピングされます。

マッチングモードを使用して JSON 形式のデータをロードする例については、「マッチング モードを使用した JSON データのロード」を参照してください。

## 戻り値

ロードジョブが完了すると、StarRocks はジョブ結果を JSON 形式で返します。 例:

```JSON
{
    "TxnId": 1003,
    "Label": "label123",
    "Status": "Success",
    "Message": "OK",
    "NumberTotalRows": 1000000,
    "NumberLoadedRows": 999999,
    "NumberFilteredRows": 1,
    "NumberUnselectedRows": 0,
    "LoadBytes": 40888898,
    "LoadTimeMs": 2144,
    "BeginTxnTimeMs": 0,
```json
    "StreamLoadPlanTimeMs": 1,
    "ReadDataTimeMs": 0,
    "WriteDataTimeMs": 11,
    "CommitAndPublishTimeMs": 16,
}
```

以下の表には、返されたジョブ結果のパラメータが記載されています。

| パラメータ名          | 説明                                                         |
| --------------------- | ------------------------------------------------------------ |
| TxnId                 | ロードジョブのトランザクションID。                           |
| Label                 | ロードジョブのラベル。                                       |
| Status                | データの最終ステータス。<ul><li>`Success`：データのロードに成功し、クエリできます。</li><li>`Publish Timeout`：ロードジョブは正常に提出されましたが、データはまだクエリできません。データのロードを再試行する必要はありません。</li><li>`Label Already Exists`：ロードジョブのラベルが別のロードジョブで使用されています。データは正常にロードされた可能性がありますし、ロード中の可能性もあります。</li><li>`Fail`：データのロードに失敗しました。ロードジョブを再試行できます。</li></ul> |
| Message               | ロードジョブのステータス。ロードジョブが失敗した場合、詳細な失敗原因が返されます。 |
| NumberTotalRows       | 読み込まれたデータレコードの総数。                         |
| NumberLoadedRows      | 正常にロードされたデータレコードの総数。`Status`の戻り値が`Success`の場合にのみ有効なパラメータです。 |
| NumberFilteredRows    | データ品質が不十分でフィルタリングされたデータレコードの数。 |
| NumberUnselectedRows  | WHERE句によってフィルタリングされたデータレコードの数。     |
| LoadBytes             | ロードされたデータの量。単位：バイト。                     |
| LoadTimeMs            | ロードジョブにかかった時間。単位：ms。                     |
| BeginTxnTimeMs        | ロードジョブのトランザクション実行にかかった時間。          |
| StreamLoadPlanTimeMs  | ロードジョブの実行計画を生成するためにかかった時間。       |
| ReadDataTimeMs        | ロードジョブのデータ読み込みにかかった時間。               |
| WriteDataTimeMs       | ロードジョブのデータ書き込みにかかった時間。               |
| CommitAndPublishTimeMs| ロードジョブのコミットとデータ公開にかかった時間。         |

ロードジョブが失敗した場合、StarRocksは`ErrorURL`も返します。例:

```JSON
{"ErrorURL": "http://172.26.195.68:8045/api/_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be"}
```

`ErrorURL`は、フィルタリングされた不適格なデータレコードの詳細を取得できるURLを提供します。オプションパラメータ`log_rejected_record_num`を使用して、記録する不適格なデータ行の最大数を指定できます。

`curl "url"`を実行して、フィルタリングされた不適格なデータレコードの詳細を直接表示できます。また、`wget "url"`を実行して、これらのデータレコードの詳細をエクスポートできます。

以下の例のように、エクスポートされたデータレコードの詳細は、名前が`_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be`と似たファイルに保存されます。このファイルを表示するために`cat`コマンドを使用できます。

その後、ロードジョブの構成を調整し、ロードジョブを再度提出できます。

## Examples

### CSVデータのロード

このセクションでは、CSVデータを使用して、さまざまなパラメータ設定と組み合わせを利用してさまざまなロード要件に対応する方法を説明します。

#### タイムアウト期間の設定

StarRocksデータベース`test_db`には`table1`という名前のテーブルがあります。このテーブルには、`col1`、`col2`、`col3`の3つの列が含まれています。

データファイル`example1.csv`にも、`table1`の`col1`、`col2`、`col3`に順にマッピングされる3つの列が含まれています。

`example1.csv`のすべてのデータを最大100秒以内に`table1`にロードする場合は、次のコマンドを実行します。

```Bash
curl --location-trusted -u <username>:<password> -H "label:label1" \
    -H "Expect:100-continue" \
    -H "timeout:100" \
    -H "max_filter_ratio:0.2" \
    -T example1.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
```

#### エラートレランスの設定

StarRocksデータベース`test_db`には`table2`という名前のテーブルがあります。このテーブルには、`col1`、`col2`、`col3`の3つの列が含まれています。

データファイル`example2.csv`にも、`table2`の`col1`、`col2`、`col3`に順にマッピングされる3つの列が含まれています。

`example2.csv`のすべてのデータを`table2`に最大誤差トレランス`0.2`でロードする場合は、次のコマンドを実行します。

```Bash
curl --location-trusted -u <username>:<password> -H "label:label2" \
    -H "Expect:100-continue" \
    -H "max_filter_ratio:0.2" \
    -T example2.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table2/_stream_load
```

#### 列のマッピングの構成

StarRocksデータベース`test_db`には`table3`という名前のテーブルがあります。このテーブルには、`col1`、`col2`、`col3`の3つの列が含まれています。

データファイル`example3.csv`にも、`table3`の`col2`、`col1`、`col3`に順にマッピングされる3つの列が含まれています。

`example3.csv`のすべてのデータを`table3`にロードする場合は、次のコマンドを実行します。

```Bash
curl --location-trusted -u <username>:<password>  -H "label:label3" \
    -H "Expect:100-continue" \
    -H "columns: col2, col1, col3" \
    -T example3.cs\ -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table3/_stream_load
```

> **注意**
>
> 上記の例では、`example3.csv`の列は`table3`の列と同じ順序でマッピングできません。したがって、`columns`パラメーターを使用して`example3.csv`と`table3`の列のマッピングを定義する必要があります。

#### フィルタ条件の設定

StarRocksデータベース`test_db`には`table4`という名前のテーブルがあります。このテーブルには、`col1`、`col2`、`col3`の3つの列が含まれています。

データファイル`example4.csv`にも、`table4`の`col1`、`col2`、`col3`に順にマッピングされる3つの列が含まれています。

`example4.csv`のうち、`col1`の値が`20180601`と等しいデータレコードのみを`table4`にロードする場合は、次のコマンドを実行します。

```Bash
curl --location-trusted -u <username>:<password> -H "label:label4" \
    -H "Expect:100-continue" \
    -H "columns: col1, col2, col3]"\
    -H "where: col1 = 20180601" \
    -T example4.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table4/_stream_load
```

> **注意**
>
> 上記の例では、`example4.csv`と`table4`はマッピング可能な列の数が同じですが、WHERE句を使用して列ベースのフィルタ条件を指定する必要があります。したがって、`columns`パラメーターを使用して`example4.csv`の列の一時的な名前を定義する必要があります。

#### 宛先パーティションの設定

StarRocksデータベース`test_db`には`table5`という名前のテーブルがあります。このテーブルには、`col1`、`col2`、`col3`の3つの列が含まれています。

データファイル`example5.csv`にも、`table5`の`col1`、`col2`、`col3`に順にマッピングされる3つの列が含まれています。

`example5.csv`のすべてのデータを`table5`のパーティション`p1`と`p2`にロードする場合は、次のコマンドを実行します。

```Bash
curl --location-trusted -u <username>:<password>  -H "label:label5" \
    -H "Expect:100-continue" \
    -H "partitions: p1, p2" \
    -T example5.csv -XPUT \
```
    http://<fe_host>:<fe_http_port>/api/test_db/table5/_stream_load
```

#### 厳格モードとタイムゾーンの設定

お客さまのStarRocksデータベース「test_db」には、「table6」という名前のテーブルが含まれています。このテーブルには3つの列が含まれており、それぞれ`col1`、`col2`、`col3`です。

お客さまのデータファイル`example6.csv`も3つの列で構成されており、これらは`table6`の`col1`、`col2`、`col3`に順にマッピングできます。

`example6.csv`からすべてのデータを`table6`に厳格モードとタイムゾーン`Africa/Abidjan`を使用してロードするには、次のコマンドを実行してください:

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "strict_mode: true" \
    -H "timezone: Africa/Abidjan" \
    -T example6.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table6/_stream_load
```

#### HLL型列を含むテーブルへのデータのロード

お客さまのStarRocksデータベース「test_db」には、「table7」という名前のテーブルが含まれています。このテーブルには2つのHLL型列、つまり`col1`と`col2`が含まれています。

お客さまのデータファイル`example7.csv`も2つの列で構成されていますが、そのうち最初の列は`table7`の`col1`にマッピングできますが、2つめの列は`table7`のどの列にもマッピングできません。`example7.csv`の最初の列の値は、`col1`にロードされる前に関数を使用してHLL型データに変換できます。

`example7.csv`からデータを`table7`にロードしたい場合は、次のコマンドを実行してください:

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "columns: temp1, temp2, col1=hll_hash(temp1), col2=hll_empty()" \
    -T example7.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table7/_stream_load
```

> **注記**
>
> 上記の例では、`example7.csv`の2つの列は`columns`パラメータを使用して順番に`temp1`と`temp2`として指定されます。その後、データを変換するために関数が使用されます:
>
> - `hll_hash`関数は`example7.csv`の`temp1`の値をHLL型データに変換し、`example7.csv`の`temp1`を`table7`の`col1`にマッピングします。
>
> - `hll_empty`関数は、`table7`の`col2`に指定されたデフォルト値を設定します。

`hll_hash`および`hll_empty`関数の使用方法については、[hll_hash](../../sql-functions/aggregate-functions/hll_hash.md)および[hll_empty](../../sql-functions/aggregate-functions/hll_empty.md)を参照してください。

#### BITMAP型列を含むテーブルへのデータのロード

お客さまのStarRocksデータベース「test_db」には、「table8」という名前のテーブルが含まれています。このテーブルには2つのBITMAP型列、つまり`col1`と`col2`が含まれています。

お客さまのデータファイル`example8.csv`も2つの列で構成されていますが、そのうち最初の列は`table8`の`col1`にマッピングできますが、2つめの列は`table8`のどの列にもマッピングできません。`example8.csv`の最初の列の値は、`col1`にロードされる前に関数を使用して変換できます。

`example8.csv`から`table8`にデータをロードしたい場合は、次のコマンドを実行してください:

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "columns: temp1, temp2, col1=to_bitmap(temp1), col2=bitmap_empty()" \
    -T example8.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table8/_stream_load
```

> **注記**
>
> 上記の例では、`example8.csv`の2つの列は`columns`パラメータを使用して順番に`temp1`と`temp2`として指定されます。その後、データを変換するために関数が使用されます:
>
> - `to_bitmap`関数は`example8.csv`の`temp1`の値をBITMAP型データに変換し、`example8.csv`の`temp1`を`table8`の`col1`にマッピングします。
>
> - `bitmap_empty`関数は、`table8`の`col2`に指定されたデフォルト値を設定します。

`to_bitmap`および`bitmap_empty`関数の使用方法については、[to_bitmap](../../../sql-reference/sql-functions/bitmap-functions/to_bitmap.md)および[bitmap_empty](../../../sql-reference/sql-functions/bitmap-functions/bitmap_empty.md)を参照してください。

#### `skip_header`、`trim_space`、`enclose`、および`escape`の設定

お客さまのStarRocksデータベース「test_db」には、「table9」という名前のテーブルが含まれています。このテーブルには3つの列、つまり`col1`、`col2`、`col3`が含まれています。

お客さまのデータファイル`example9.csv`も3つの列で構成されており、これらは`table13`の`col2`、`col1`、`col3`に順にマッピングできます。

`example9.csv`からすべてのデータを`table9`にロードし、`example9.csv`の先頭5行をスキップし、列の区切り文字の前後のスペースを削除し、`enclose`を`\`、`escape`を`\`に設定したい場合は、次のコマンドを実行してください:

```Bash
curl --location-trusted -u <username>:<password> -H "label:3875" \
    -H "Expect:100-continue" \
    -H "trim_space: true" -H "skip_header: 5" \
    -H "column_separator:," -H "enclose:\"" -H "escape:\\" \
    -H "columns: col2, col1, col3" \
    -T example9.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/tbl9/_stream_load
```

### JSONデータのロード

このセクションでは、JSONデータをロードする際に注意する必要があるパラメータ設定について説明します。

お客さまのStarRocksデータベース「test_db」には、「tbl1」という名前のテーブルが含まれており、そのスキーマは次の通りです:

```SQL
`category` varchar(512) NULL COMMENT "",`author` varchar(512) NULL COMMENT "",`title` varchar(512) NULL COMMENT "",`price` double NULL COMMENT ""
```

#### シンプルモードを使用してJSONデータをロード

お客さまのデータファイル`example1.json`が次のデータを含んでいるとします:

```JSON
{"category":"C++","author":"avc","title":"C++ primer","price":895}
```

`example1.json`からすべてのデータを`tbl1`にロードするには、次のコマンドを実行してください:

```Bash
curl --location-trusted -u <username>:<password> -H "label:label6" \
    -H "Expect:100-continue" \
    -H "format: json" \
    -T example1.json -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/tbl1/_stream_load
```

> **注記**
>
> 上記の例では、`columns`および`jsonpaths`パラメータは指定されていません。したがって、`example1.json`のキーは`tbl1`の列に名前でマッピングされます。

スループットを向上させるために、Stream Loadは一度に複数のデータレコードをロードすることをサポートしています。例:

```JSON
[{"category":"C++","author":"avc","title":"C++ primer","price":89.5},{"category":"Java","author":"avc","title":"Effective Java","price":95},{"category":"Linux","author":"avc","title":"Linux kernel","price":195}]
```

#### マッチングモードを使用してJSONデータをロード

StarRocksはJSONデータのマッチングと処理を行うために次の手順を実行します:

1.（任意）`strip_outer_array`パラメータ設定に指示に従って、JSONデータの最外層の配列構造を取り除きます。

   > **注記**
   >
   > この手順は、JSONデータの最外層が角かっこ`[]`で示された配列構造である場合のみ実行されます。`strip_outer_array`を`true`に設定する必要があります。

2.（任意）`json_root`パラメータ設定によって、JSONデータのルート要素をマッチングします。

   > **注記**
   >
   > この手順は、JSONデータにルート要素がある場合にのみ実行されます。`json_root`パラメータを使用してルート要素を指定する必要があります。

3. `jsonpaths`パラメータ設定によって指定されたJSONデータを抽出します。

##### ルート要素が指定されていないマッチングモードを使用してJSONデータをロード

お客さまのデータファイル`example2.json`が次のデータを含んでいるとします:
```JSON
{"category":"xuxb111","author":"1avc","price":895}
{"category":"xuxb222","author":"2avc","price":895}
{"category":"xuxb333","author":"3avc","price":895}
```

`example2.json`から`category`、`author`、`price`のみを読み込むには、次のコマンドを実行します:

```Bash
curl --location-trusted -u <username>:<password> -H "label:label7" \
    -H "Expect:100-continue" \
    -H "format: json" \
    -H "strip_outer_array: true" \
    -H "jsonpaths: [\"$.category\",\"$.price\",\"$.author\"]" \
    -H "columns: category, price, author" \
    -T example2.json -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/tbl1/_stream_load
```

> **注意**
>
> 上記の例では、JSONデータの最も外側の層が角かっこ`[]`で示される配列構造であることが示されています。配列構造は、それぞれがデータレコードを表す複数のJSONオブジェクトで構成されています。したがって、`strip_outer_array`を`true`に設定して最も外側の配列構造を削除する必要があります。読み込み中に読み込みたくない`title`キーは無視されます。

##### ルート要素が指定された一致モードを使用してJSONデータをロードする

データファイル`example3.json`が以下のデータで構成されているとします:

```JSON
{"id": 10001,"RECORDS":[{"category":"11","title":"SayingsoftheCentury","price":895,"timestamp":1589191587},{"category":"22","author":"2avc","price":895,"timestamp":1589191487},{"category":"33","author":"3avc","title":"SayingsoftheCentury","timestamp":1589191387}],"comments": ["3 records", "there will be 3 rows"]}
```

`example3.json`から`category`、`author`、`price`のみを読み込むには、次のコマンドを実行します:

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "format: json" \
    -H "json_root: $.RECORDS" \
    -H "strip_outer_array: true" \
    -H "jsonpaths: [\"$.category\",\"$.price\",\"$.author\"]" \
    -H "columns: category, price, author" -H "label:label8" \
    -T example3.json -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/tbl1/_stream_load
```

> **注意**
>
> 上記の例では、JSONデータの最も外側の層が角かっこ`[]`で示される配列構造であることが示されています。配列構造は、それぞれがデータレコードを表す複数のJSONオブジェクトで構成されています。したがって、`strip_outer_array`を`true`に設定して最も外側の配列構造を削除する必要があります。また、`json_root`パラメータを使用して、JSONデータのルート要素である配列を指定しています。