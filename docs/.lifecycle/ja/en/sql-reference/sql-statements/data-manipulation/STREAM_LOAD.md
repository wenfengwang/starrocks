---
displayed_sidebar: English
---

# STREAM LOAD

## 説明

StarRocksは、ローカルファイルシステムまたはストリーミングデータソースからデータをロードするためのHTTPベースのStream Loadという方法を提供しています。ロードジョブを送信すると、StarRocksはジョブを同期的に実行し、ジョブが完了した後に結果を返します。ジョブの結果に基づいて、ジョブが成功したかどうかを判断できます。Stream Loadの適用シナリオ、制限、原理、およびサポートされるデータファイル形式についての情報は、[ローカルファイルシステムからStream Loadを介してロードする](../../../loading/StreamLoad.md#loading-from-a-local-file-system-via-stream-load)を参照してください。

> **注意**
>
> - Stream Loadを使用してStarRocksテーブルにデータをロードすると、そのテーブルに作成されたマテリアライズドビューのデータも更新されます。
> - StarRocksテーブルにデータをロードするには、そのStarRocksテーブルにINSERT権限を持つユーザーである必要があります。INSERT権限がない場合は、[GRANT](../account-management/GRANT.md)の指示に従って、StarRocksクラスタに接続するために使用するユーザーにINSERT権限を付与してください。

## 構文

```Bash
curl --location-trusted -u <username>:<password> -XPUT <url>
(
    data_desc
)
[opt_properties]        
```

このトピックでは、例としてcurlを使用してStream Loadを使用したデータのロード方法について説明します。curl以外にも、他のHTTP互換ツールや言語を使用してStream Loadを実行することができます。ロード関連のパラメータはHTTPリクエストヘッダフィールドに含まれます。これらのパラメータを入力する際には、以下の点に注意してください：

- このトピックで示されているように、チャンク転送エンコーディングを使用することができます。チャンク転送エンコーディングを選択しない場合は、転送されるコンテンツの長さを示す`Content-Length`ヘッダフィールドを入力する必要があり、これによりデータの整合性が保証されます。

  > **注**
  >
  > curlを使用してStream Loadを実行する場合、StarRocksは自動的に`Content-Length`ヘッダフィールドを追加するため、手動で入力する必要はありません。

- `Expect`ヘッダフィールドを追加し、その値を`100-continue`として指定する必要があります、例えば`"Expect:100-continue"`。これにより、ジョブリクエストが拒否された場合に不要なデータ転送を防ぎ、リソースのオーバーヘッドを削減することができます。

StarRocksでは、いくつかのリテラルがSQL言語によって予約キーワードとして使用されていることに注意してください。これらのキーワードをSQLステートメントで直接使用しないでください。このようなキーワードをSQLステートメントで使用したい場合は、バッククォート(`)で囲んでください。[キーワード](../../../sql-reference/sql-statements/keywords.md)を参照してください。

## パラメータ

### ユーザー名とパスワード

StarRocksクラスタに接続するために使用するアカウントのユーザー名とパスワードを指定します。これは必須のパラメータです。パスワードが設定されていないアカウントを使用する場合は、`<username>:`のみを入力する必要があります。

### XPUT

HTTPリクエストメソッドを指定します。これは必須のパラメータです。Stream LoadはPUTメソッドのみをサポートしています。

### url

StarRocksテーブルのURLを指定します。構文：

```Plain
http://<fe_host>:<fe_http_port>/api/<database_name>/<table_name>/_stream_load
```

以下の表は、URLのパラメータを説明しています。

| パラメータ     | 必須 | 説明                                                  |
| ------------- | -------- | ------------------------------------------------------------ |
| fe_host       | はい      | StarRocksクラスタ内のFEノードのIPアドレス。<br/>**注**<br/>特定のBEノードにロードジョブを送信する場合は、BEノードのIPアドレスを入力する必要があります。 |
| fe_http_port  | はい      | StarRocksクラスタ内のFEノードのHTTPポート番号。デフォルトのポート番号は`8030`です。<br/>**注**<br/>特定のBEノードにロードジョブを送信する場合は、BEノードのHTTPポート番号を入力する必要があります。デフォルトのポート番号は`8030`です。 |
| database_name | はい      | StarRocksテーブルが属するデータベースの名前。 |
| table_name    | はい      | StarRocksテーブルの名前。                             |

> **注**
>
> [SHOW FRONTENDS](../Administration/SHOW_FRONTENDS.md)を使用して、FEノードのIPアドレスとHTTPポートを表示できます。

### data_desc

ロードしたいデータファイルを記述します。`data_desc`記述子には、データファイルの名前、形式、列区切り、行区切り、宛先パーティション、およびStarRocksテーブルに対する列のマッピングを含めることができます。構文：

```Bash
-T <file_path>
-H "format: CSV | JSON"
-H "column_separator: <column_separator>"
-H "row_delimiter: <row_delimiter>"
-H "columns: <column1_name>[, <column2_name>, ... ]"
-H "partitions: <partition1_name>[, <partition2_name>, ...]"
-H "temporary_partitions: <temporary_partition1_name>[, <temporary_partition2_name>, ...]"
-H "jsonpaths: [ \"<json_path1>\"[, \"<json_path2>\", ...] ]"
-H "strip_outer_array: true | false"
-H "json_root: <json_path>"
```

`data_desc`記述子のパラメータは、共通パラメータ、CSVパラメータ、JSONパラメータの3種類に分けられます。

#### 共通パラメータ

| パラメータ  | 必須 | 説明                                                  |
| ---------- | -------- | ------------------------------------------------------------ |
| file_path  | はい      | データファイルの保存パス。必要に応じて、ファイル名の拡張子を含めることができます。 |
| format     | いいえ       | データファイルの形式。有効な値は`CSV`と`JSON`です。デフォルト値は`CSV`です。 |
| partitions | いいえ       | データファイルをロードするパーティション。デフォルトでは、このパラメータを指定しない場合、StarRocksはテーブルのすべてのパーティションにデータファイルをロードします。 |
| temporary_partitions|  いいえ       | データファイルをロードする[一時パーティション](../../../table_design/Temporary_partition.md)の名前。複数の一時パーティションを指定する場合は、コンマ(,)で区切る必要があります。|
| columns    | いいえ       | データファイルとStarRocksテーブル間の列のマッピング。<br/>データファイルのフィールドがStarRocksテーブルの列に順番にマッピングできる場合、このパラメータを指定する必要はありません。このパラメータを使用してデータ変換を実装することもできます。例えば、CSVデータファイルをロードし、そのファイルが2つの列で構成されており、StarRocksテーブルの`id`と`city`の2つの列に順番にマッピングできる場合、`"columns: city,tmp_id, id = tmp_id * 100"`と指定できます。詳細については、このトピックの「[列のマッピング](#column-mapping)」セクションを参照してください。 |

#### CSVパラメータ

| パラメータ        | 必須 | 説明                                                  |
| ---------------- | -------- | ------------------------------------------------------------ |

| column_separator | いいえ       | データファイルでフィールドを区切るために使用される文字。このパラメーターを指定しない場合、デフォルトでは `\t`（タブ）になります。<br/>このパラメーターで指定する列区切り文字がデータファイルで使用されている列区切り文字と同じであることを確認してください。<br/>**注記**<br/>CSVデータの場合、コンマ(,)、タブ、パイプ(\|)など、50バイトを超えないUTF-8文字列をテキスト区切り文字として使用できます。 |
| row_delimiter    | いいえ       | データファイルで行を区切るために使用される文字。このパラメーターを指定しない場合、デフォルト値は `\n` です。 |
| skip_header      | いいえ       | データファイルがCSV形式の場合、データファイルの最初の数行をスキップするかどうかを指定します。タイプ: INTEGER。デフォルト値: `0`。<br />一部のCSV形式のデータファイルでは、最初の数行を使用して列名や列データ型などのメタデータを定義します。`skip_header` パラメーターを設定することで、StarRocksはデータロード中にデータファイルの最初の数行をスキップすることができます。たとえば、このパラメーターを `1` に設定した場合、StarRocksはデータロード中にデータファイルの最初の行をスキップします。<br />データファイルの最初の数行は、ロードコマンドで指定した行区切り文字を使用して区切られている必要があります。 |
| trim_space       | いいえ       | データファイルがCSV形式の場合、列区切り文字の前後のスペースを削除するかどうかを指定します。タイプ: BOOLEAN。デフォルト値: `false`。<br />一部のデータベースでは、CSV形式のデータファイルをエクスポートする際に列区切り文字の前後にスペースが追加されます。このようなスペースは、その位置に応じて先頭スペースまたは末尾スペースと呼ばれます。`trim_space` パラメーターを設定することで、StarRocksはデータロード中にこれらの不要なスペースを削除できます。<br />ただし、StarRocksは`enclose`で指定された文字で囲まれたフィールド内のスペース（先頭スペースと末尾スペースを含む）を削除しません。例えば、以下のフィールド値ではパイプ(<code class="language-text">&#124;</code>)を列区切り文字として、ダブルクォーテーション(`"`)を`enclose`で指定された文字として使用しています：<br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> <br /><code class="language-text">&#124;" Love StarRocks "&#124;</code> <br /><code class="language-text">&#124; "Love StarRocks" &#124;</code> <br />`trim_space`を`true`に設定すると、StarRocksは上記のフィールド値を次のように処理します：<br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> <br /><code class="language-text">&#124;" Love StarRocks "&#124;</code> <br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> |
| enclose          | いいえ       | データファイルがCSV形式の場合、[RFC4180](https://www.rfc-editor.org/rfc/rfc4180)に従ってフィールド値を囲むために使用する文字を指定します。タイプ: 単一バイト文字。デフォルト値: `NONE`。一般的な文字はシングルクォート(`'`)とダブルクォート(`"`)です。<br />`enclose`で指定された文字で囲まれたすべての特殊文字（行区切り文字と列区切り文字を含む）は通常の記号と見なされます。StarRocksはRFC4180以上の機能を提供し、任意の単一バイト文字を`enclose`で指定された文字として使用できます。<br />フィールド値に`enclose`で指定された文字が含まれている場合、その`enclose`で指定された文字をエスケープするために同じ文字を使用できます。例えば、`enclose`を`"`に設定し、フィールド値が`a "quoted" c`である場合、データファイルには`"a ""quoted"" c"`として入力できます。 |
| escape           | いいえ       | 行区切り文字、列区切り文字、エスケープ文字、`enclose`で指定された文字など、さまざまな特殊文字をエスケープするために使用される文字を指定します。これらの文字はStarRocksによって通常の文字と見なされ、それらが存在するフィールド値の一部として解析されます。タイプ: 単一バイト文字。デフォルト値: `NONE`。一般的な文字はバックスラッシュ(`\`)で、SQLステートメントではダブルバックスラッシュ(`\\`)として記述する必要があります。<br />**注記**<br />`escape`で指定された文字は、`enclose`で指定された文字のペアの内外の両方に適用されます。<br />例えば、`enclose`を`"`に、`escape`を`\`に設定した場合、StarRocksは`"say \"Hello world\""`を`say "Hello world"`として解析します。列区切り文字がコンマ(`,`)である場合、`escape`を`\`に設定すると、StarRocksは`a, b\, c`を`a`と`b, c`の2つのフィールド値に解析します。</li></ul> |

> **注記**
>
> - CSVデータの場合、コンマ(,)、タブ、パイプ(\|)などのUTF-8文字列をテキスト区切り文字として使用できます。その長さは50バイトを超えてはなりません。
> - Null値は`\N`を使用して表されます。例えば、データファイルが3つの列で構成されており、そのデータファイルのレコードが1列目と3列目にデータを保持しているが2列目にはデータがない場合、2列目に`\N`を使用してnull値を示す必要があります。つまり、レコードは`a,\N,b`としてコンパイルされる必要があり、`a,,b`ではなく、`a,,b`は2列目のレコードが空文字列を保持していることを示します。
> - `skip_header`、`trim_space`、`enclose`、`escape`などのフォーマットオプションは、v3.0以降でサポートされています。

#### JSONパラメーター

| パラメーター         | 必須 | 説明                                                  |
| ----------------- | -------- | ------------------------------------------------------------ |
| jsonpaths         | いいえ       | JSONデータファイルからロードしたいキーの名前。このパラメーターは、マッチモードを使用してJSONデータをロードする場合のみ指定する必要があります。このパラメーターの値はJSON形式です。[JSONデータロードのための列マッピングの設定](#configure-column-mapping-for-json-data-loading)を参照してください。           |
| strip_outer_array | いいえ       | 最も外側の配列構造を取り除くかどうかを指定します。有効な値: `true`、`false`。デフォルト値: `false`。<br/>実際のビジネスシナリオでは、JSONデータが角括弧`[]`で示される最も外側の配列構造を持つことがあります。この場合、このパラメーターを`true`に設定することで、StarRocksは最も外側の角括弧`[]`を取り除き、各内部配列を個別のデータレコードとしてロードします。このパラメーターを`false`に設定すると、StarRocksはJSONデータファイル全体を1つの配列として解析し、その配列を単一のデータレコードとしてロードします。<br/>例えば、JSONデータが`[ {"category" : 1, "author" : 2}, {"category" : 3, "author" : 4} ]`の場合、このパラメーターを`true`に設定すると、`{"category" : 1, "author" : 2}`と`{"category" : 3, "author" : 4}`は個別のデータレコードとして解析され、StarRocksの別々のテーブル行にロードされます。 |

| json_root         | いいえ       |JSONデータファイルからロードするJSONデータのルート要素。このパラメータは、一致モードを使用してJSONデータをロードする場合にのみ指定する必要があります。このパラメータの値は有効なJsonPath文字列です。デフォルトでは、このパラメータの値は空で、JSONデータファイルの全データがロードされることを意味します。詳細は、このトピックの「[ルート要素を指定した一致モードでのJSONデータのロード](#load-json-data-using-matched-mode-with-root-element-specified)」セクションを参照してください。 |
| ignore_json_size  | いいえ       | HTTPリクエストのJSON本体のサイズをチェックするかどうかを指定します。<br/>**注記**<br/>デフォルトでは、HTTPリクエストのJSON本体のサイズは100MBを超えることはできません。JSON本体が100MBを超えるサイズの場合、エラー「このバッチのサイズがjson型データの最大サイズ[104857600]を超えています[8617627793]。ignore_json_sizeを設定してチェックをスキップしますが、大量のメモリを消費する可能性があります。」が報告されます。このエラーを防ぐために、HTTPリクエストヘッダーに`"ignore_json_size:true"`を追加し、StarRocksにJSON本体のサイズをチェックしないよう指示することができます。 |

JSONデータをロードする際には、JSONオブジェクトごとのサイズが4GBを超えないように注意してください。JSONデータファイル内の個別のJSONオブジェクトのサイズが4GBを超える場合、「このパーサーはそのような大きなドキュメントをサポートできません」というエラーが報告されます。

### opt_properties

ロードジョブ全体に適用されるオプションのパラメータを指定します。構文：

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

以下の表は、オプションのパラメータについて説明しています。

| パラメータ        | 必須 | 説明                                                  |
| ---------------- | -------- | ------------------------------------------------------------ |
| label            | いいえ       | ロードジョブのラベル。このパラメータを指定しない場合、StarRocksはロードジョブに自動的にラベルを生成します。<br/>StarRocksは、同じラベルを使ってデータバッチを複数回ロードすることを許可しません。そのため、同じデータが繰り返しロードされることを防ぎます。ラベルの命名規約については、[システム制限](../../../reference/System_limit.md)を参照してください。<br/>デフォルトでは、StarRocksは直近3日間に成功したロードジョブのラベルを保持します。ラベル保持期間を変更するには、[FEパラメータ](../../../administration/FE_configuration.md) `label_keep_max_second`を使用できます。 |
| where            | いいえ       | StarRocksが前処理データをフィルタリングする条件。StarRocksは、WHERE句で指定されたフィルタ条件を満たす前処理データのみをロードします。 |
| max_filter_ratio | いいえ       | ロードジョブの最大エラー許容率。エラー許容率は、ロードジョブに要求された全データレコード中で、データ品質が不十分によりフィルタリングされる可能性のあるデータレコードの最大割合です。有効な値: `0`から`1`。デフォルト値: `0`。<br/>デフォルト値`0`を保持することを推奨します。これにより、資格のないデータレコードが検出された場合、ロードジョブは失敗し、データの正確性が保証されます。<br/>資格のないデータレコードを無視したい場合は、このパラメータを`0`より大きい値に設定できます。これにより、データファイルに資格のないデータレコードが含まれていても、ロードジョブが成功することができます。<br/>**注記**<br/>資格のないデータレコードには、WHERE句によってフィルタリングされたデータレコードは含まれません。 |
| log_rejected_record_num | いいえ           | ログに記録される資格のないデータ行の最大数を指定します。このパラメータはv3.1以降でサポートされています。有効な値: `0`、`-1`、および0以外の正の整数。デフォルト値: `0`。<ul><li>`0`は、フィルタリングされたデータ行がログに記録されないことを指定します。</li><li>`-1`は、フィルタリングされたすべてのデータ行がログに記録されることを指定します。</li><li>0以外の正の整数（例えば`n`）は、各BEでフィルタリングされた最大`n`データ行がログに記録されることを指定します。</li></ul> |
| timeout          | いいえ       | ロードジョブのタイムアウト期間。有効な値: `1`から`259200`。単位: 秒。デフォルト値: `600`。<br/>**注記** この`timeout`パラメータに加えて、[FEパラメータ](../../../administration/FE_configuration.md) `stream_load_default_timeout_second`を使用して、StarRocksクラスタ内のすべてのストリームロードジョブのタイムアウト期間を一元管理することもできます。`timeout`パラメータを指定すると、そのパラメータによって指定されたタイムアウト期間が優先されます。`timeout`パラメータを指定しない場合、`stream_load_default_timeout_second`パラメータによって指定されたタイムアウト期間が優先されます。 |
| strict_mode      | いいえ       | [strictモード](../../../loading/load_concept/strict_mode.md)を有効にするかどうかを指定します。有効な値: `true`または`false`。デフォルト値: `false`。`true`はstrictモードを有効にし、`false`はstrictモードを無効にします。 |
| timezone         | いいえ       | ロードジョブに使用されるタイムゾーン。デフォルト値: `Asia/Shanghai`。このパラメータの値は、strftime、alignment_timestamp、from_unixtimeなどの関数によって返される結果に影響します。このパラメータで指定されるタイムゾーンはセッションレベルのタイムゾーンです。詳細は、[タイムゾーンの設定](../../../administration/timezone.md)を参照してください。 |
| load_mem_limit   | いいえ       | ロードジョブに割り当て可能なメモリの最大量。単位: バイト。デフォルトでは、ロードジョブの最大メモリサイズは2GBです。このパラメータの値は、各BEに割り当て可能なメモリの最大量を超えてはなりません。 |
| merge_condition  | いいえ       | 更新が有効になる条件として使用する列の名前を指定します。ソースレコードから宛先レコードへの更新は、ソースデータレコードが指定された列で宛先データレコードよりも大きいか等しい値を持つ場合にのみ有効です。StarRocksはv2.5以降、条件付き更新をサポートしています。詳細は、[ロードを通じたデータの変更](../../../loading/Load_to_Primary_Key_tables.md)を参照してください。<br/>**注記**<br/>指定する列は主キー列であってはなりません。また、条件付き更新はプライマリキーテーブルを使用するテーブルのみがサポートしています。|

## 列マッピング

### CSVデータロードの列マッピングを設定する

データファイルの列がStarRocksテーブルの列に一対一で順番にマッピングできる場合、データファイルとStarRocksテーブル間の列マッピングを設定する必要はありません。


データファイルの列をStarRocksテーブルの列に1対1で順番にマッピングできない場合は、`columns`パラメーターを使用して、データファイルとStarRocksテーブル間の列マッピングを構成する必要があります。これには、次の2つのユースケースが含まれます。

- **列数は同じですが、列の順序が異なります。** **また、データファイルからのデータは、StarRocksテーブルの列にロードされる前に関数によって計算する必要はありません。**

  `columns`パラメーターでは、データファイルの列の並び順と同じ順序でStarRocksテーブルの列名を指定する必要があります。

  例えば、StarRocksテーブルは`col1`、`col2`、`col3`の順に3つの列で構成され、データファイルも3つの列で構成されており、それらはStarRocksテーブルの`col3`、`col2`、`col1`の順にマッピングできます。この場合、`"columns: col3, col2, col1"`と指定する必要があります。

- **列数が異なり、列の順序も異なります。また、データファイルからのデータは、StarRocksテーブルの列にロードされる前に関数によって計算される必要があります。**

  `columns`パラメーターでは、データファイルの列の並び順と同じ順序でStarRocksテーブルの列名を指定し、データを計算するために使用する関数も指定する必要があります。以下に2つの例を示します。

  - StarRocksテーブルは`col1`、`col2`、`col3`の順に3つの列で構成されています。データファイルは4つの列で構成されており、最初の3つの列はStarRocksテーブルの`col1`、`col2`、`col3`に順番にマッピングできますが、4番目の列はStarRocksテーブルのどの列にもマッピングできません。この場合、データファイルの4番目の列に一時的な名前を指定する必要があり、その一時的な名前はStarRocksテーブルの列名とは異なるものでなければなりません。例えば、`"columns: col1, col2, col3, temp"`と指定することで、データファイルの4番目の列に一時的な名前`temp`を付けることができます。
  - StarRocksテーブルは`year`、`month`、`day`の順に3つの列で構成されています。データファイルは`yyyy-mm-dd hh:mm:ss`形式の日付と時刻の値を格納する1つの列のみで構成されています。この場合、`"columns: col, year = year(col), month=month(col), day=day(col)"`と指定することができます。ここで`col`はデータファイルの列の一時名であり、関数`year = year(col)`、`month=month(col)`、`day=day(col)`はデータファイルの列`col`からデータを抽出し、それをStarRocksテーブルの対応する列にロードするために使用されます。例えば、`year = year(col)`はデータファイルの列`col`から`yyyy`のデータを抽出し、StarRocksテーブルの`year`列にロードするために使用されます。

詳細な例については、[列マッピングの構成](#configure-column-mapping)を参照してください。

### JSONデータのロード時の列マッピングの構成

JSONドキュメントのキーがStarRocksテーブルの列名と同じ場合、シンプルモードを使用してJSON形式のデータをロードできます。シンプルモードでは、`jsonpaths`パラメーターを指定する必要はありません。このモードでは、JSON形式のデータは中括弧`{}`で示されるオブジェクトである必要があります。例えば、`{"category": 1, "author": 2, "price": "3"}`のようになります。この例では、`category`、`author`、`price`はキー名であり、これらのキーはStarRocksテーブルの`category`、`author`、`price`列に名前で1対1にマッピングできます。

JSONドキュメントのキー名がStarRocksテーブルの列名と異なる場合、マッチモードを使用してJSON形式のデータをロードできます。マッチモードでは、`jsonpaths`および`COLUMNS`パラメーターを使用して、JSONドキュメントとStarRocksテーブル間の列マッピングを指定する必要があります。

- `jsonpaths`パラメーターでは、JSONドキュメント内のキーをその配置順に指定します。
- `COLUMNS`パラメーターでは、JSONキーとStarRocksテーブルの列とのマッピングを指定します：
  - `COLUMNS`パラメーターで指定された列名は、JSONキーに対して順番に1対1でマッピングされます。
  - `COLUMNS`パラメーターで指定された列名は、StarRocksテーブルの列に対して名前で1対1でマッピングされます。

マッチモードを使用してJSON形式のデータをロードする例については、[マッチモードを使用したJSONデータのロード](#load-json-data-using-matched-mode)を参照してください。

## 戻り値

ロードジョブが終了すると、StarRocksはジョブ結果をJSON形式で返します。例：

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
    "StreamLoadPlanTimeMs": 1,
    "ReadDataTimeMs": 0,
    "WriteDataTimeMs": 11,
    "CommitAndPublishTimeMs": 16,
}
```

次の表は、返されるジョブ結果のパラメーターを説明しています。

| パラメーター              | 説明                                                  |
| ---------------------- | ------------------------------------------------------------ |
| TxnId                  | ロードジョブのトランザクションID。                          |
| Label                  | ロードジョブのラベル。                                   |
| Status                 | ロードされたデータの最終ステータス。<ul><li>`Success`: データが正常にロードされ、クエリ可能です。</li><li>`Publish Timeout`: ロードジョブは正常に送信されましたが、データはまだクエリできません。データのロードを再試行する必要はありません。</li><li>`Label Already Exists`: ロードジョブのラベルは別のロードジョブで使用されています。データは既に正常にロードされているか、ロード中の可能性があります。</li><li>`Fail`: データのロードに失敗しました。ロードジョブを再試行できます。</li></ul> |
| Message                | ロードジョブの状況。ロードジョブが失敗した場合、詳細な失敗原因が返されます。 |
| NumberTotalRows        | 読み取られたデータレコードの総数。              |
| NumberLoadedRows       | 正常にロードされたデータレコードの総数。このパラメーターは、`Status`の戻り値が`Success`の場合にのみ有効です。 |
| NumberFilteredRows     | データ品質が不十分でフィルタリングされたデータレコードの数。 |
| NumberUnselectedRows   | WHERE句によってフィルタリングされたデータレコードの数。 |
| LoadBytes              | ロードされたデータの量。単位はバイトです。              |
| LoadTimeMs             | ロードジョブにかかった時間。単位はミリ秒です。  |
| BeginTxnTimeMs         | ロードジョブのトランザクションを実行するのにかかった時間。 |
| StreamLoadPlanTimeMs   | ロードジョブの実行計画を生成するのにかかった時間。 |
| ReadDataTimeMs         | ロードジョブのデータ読み取りにかかった時間。 |
| WriteDataTimeMs        | ロードジョブのデータ書き込みにかかった時間。 |
| CommitAndPublishTimeMs | ロードジョブのデータコミットと公開にかかった時間。 |

ロードジョブが失敗した場合、StarRocksは`ErrorURL`も返します。例：

```JSON
{
    "ErrorURL": "http://example.com/error_log"
}
```
{"ErrorURL": "http://172.26.195.68:8045/api/_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be"}
```

`ErrorURL`は、フィルタリングで除外された非適格データレコードの詳細を取得できるURLを提供します。非適格データ行をログに記録する最大数を指定するには、ロードジョブをサブミットする際に設定されるオプショナルパラメータ`log_rejected_record_num`を使用します。

`curl "url"`を実行すると、フィルタリングで除外された非適格データレコードの詳細を直接閲覧できます。また、`wget "url"`を実行してこれらのデータレコードの詳細をエクスポートすることもできます：

```Bash
wget http://172.26.195.68:8045/api/_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be
```

エクスポートされたデータレコードの詳細は、`_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be`という名前のローカルファイルに保存されます。`cat`コマンドを使用してファイルの内容を確認できます。

その後、ロードジョブの設定を調整し、ロードジョブを再度サブミットします。

## 例

### CSVデータのロード

このセクションでは、CSVデータを例に、さまざまなパラメータ設定と組み合わせを用いて、多様なロード要件を満たす方法について説明します。

#### タイムアウト期間の設定

StarRocksデータベース`test_db`には`table1`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列から成り立っています。

データファイル`example1.csv`も3つの列から成り立っており、`table1`の`col1`、`col2`、`col3`に順番にマッピングできます。

`example1.csv`からすべてのデータを`table1`に最大100秒以内でロードしたい場合、以下のコマンドを実行します：

```Bash
curl --location-trusted -u <username>:<password> -H "label:label1" \
    -H "Expect:100-continue" \
    -H "timeout:100" \
    -H "max_filter_ratio:0.2" \
    -T example1.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
```

#### 許容誤差の設定

StarRocksデータベース`test_db`には`table2`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列から成り立っています。

データファイル`example2.csv`も3つの列から成り立っており、`table2`の`col1`、`col2`、`col3`に順番にマッピングできます。

`example2.csv`から`table2`へのデータロード時に最大許容誤差を`0.2`に設定したい場合、以下のコマンドを実行します：

```Bash
curl --location-trusted -u <username>:<password> -H "label:label2" \
    -H "Expect:100-continue" \
    -H "max_filter_ratio:0.2" \
    -T example2.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table2/_stream_load
```

#### 列マッピングの設定

StarRocksデータベース`test_db`には`table3`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列から成り立っています。

データファイル`example3.csv`も3つの列から成り立っており、`table3`の`col2`、`col1`、`col3`に順番にマッピングできます。

`example3.csv`から`table3`へのデータをロードしたい場合、以下のコマンドを実行します：

```Bash
curl --location-trusted -u <username>:<password>  -H "label:label3" \
    -H "Expect:100-continue" \
    -H "columns: col2, col1, col3" \
    -T example3.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table3/_stream_load
```

> **注記**
>
> 上記の例では、`example3.csv`の列を`table3`の列と同じ順序でマッピングすることはできません。したがって、`columns`パラメータを使用して`example3.csv`と`table3`の間の列マッピングを設定する必要があります。

#### フィルタ条件の設定

StarRocksデータベース`test_db`には`table4`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列から成り立っています。

データファイル`example4.csv`も3つの列から成り立っており、`table4`の`col1`、`col2`、`col3`に順番にマッピングできます。

`example4.csv`の最初の列の値が`20180601`に等しいデータレコードのみを`table4`にロードしたい場合、以下のコマンドを実行します：

```Bash
curl --location-trusted -u <username>:<password> -H "label:label4" \
    -H "Expect:100-continue" \
    -H "columns: col1, col2, col3" \
    -H "where: col1 = 20180601" \
    -T example4.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table4/_stream_load
```

> **注記**
>
> 上記の例では、`example4.csv`と`table4`は順番にマッピングできる列の数が同じですが、列ベースのフィルタ条件を指定するためにWHERE句を使用する必要があります。したがって、`columns`パラメータを使用して`example4.csv`の列に一時的な名前を定義する必要があります。

#### 宛先パーティションの設定

StarRocksデータベース`test_db`には`table5`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列から成り立っています。

データファイル`example5.csv`も3つの列から成り立っており、`table5`の`col1`、`col2`、`col3`に順番にマッピングできます。

`example5.csv`から`table5`のパーティション`p1`と`p2`にデータをロードしたい場合、以下のコマンドを実行します：

```Bash
curl --location-trusted -u <username>:<password>  -H "label:label5" \
    -H "Expect:100-continue" \
    -H "partitions: p1, p2" \
    -T example5.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table5/_stream_load
```

#### 厳密モードとタイムゾーンの設定

StarRocksデータベース`test_db`には`table6`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列から成り立っています。

データファイル`example6.csv`も3つの列から成り立っており、`table6`の`col1`、`col2`、`col3`に順番にマッピングできます。

厳密モードとタイムゾーン`Africa/Abidjan`を使用して`example6.csv`から`table6`へのデータをロードしたい場合、以下のコマンドを実行します：

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "strict_mode: true" \
    -H "timezone: Africa/Abidjan" \
    -T example6.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table6/_stream_load
```

#### HLL型の列を含むテーブルへのデータロード

StarRocksデータベース`test_db`には`table7`という名前のテーブルが含まれています。このテーブルは、`col1`と`col2`の2つのHLL型列で構成されています。

データファイル`example7.csv`も2つの列から成り立っており、最初の列は`table7`の`col1`にマッピングでき、2番目の列は`table7`のどの列にもマッピングできません。`example7.csv`の最初の列の値は、`table7`の`col1`にロードする前に関数を使用してHLL型データに変換できます。

`example7.csv`から`table7`へのデータをロードしたい場合、以下のコマンドを実行します：

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "columns: temp1, temp2, col1=hll_hash(temp1), col2=hll_empty()" \
    -T example7.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table7/_stream_load
    http://<fe_host>:<fe_http_port>/api/test_db/table7/_stream_load
```

> **注記**
>
> 上記の例では、`example7.csv`の2つの列が`columns`パラメータを使用して順に`temp1`と`temp2`と名付けられています。次に、以下のようにデータを変換するために関数が使用されます：
>
> - `hll_hash`関数は、`example7.csv`の`temp1`の値をHLL型データに変換し、`example7.csv`の`temp1`を`table7`の`col1`にマッピングするために使用されます。
>
> - `hll_empty`関数は、`table7`の`col2`に指定されたデフォルト値を埋めるために使用されます。

関数`hll_hash`と`hll_empty`の使用方法については、[hll_hash](../../sql-functions/aggregate-functions/hll_hash.md)と[hll_empty](../../sql-functions/aggregate-functions/hll_empty.md)を参照してください。

#### BITMAP型の列を含むテーブルにデータをロードする

あなたのStarRocksデータベース`test_db`には`table8`という名前のテーブルが含まれています。このテーブルは、順に`col1`と`col2`という2つのBITMAP型の列で構成されています。

あなたのデータファイル`example8.csv`も2つの列で構成されており、最初の列は`table8`の`col1`にマッピングでき、2番目の列は`table8`のどの列にもマッピングできません。`example8.csv`の最初の列の値は、`table8`の`col1`にロードする前に関数を使用して変換できます。

`example8.csv`から`table8`へデータをロードしたい場合は、以下のコマンドを実行します：

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "columns: temp1, temp2, col1=to_bitmap(temp1), col2=bitmap_empty()" \
    -T example8.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table8/_stream_load
```

> **注記**
>
> 上記の例では、`example8.csv`の2つの列が`columns`パラメータを使用して順に`temp1`と`temp2`と名付けられています。次に、以下のようにデータを変換するために関数が使用されます：
>
> - `to_bitmap`関数は、`example8.csv`の`temp1`の値をBITMAP型データに変換し、`example8.csv`の`temp1`を`table8`の`col1`にマッピングするために使用されます。
>
> - `bitmap_empty`関数は、`table8`の`col2`に指定されたデフォルト値を埋めるために使用されます。

関数`to_bitmap`と`bitmap_empty`の使用方法については、[to_bitmap](../../../sql-reference/sql-functions/bitmap-functions/to_bitmap.md)と[bitmap_empty](../../../sql-reference/sql-functions/bitmap-functions/bitmap_empty.md)を参照してください。

#### `skip_header`、`trim_space`、`enclose`、`escape`の設定

あなたのStarRocksデータベース`test_db`には`table9`という名前のテーブルが含まれています。このテーブルは、順に`col1`、`col2`、`col3`という3つの列で構成されています。

あなたのデータファイル`example9.csv`も3つの列で構成されており、それぞれ`table9`の`col2`、`col1`、`col3`に順にマッピングされます。

`example9.csv`から`table9`へすべてのデータをロードしたい場合、`example9.csv`の最初の5行をスキップし、列セパレータの前後のスペースを削除し、`enclose`を`"`に、`escape`を`\`に設定するには、以下のコマンドを実行します：

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

このセクションでは、JSONデータをロードする際に注意すべきパラメータ設定について説明します。

StarRocksデータベース`test_db`には、`tbl1`という名前のテーブルがあり、そのスキーマは以下の通りです：

```SQL
`category` varchar(512) NULL COMMENT "", `author` varchar(512) NULL COMMENT "", `title` varchar(512) NULL COMMENT "", `price` double NULL COMMENT ""
```

#### シンプルモードを使用したJSONデータのロード

あなたのデータファイル`example1.json`が以下のデータで構成されているとします：

```JSON
{"category":"C++","author":"avc","title":"C++ primer","price":895}
```

`example1.json`から`tbl1`へすべてのデータをロードするには、以下のコマンドを実行します：

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

スループットを向上させるために、Stream Loadは一度に複数のデータレコードをロードすることをサポートしています。例えば：

```JSON
[{"category":"C++","author":"avc","title":"C++ primer","price":89.5},{"category":"Java","author":"avc","title":"Effective Java","price":95},{"category":"Linux","author":"avc","title":"Linux kernel","price":195}]
```

#### マッチモードを使用したJSONデータのロード

StarRocksは、以下の手順でJSONデータをマッチングして処理します：

1. (オプション) `strip_outer_array`パラメータの設定に従って、最も外側の配列構造を剥がします。

   > **注記**
   >
   > この手順は、JSONデータの最外層が角括弧`[]`で示される配列構造である場合にのみ実行されます。`strip_outer_array`を`true`に設定する必要があります。

2. (オプション) `json_root`パラメータの設定に従って、JSONデータのルート要素をマッチングします。

   > **注記**
   >
   > この手順は、JSONデータにルート要素がある場合にのみ実行されます。`json_root`パラメータを使用してルート要素を指定する必要があります。

3. `jsonpaths`パラメータの設定に従って、指定されたJSONデータを抽出します。

##### ルート要素を指定せずにマッチモードを使用してJSONデータをロードする

あなたのデータファイル`example2.json`が以下のデータで構成されているとします：

```JSON
[{"category":"xuxb111","author":"1avc","title":"SayingsoftheCentury","price":895},{"category":"xuxb222","author":"2avc","title":"SayingsoftheCentury","price":895},{"category":"xuxb333","author":"3avc","title":"SayingsoftheCentury","price":895}]
```

`example2.json`から`category`、`author`、`price`のみをロードするには、以下のコマンドを実行します：

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

> **注記**
>
> 上記の例では、JSONデータの最外層が角括弧`[]`で示される配列構造です。配列構造は、それぞれがデータレコードを表す複数のJSONオブジェクトで構成されています。したがって、最外層の配列構造を剥がすために`strip_outer_array`を`true`に設定する必要があります。ロードしたくないキー**title**はロード中に無視されます。

##### ルート要素が指定されたマッチモードを使用してJSONデータをロードする

あなたのデータファイル`example3.json`が以下のデータで構成されているとします：

```JSON

{"id": 10001,"RECORDS":[{"category":"11","title":"SayingsoftheCentury","price":895,"timestamp":1589191587},{"category":"22","author":"2avc","price":895,"timestamp":1589191487},{"category":"33","author":"3avc","title":"SayingsoftheCentury","timestamp":1589191387}],"comments": ["3 records", "there will be 3 rows"]}
```

`example3.json`から`category`、`author`、`price`のみをロードするには、次のコマンドを実行します。

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect: 100-continue" \
    -H "Content-Type: application/json" \
    -H "json_root: $.RECORDS" \
    -H "strip_outer_array: true" \
    -H "jsonpaths: [\"$.category\",\"$.price\",\"$.author\"]" \
    -H "columns: category, price, author" -H "label: label8" \
    -T example3.json -X PUT \
    http://<fe_host>:<fe_http_port>/api/test_db/tbl1/_stream_load
```

> **注記**
>
> 上記の例では、JSONデータの最外層が角括弧`[]`で示される配列構造であることがわかります。この配列構造は、それぞれがデータレコードを表す複数のJSONオブジェクトで構成されています。そのため、最外層の配列構造を取り除くためには`strip_outer_array`を`true`に設定する必要があります。ロードしたくないキー`title`と`timestamp`はロード中に無視されます。さらに、`json_root`パラメータはJSONデータのルート要素（配列）を指定するために使用されます。
