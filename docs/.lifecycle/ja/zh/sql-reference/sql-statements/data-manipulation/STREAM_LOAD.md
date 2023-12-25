---
displayed_sidebar: Chinese
---

# STREAM LOAD

## 機能

Stream LoadはHTTPプロトコルに基づく同期的なインポート方法で、ローカルファイルやデータストリームをStarRocksにインポートすることをサポートしています。インポートジョブを送信した後、StarRocksはインポートジョブを同期的に実行し、インポートジョブの結果情報を返します。返された結果情報を通じて、インポートジョブが成功したかどうかを判断できます。Stream Loadの適用シナリオ、使用制限、基本原理、サポートされるデータファイル形式などの情報については、[ローカルからStream Loadを使用してインポートする](../../../loading/StreamLoad.md#ローカルから-stream-load-を使用してインポートする)を参照してください。

> **注意**
>
> - Stream Load操作は、StarRocksの元のテーブルに関連するマテリアライズドビューのデータも同時に更新します。
> - Stream Load操作には、対象テーブルのINSERT権限が必要です。ユーザーアカウントにINSERT権限がない場合は、[GRANT](../account-management/GRANT.md)を参照してユーザーに権限を付与してください。

## 文法

```Bash
curl --location-trusted -u <username>:<password> -XPUT <url>
(
    data_desc
)
[opt_properties]        
```

この記事では、curlツールを例にして、Stream Loadを使用してデータをインポートする方法を紹介します。curlツールの使用に限らず、HTTPプロトコルをサポートする他のツールや言語を使用してインポートジョブを送信してデータをインポートすることもできます。インポートに関連するパラメータはHTTPリクエストのヘッダに位置しています。これらのインポート関連のパラメータを送信する際には、以下の点に注意が必要です：

- この記事の例に示されているように、**チャンクアップロード**方式を推奨します。**非チャンクアップロード**方式を使用する場合は、アップロードする内容の長さを示すためにリクエストヘッダの`Content-Length`フィールドを使用する必要があり、これによりデータの完全性を保証します。

  > **説明**
  >
  > curlツールを使用してインポートジョブを送信する際には、`Content-Length`フィールドが自動的に追加されるため、手動で`Content-Length`を指定する必要はありません。

- HTTPリクエストのヘッダフィールド`Expect`で`100-continue`を指定する必要があります。つまり、`"Expect:100-continue"`です。これにより、サーバーがインポートジョブのリクエストを拒否した場合に不要なデータ転送を避け、不要なリソース消費を減らすことができます。

StarRocksでは、一部のテキストはSQL言語の予約キーワードであり、直接SQL文で使用することはできません。これらの予約キーワードをSQL文で使用したい場合は、バッククォート(`)で囲む必要があります。[キーワード](../../../sql-reference/sql-statements/keywords.md)を参照してください。

## パラメータ説明

### usernameとpassword

StarRocksクラスタのアカウントのユーザー名とパスワードを指定するために使用します。必須パラメータです。アカウントにパスワードが設定されていない場合は、`<username>:`のみを入力する必要があります。

### XPUT

HTTPリクエストメソッドを指定するために使用します。必須パラメータです。Stream Loadは現在PUTメソッドのみをサポートしています。

### url

StarRocksテーブルのURLアドレスを指定するために使用します。必須パラメータです。以下のような構文です：

```Plain
http://<fe_host>:<fe_http_port>/api/<database_name>/<table_name>/_stream_load
```

`url`のパラメータは以下の表の通りです。

| パラメータ名      | 必須 | 説明                                                     |
| ------------- | ---- | ------------------------------------------------------------ |
| fe_host       | はい | StarRocksクラスタ内のFEのIPアドレスを指定します。<br />**説明**<br />インポートジョブを直接BEノードに送信する場合は、そのBEのIPアドレスを入力する必要があります。 |
| fe_http_port  | はい | StarRocksクラスタ内のFEのHTTPポート番号を指定します。デフォルトのポート番号は8030です。<br />**説明**<br />インポートジョブを特定のBEノードに直接送信する場合は、そのBEのHTTPポート番号を入力する必要があります。デフォルトのポート番号は8040です。 |
| database_name | はい | 対象のStarRocksテーブルが存在するデータベースの名前を指定します。 |
| table_name    | はい | 対象のStarRocksテーブルの名前を指定します。 |

> **説明**
>
> [SHOW FRONTENDS](../Administration/SHOW_FRONTENDS.md)コマンドを使用して、FEノードのIPアドレスとHTTPポート番号を確認できます。

### data_desc

ソースデータファイルを記述するために使用され、ソースデータファイルの名前、形式、列区切り文字、行区切り文字、対象パーティション、およびStarRocksテーブルとの列の対応関係などを含みます。以下のような構文です：

```Bash
-T <file_path>
-H "format: CSV | JSON"
-H "column_separator: <column_separator>"
-H "row_delimiter: <row_delimiter>"
-H "columns: <column1_name>[, <column2_name>，... ]"
-H "partitions: <partition1_name>[, <partition2_name>, ...]"
-H "temporary_partitions: <temporary_partition1_name>[, <temporary_partition2_name>, ...]"
-H "jsonpaths: [ \"<json_path1>\"[, \"<json_path2>\", ...] ]"
-H "strip_outer_array:  true | false"
-H "json_root: <json_path>"
```

`data_desc`のパラメータは、共通パラメータ、CSVに適用されるパラメータ、およびJSONに適用されるパラメータの3つのカテゴリに分けることができます。

#### 共通パラメータ

| **パラメータ名** | **必須** | **説明**                                                 |
| ------------ | ---- | ------------------------------------------------------------ |
| file_path    | はい | ソースデータファイルの保存パスを指定します。ファイル名には拡張子を含むかどうかを選択できます。 |
| format       | いいえ | インポートするデータの形式を指定します。`CSV`と`JSON`の値が取れます。デフォルト値は`CSV`です。 |
| partitions   | いいえ | どのパーティションにデータをインポートするかを指定します。このパラメータを指定しない場合は、デフォルトでStarRocksテーブルのすべてのパーティションにインポートされます。 |
| temporary_partitions | いいえ | どの[一時パーティション](../../../table_design/Temporary_partition.md)にデータをインポートするかを指定します。 |
| columns      | いいえ | ソースデータファイルとStarRocksテーブル間の列の対応関係を指定します。ソースデータファイルの列がStarRocksテーブルの列と順番に一致している場合は、`columns`パラメータを指定する必要はありません。`columns`パラメータを使用してデータ変換を実現できます。たとえば、CSV形式のデータファイルをインポートする場合、ファイルには2列があり、それぞれが目的のStarRocksテーブルの`id`と`city`の2列に対応できます。データファイルの最初の列のデータを100倍にしてからStarRocksテーブルにインポートする変換を実現するには、`"columns: city,tmp_id, id = tmp_id * 100"`と指定できます。詳細は、この記事の"[列マッピング](#列マッピング)"セクションを参照してください。 |

#### CSVに適用されるパラメータ

| **パラメータ名**     | **必須** | **説明**                                                 |
| ---------------- | ---- | ------------------------------------------------------------ |
| column_separator | いいえ | ソースデータファイル内の列区切り文字を指定します。このパラメータを指定しない場合、デフォルトは`\t`、つまりタブです。ここで指定する列区切り文字がソースデータファイル内の列区切り文字と一致していることを確認する必要があります。<br />**説明**<br />StarRocksは、最大50バイトのUTF-8エンコード文字列を列区切り文字として設定することをサポートしており、コンマ(,)、タブ、パイプ(\|)などの一般的な区切り文字が含まれます。 |
| row_delimiter    | いいえ | ソースデータファイル内の行区切り文字を指定します。このパラメータを指定しない場合、デフォルトは`\n`です。 |
| skip_header      | いいえ | CSVファイルの最初の数行をスキップすることを指定します。値のタイプはINTEGERです。デフォルト値は`0`です。<br />一部のCSVファイルでは、最初の数行に列名や列のタイプなどのメタデータ情報が定義されています。このパラメータを設定することで、StarRocksがデータをインポートする際にCSVファイルの最初の数行を無視することができます。たとえば、このパラメータを`1`に設定すると、StarRocksはデータをインポートする際にCSVファイルの最初の行を無視します。<br />ここで使用される行区切り文字は、インポートコマンドで設定した行区切り文字と一致している必要があります。 |
| trim_space       | いいえ | CSVファイル内の列区切り文字の前後のスペースを削除するかどうかを指定します。値のタイプはBOOLEANです。デフォルト値は`false`です。<br />一部のデータベースは、CSVファイルにデータをエクスポートする際に、列区切り文字の前後にいくつかのスペースを追加することがあります。位置によって、これらのスペースは「前置きスペース」または「後置きスペース」と呼ばれることがあります。このパラメータを設定することで、StarRocksがデータをインポートする際にこれらの不要なスペースを削除することができます。<br />ただし、`enclose`で指定された文字で囲まれたフィールド内のスペース（フィールドの前置きスペースと後置きスペースを含む）は削除されないことに注意してください。たとえば、列区切り文字がパイプ(<code class="language-text">&#124;</code>)で、`enclose`で指定された文字がダブルクォート(`"`)の場合：<br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> <br /><code class="language-text">&#124;" Love StarRocks "&#124;</code> <br /><code class="language-text">&#124; "Love StarRocks" &#124;</code> <br />`trim_space`を`true`に設定すると、StarRocksが処理した後の結果データは以下のようになります：<br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> <br /><code class="language-text">&#124;" Love StarRocks "&#124;</code> <br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> |
| enclose          | いいえ | [RFC4180](https://www.rfc-editor.org/rfc/rfc4180)に基づき、CSVファイル内のフィールドを囲むために使用される文字を指定します。値のタイプは単一バイト文字です。デフォルト値は`NONE`です。最も一般的に使用される`enclose`文字はシングルクォート(`'`)またはダブルクォート(`"`)です。<br />`enclose`で指定された文字で囲まれたフィールド内のすべての特殊文字（行区切り文字、列区切り文字などを含む）は通常の記号と見なされます。RFC4180標準よりもさらに進んで、StarRocksが提供する`enclose`属性は任意の単一バイトの文字を設定することをサポートしています。<br />フィールド内に`enclose`で指定された文字が含まれている場合は、同じ文字を使用して`enclose`で指定された文字をエスケープできます。たとえば、`enclose`をダブルクォート(`"`)に設定した場合、フィールド値`a "quoted" c`はCSVファイル内で`"a ""quoted"" c"`として記述されるべきです。 |

| パラメータ名 | 必須 | 説明 |
| ------------ | ---- | ---- |
| escape       | 否   | エスケープに使用する文字を指定します。行区切り文字、列区切り文字、エスケープ文字、`enclose` 指定文字など、さまざまな特殊文字をエスケープして、StarRocks がこれらの特殊文字を通常の文字として解析し、フィールド値の一部として解釈するようにします。値のタイプ：1バイト文字。デフォルト値：`NONE`。最も一般的な `escape` 文字はスラッシュ (`\`) であり、SQL ステートメントではダブルスラッシュ (`\\`) として書く必要があります。<br />**説明**<br />`escape` 指定文字は、`enclose` 指定文字の内部および外部の両方に対して同じように機能します。<br />以下に2つの例を示します：<ul><li>`enclose` をダブルクォーテーション (`"`) に設定し、`escape` をスラッシュ (`\`) に設定した場合、StarRocks は `"say \"Hello world\""` をフィールド値 `say "Hello world"` として解析します。</li><li>列区切り文字がカンマ (`,`) の場合、`escape` をスラッシュ (`\`) に設定した場合、StarRocks は `a, b\, c` を `a` と `b, c` の2つのフィールド値として解析します。</li></ul> |

> **説明**
>
> CSV 形式のデータの場合、次の2点に注意してください：
>
> - StarRocks は、最大50バイトのUTF-8エンコード文字列を列区切り文字として設定できます。これには、一般的なカンマ (,)、タブ、パイプ (|) などが含まれます。
> - 空値 (null) は `\N` で表されます。たとえば、データファイルには3つの列があり、ある行のデータの1列目と3列目のデータがそれぞれ `a` と `b` であり、2列目にデータがない場合、2列目は空値を表すために `\N` を使用する必要があります。 `a,\N,b` と書きます。`a,,b` ではありません。`a,,b` は2列目が空の文字列を表します。
> - `skip_header`、`trim_space`、`enclose`、`escape`などの属性は、3.0以降のバージョンでサポートされています。

#### JSON 用パラメータ

| **パラメータ名**     | **必須** | **説明** |
| ----------------- | ------------ | ------------------------------------------------------------ |
| jsonpaths         | 否           | インポートするフィールドの名前を指定します。JSONデータをマッチングモードでインポートする場合にのみ、このパラメータを指定する必要があります。パラメータの値はJSON形式です。[JSONデータのインポート時の列マッピングの設定](#JSONデータのインポート時の列マッピングの設定)を参照してください。     |
| strip_outer_array | 否           | 最外部の配列構造をトリムするかどうかを指定します。値の範囲：`true` または `false`。デフォルト値：`false`。実際のビジネスシナリオでは、インポートするJSONデータは最外部に配列構造を持つ場合があります。この場合、このパラメータの値を `true` に設定することをお勧めします。これにより、StarRocks は外側の角括弧 `[]` をトリムし、角括弧 `[]` 内の各内部配列を個別の行データとしてインポートします。このパラメータの値を `false` に設定すると、StarRocks はJSONデータファイル全体を配列として解析し、1行のデータとしてインポートします。たとえば、インポートするJSONデータが `[ {"category" : 1, "author" : 2}, {"category" : 3, "author" : 4} ]` の場合、このパラメータの値を `true` に設定すると、StarRocks は `{"category" : 1, "author" : 2}` と `{"category" : 3, "author" : 4}` を2行のデータとして解析し、対応するStarRocksテーブルのデータ行にインポートします。 |
| json_root         | 否           | インポートするJSONデータのルート要素を指定します。JSONデータをマッチングモードでインポートする場合にのみ、このパラメータを指定する必要があります。パラメータの値は有効なJsonPath文字列です。デフォルト値は空で、データファイル全体のデータをインポートします。詳細については、この記事の提供する例「[データのインポートとJSONルートノードの指定](#データのインポートとJSONルートノードの指定)」を参照してください。 |
| ignore_json_size | 否   | HTTPリクエストのJSONボディのサイズをチェックするかどうかを指定します。<br />**説明**<br />HTTPリクエストのJSONボディのサイズはデフォルトで100MBを超えることはできません。JSONボディのサイズが100MBを超える場合、「The size of this batch exceeds max size of json [104857600] type data data . [8617627793]チェックをスキップするようにignore_json_sizeを設定しますが、大量のメモリを消費する可能性があります。」というエラーメッセージが表示されます。このエラーを回避するためには、HTTPリクエストヘッダに `"ignore_json_size:true"` を追加して、JSONボディのサイズのチェックを無視するように設定します。 |

また、JSON形式のデータをインポートする際には、単一のJSONオブジェクトのサイズが4GBを超えてはなりません。JSONファイルの中の単一のJSONオブジェクトのサイズが4GBを超える場合、「This parser can't support a document that big.」というエラーメッセージが表示されます。

### opt_properties

インポートに関連するいくつかのオプションパラメータを指定するために使用されます。指定されたパラメータ設定は、インポートジョブ全体に適用されます。構文は次のようになります：

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

パラメータの説明は以下の表に示されています。

| **パラメータ名**     | **必須** | **説明**                                                 |
| ---------------- | ------------ | ------------------------------------------------------------ |
| ラベル            | 否           | インポートジョブのラベルを指定します。ラベルを指定しない場合、StarRocks はインポートジョブに自動的にラベルを生成します。同じラベルのデータは複数回の成功したインポートを行うことはできません。これにより、データの重複インポートを回避できます。ラベルの命名規則については、[システム制限](../../../reference/System_limit.md)を参照してください。StarRocks は、デフォルトで直近3日間の成功したインポートジョブのラベルを保持します。デフォルトの保持期間を設定するには、[FE設定パラメータ](../../../administration/FE_configuration.md#インポートとエクスポート) `label_keep_max_second` を使用できます。 |
| どこ            | 否           | フィルタ条件を指定します。このパラメータを指定すると、StarRocks は変換後のデータを指定したフィルタ条件でフィルタリングします。WHERE句で指定されたフィルタ条件に一致するデータのみがインポートされます。 |
| max_filter_ratio | 否           | インポートジョブの最大許容エラーレートを指定します。つまり、データ品質が不適格なためにフィルタリングされたデータ行の割合が最大の比率を占めることができるようにします。値の範囲：`0`~`1`。デフォルト値：`0` 。<br />データ行にエラーがある場合、インポートジョブは失敗し、データの正確性が保証されますので、デフォルト値 `0` を保持することをお勧めします。<br />エラーのあるデータ行を無視する場合は、このパラメータの値を `0` より大きく設定できます。この場合、エラーのあるデータ行があっても、インポートジョブは成功します。<br />**説明**<br />ここでフィルタリングされるデータ行は、WHERE句でフィルタリングされたデータ行は含まれません。 |
| log_rejected_record_num | 否           | データ品質が不適格なためにフィルタリングされたデータ行の最大数を指定します。このパラメータはバージョン3.1以降でサポートされています。値の範囲：`0`、`-1`、0より大きい正の整数。デフォルト値：`0`。<ul><li>`0` を指定すると、フィルタリングされたデータ行は記録されません。</li><li>`-1` を指定すると、フィルタリングされたすべてのデータ行が記録されます。</li><li>0より大きい正の整数（たとえば `n`）を指定すると、各BEノードで最大`n`件のフィルタリングされたデータ行を記録できます。</li></ul> |
| タイムアウト          | 否           | インポートジョブのタイムアウト時間を指定します。値の範囲：1〜259200。単位：秒。デフォルト値：`600`。<br />**説明**<br />`timeout` パラメータ以外にも、[FE設定パラメータ](../../../administration/FE_configuration.md#インポートとエクスポート) `stream_load_default_timeout_second` を使用して、Stream Load インポートジョブのタイムアウト時間を一元的に制御することもできます。`timeout` パラメータが指定されている場合、そのインポートジョブのタイムアウト時間は `timeout` パラメータに従います。`timeout` パラメータが指定されていない場合、そのインポートジョブのタイムアウト時間は `stream_load_default_timeout_second` に従います。 |
| strict_mode      | 否           | 厳密モードを有効にするかどうかを指定します。値の範囲：`true` または `false`。デフォルト値：`false`。`true` は有効、`false` は無効を表します。<br />このモードについての詳細は、[厳密モード](../../../loading/load_concept/strict_mode.md)を参照してください。|
| タイムゾーン         | 否           | インポートジョブで使用するタイムゾーンを指定します。デフォルトは東8区 (Asia/Shanghai) です。<br />このパラメータの値は、関連するすべてのインポートでタイムゾーンに関連する関数の結果に影響を与えます。タイムゾーンに影響を受ける関数には、strftime、alignment_timestamp、from_unixtime などがあります。詳細については、[タイムゾーンの設定](../../../administration/timezone.md)を参照してください。インポートパラメータ `timezone` で設定されたタイムゾーンは、「[タイムゾーンの設定](../../../administration/timezone.md)」で説明されているセッションレベルのタイムゾーンに対応します。 |
| load_mem_limit   | 否           | インポートジョブのメモリ制限であり、BEのメモリ制限を超えることはありません。単位：バイト。デフォルトのメモリ制限は2GBです。 |
| merge_condition  | 否           | 更新が有効になる条件として指定する列名を指定します。これにより、インポートされたデータの列の値が現在の値以上の場合にのみ更新が有効になります。StarRocks v2.5以降、条件付き更新がサポートされています。[インポートによるデータ変更の実現](../../../loading/Load_to_Primary_Key_tables.md)を参照してください。 <br/>**説明**<br/>指定された列はプライマリキーモデルテーブルでなければならず、主キーモデルテーブルのみが条件付き更新をサポートします。  |

## 列マッピング

### CSVデータのインポート時の列マッピングの設定

ソースデータファイルの列とターゲットテーブルの列が順番に対応している場合、列マッピングと変換の設定は必要ありません。

ソースデータファイルの列とターゲットテーブルの列が順番に対応していない場合、数量または順序が異なる場合、`COLUMNS` パラメータを使用して列マッピングと変換の設定をする必要があります。一般的には、次の2つのシナリオがあります：

- **列の数は同じですが、順序が異なり、データを関数で計算する必要はなく、直接ターゲットテーブルの対応する列に入れることができます。** この場合、`COLUMNS` パラメータでソースデータファイルの列の順序に従って、ターゲットテーブルの対応する列名を使用して列マッピングと変換の設定を行う必要があります。


```
  例えば、ターゲットテーブルには `col1`、`col2`、`col3` の3列があり、ソースデータファイルにも `col3`、`col2`、`col1` の順に3列があります。この場合、`COLUMNS(col3, col2, col1)` を指定する必要があります。

- **列の数や順序が一致しない場合、または一部の列のデータを関数で計算した後にターゲットテーブルの対応する列に挿入する必要がある場合。** このシナリオでは、`COLUMNS` パラメータでソースデータファイルの列の順序に従ってターゲットテーブルの対応する列名を使用して列マッピングを設定するだけでなく、データ計算に使用する関数も指定する必要があります。以下は2つの例です:

  - ターゲットテーブルには `col1`、`col2`、`col3` の3列があり、ソースデータファイルには4列あり、最初の3列は `col1`、`col2`、`col3` に対応していますが、4列目にはターゲットテーブルに対応する列がありません。この場合、`COLUMNS(col1, col2, col3, temp)` を指定し、最後の列は占位として任意の名前（例：`temp`）を指定できます。
  - ターゲットテーブルには `year`、`month`、`day` の3列があり、ソースデータファイルには `yyyy-mm-dd hh:mm:ss` 形式の日時データが1列だけ含まれています。この場合、`COLUMNS(col, year = year(col), month = month(col), day = day(col))` を指定できます。ここで、`col` はソースデータファイルの列の一時的な名前で、`year = year(col)`、`month = month(col)`、`day = day(col)` は `col` 列から対応するデータを抽出してターゲットテーブルの対応する列に挿入するために使用されます。例えば、`year = year(col)` は `col` 列の `yyyy` 部分を `year` 関数で抽出し、ターゲットテーブルの `year` 列に挿入します。

列マッピングの設定に関する操作例については、[列マッピングの設定](#设置列映射关系)を参照してください。

### JSON データをインポートする際の列マッピングの設定

JSON ファイルの Key 名がターゲットテーブルの列名と一致する場合、シンプルモードを使用してデータをインポートできます。シンプルモードでは、`jsonpaths` パラメータを設定する必要はありません。このモードでは、JSON データが `{}` で表されるオブジェクトタイプであることが要求されます。例えば、`{"category": 1, "author": 2, "price": "3"}` では、`category`、`author`、`price` が Key の名前で、それぞれターゲットテーブルの `category`、`author`、`price` 列に直接対応しています。

JSON ファイルの Key 名がターゲットテーブルの列名と一致しない場合は、マッチングモードを使用してデータをインポートする必要があります。マッチングモードでは、`jsonpaths` と `COLUMNS` の2つのパラメータを使用して、JSON ファイルの Key とターゲットテーブルの列の間のマッピングと変換関係を指定します:

- `jsonpaths` パラメータでは、JSON ファイルの Key の順序に従ってインポートする Key を指定します。
- `COLUMNS` パラメータでは、JSON ファイルの Key とターゲットテーブルの列の間のマッピング関係とデータ変換関係を指定します。
  - `COLUMNS` パラメータで指定された列名は、`jsonpaths` パラメータで指定された Key と順序を一致させます。
  - `COLUMNS` パラメータで指定された列名は、ターゲットテーブルの列名と名前を一致させます。

マッチングモードを使用して JSON データをインポートする例については、[マッチングモードを使用したデータのインポート](#使用匹配模式导入数据)を参照してください。

## 戻り値

インポートが終了すると、インポートジョブの結果情報が JSON 形式で返されます。以下のように表示されます:

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

戻り値のパラメータの説明は以下の表の通りです。

| **パラメータ名**           | **説明**                                                     |
| ---------------------- | ------------------------------------------------------------ |
| TxnId                  | インポートジョブのトランザクションID。                                          |
| Label                  | インポートジョブのラベル。                                             |
| Status                 | このインポートのデータの最終状態。<ul><li>`Success`：データのインポートに成功し、データが表示されています。</li><li>`Publish Timeout`：インポートジョブが成功して提出されましたが、何らかの理由でデータがすぐには表示されません。成功したと見なし、インポートを再試行する必要はありません。</li><li>`Label Already Exists`：そのラベルはすでに他のインポートジョブによって使用されています。データはインポートされている可能性がありますが、インポート中の可能性もあります。</li><li>`Fail`：データのインポートに失敗しました。ラベルを指定してインポートジョブを再試行できます。</li></ul> |
| Message                | インポートジョブの詳細な状態。インポートジョブが失敗した場合、ここに具体的な失敗理由が返されます。 |
| NumberTotalRows        | 読み取られた総行数。                                             |
| NumberLoadedRows       | 成功してインポートされた総行数。`Status` が `Success` の場合にのみ有効です。 |
| NumberFilteredRows     | データ品質が不適合でフィルタリングされた行数。                   |
| NumberUnselectedRows   | WHERE 句で指定された条件に基づいてフィルタリングされた行数。          |
| LoadBytes              | このインポートのデータ量のサイズ。単位：バイト (Bytes)。                   |
| LoadTimeMs             | このインポートにかかった時間。単位：ミリ秒 (ms)。                        |
| BeginTxnTimeMs         | インポートジョブがトランザクションを開始した時間。                                     |
| StreamLoadPlanTimeMs   | インポートジョブが実行計画を生成した時間。                                 |
| ReadDataTimeMs         | インポートジョブがデータを読み取った時間。                                     |
| WriteDataTimeMs        | インポートジョブがデータを書き込んだ時間。                                     |
| CommitAndPublishTimeMs | インポートジョブがコミットしてデータを公開した時間。                               |

インポートジョブが失敗した場合、`ErrorURL` も返されます。以下のように表示されます:

```JSON
{
    "ErrorURL": "http://172.26.195.68:8045/api/_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be"
}
```

`ErrorURL` を通じて、データ品質が不適合でフィルタリングされたエラーデータ行の具体的な情報を確認できます。インポートジョブを提出する際に、オプションのパラメータ `log_rejected_record_num` を使用して、記録できるエラーデータ行の最大数を指定できます。

`curl "url"` コマンドを使用して直接エラーデータ行の情報を確認することも、`wget "url"` コマンドを使用してエラーデータ行の情報をエクスポートすることもできます。以下のように表示されます:

```Bash
wget http://172.26.195.68:8045/api/_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be
```

エクスポートされたエラーデータ行の情報は、`_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be` という名前のローカルファイルに保存されます。`cat _load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be` コマンドを使用してファイルの内容を確認できます。

エラー情報に基づいてインポートジョブを調整し、再提出することができます。

## 例

### CSV データのインポート

このセクションでは、CSV データを例にして、インポートジョブを作成する際に、さまざまなパラメータ設定を使用して異なるビジネスシナリオの要件を満たす方法について詳しく説明します。

#### **タイムアウト時間の設定**

StarRocks データベース `test_db` のテーブル `table1` には `col1`、`col2`、`col3` の3列が含まれています。

データファイル `example1.csv` にも3列が含まれており、`table1` の `col1`、`col2`、`col3` にそれぞれ対応しています。

`example1.csv` のすべてのデータを `table1` にインポートし、タイムアウト時間を最大100秒に設定する場合、以下のコマンドを実行できます:

```Bash
curl --location-trusted -u <username>:<password> -H "label:label1" \
    -H "Expect:100-continue" \
    -H "timeout:100" \
    -H "max_filter_ratio:0.2" \
    -T example1.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
```

#### **最大許容エラー率の設定**

StarRocks データベース `test_db` のテーブル `table2` には `col1`、`col2`、`col3` の3列が含まれています。

データファイル `example2.csv` にも3列が含まれており、`table2` の `col1`、`col2`、`col3` にそれぞれ対応しています。

`example2.csv` のすべてのデータを `table2` にインポートし、最大許容エラー率を0.2に設定する場合、以下のコマンドを実行できます:

```Bash
curl --location-trusted -u <username>:<password> -H "label:label2" \
    -H "Expect:100-continue" \
    -H "max_filter_ratio:0.2" \
    -T example2.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table2/_stream_load
```

#### **列マッピングの設定**

StarRocks データベース `test_db` のテーブル `table3` には `col1`、`col2`、`col3` の3列が含まれています。

データファイル `example3.csv` にも3列が含まれており、`table3` の `col2`、`col1`、`col3` にそれぞれ対応しています。

`example3.csv` のすべてのデータを `table3` にインポートする場合、以下のコマンドを実行できます:

```Bash
curl --location-trusted -u <username>:<password>  -H "label:label3" \
    -H "Expect:100-continue" \
    -H "columns: col2, col1, col3" \
    -T example3.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table3/_stream_load
```

> **説明**
>
> 上記の例では、`example3.csv` と `table3` が含む列が順番に対応していないため、`columns` パラメータを使用して `example3.csv` と `table3` の間の列マッピングを設定する必要があります。

#### **フィルタ条件の設定**


StarRocksのデータベース`test_db`にあるテーブル`table4`は、`col1`、`col2`、`col3`の3つの列を順に含んでいます。

データファイル`example4.csv`にも3つの列があり、`table4`の`col1`、`col2`、`col3`にそれぞれ対応しています。

`example4.csv`の第一列の値が`20180601`であるデータ行のみを`table4`にインポートしたい場合は、以下のコマンドを実行します：

```Bash
curl --location-trusted -u <username>:<password> -H "label:label4" \
    -H "Expect:100-continue" \
    -H "columns: col1, col2, col3]" \
    -H "where: col1 = 20180601" \
    -T example4.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table4/_stream_load
```

> **説明**
>
> 上記の例では、`example4.csv`と`table4`が同じ数の列を含み、順番に対応していますが、列に基づくフィルタ条件をWHERE句で指定する必要があるため、`columns`パラメータを使用して`example4.csv`の列に一時的な名前を付けます。

#### **ターゲットパーティションの設定**

StarRocksのデータベース`test_db`にあるテーブル`table5`は、`col1`、`col2`、`col3`の3つの列を順に含んでいます。

データファイル`example5.csv`にも3つの列があり、`table5`の`col1`、`col2`、`col3`にそれぞれ対応しています。

`example5.csv`の全てのデータを`table5`のパーティション`p1`と`p2`にインポートしたい場合は、以下のコマンドを実行します：

```Bash
curl --location-trusted -u <username>:<password> -H "label:label5" \
    -H "Expect:100-continue" \
    -H "partitions: p1, p2" \
    -T example5.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table5/_stream_load
```

#### **厳格モードとタイムゾーンの設定**

StarRocksのデータベース`test_db`にあるテーブル`table6`は、`col1`、`col2`、`col3`の3つの列を順に含んでいます。

データファイル`example6.csv`にも3つの列があり、`table6`の`col1`、`col2`、`col3`にそれぞれ対応しています。

`example6.csv`の全てのデータを`table6`にインポートし、厳格モードでフィルタリングを行い、タイムゾーン`Africa/Abidjan`を使用する場合は、以下のコマンドを実行します：

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "strict_mode: true" \
    -H "timezone: Africa/Abidjan" \
    -T example6.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table6/_stream_load
```

#### **HLL型の列を含むテーブルへのデータインポート**

StarRocksのデータベース`test_db`にあるテーブル`table7`は、`col1`、`col2`の2つのHLL型の列を順に含んでいます。

データファイル`example7.csv`には2つの列があり、第一列は`table7`のHLL型の列`col1`に対応し、関数を通じてHLL型のデータに変換して`col1`列に格納できます。第二列は`table7`のどの列にも対応していません。

`example7.csv`の対応するデータを`table7`にインポートしたい場合は、以下のコマンドを実行します：

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "columns: temp1, temp2, col1=hll_hash(temp1), col2=hll_empty()" \
    -T example7.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table7/_stream_load
```

> **説明**
>
> 上記の例では、`columns`パラメータを使用して`example7.csv`の2つの列を`temp1`、`temp2`として一時的に命名し、関数を使用してデータ変換ルールを指定しています。これには以下が含まれます：
>
> - `hll_hash`関数を使用して`example7.csv`の`temp1`列をHLL型のデータに変換し、`table7`の`col1`列にマッピングします。
>
> - `hll_empty`関数を使用して、`table7`の第二列にデフォルト値を補充します。

`hll_hash`関数と`hll_empty`関数の使用方法については、[hll_hash](../../sql-functions/aggregate-functions/hll_hash.md)と[hll_empty](../../sql-functions/aggregate-functions/hll_empty.md)を参照してください。

#### **BITMAP型の列を含むテーブルへのデータインポート**

StarRocksのデータベース`test_db`にあるテーブル`table8`は、`col1`、`col2`の2つのBITMAP型の列を順に含んでいます。

データファイル`example8.csv`には2つの列があり、第一列は`table8`のBITMAP型の列`col1`に対応し、関数を通じてBITMAP型のデータに変換して`col1`列に格納できます。第二列は`table8`のどの列にも対応していません。

`example8.csv`の対応するデータを`table8`にインポートしたい場合は、以下のコマンドを実行します：

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "columns: temp1, temp2, col1=to_bitmap(temp1), col2=bitmap_empty()" \
    -T example8.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table8/_stream_load
```

> **説明**
>
> 上記の例では、`columns`パラメータを使用して`example8.csv`の2つの列を`temp1`、`temp2`として一時的に命名し、関数を使用してデータ変換ルールを指定しています。これには以下が含まれます：
>
> - `to_bitmap`関数を使用して`example8.csv`の`temp1`列をBITMAP型のデータに変換し、`table8`の`col1`列にマッピングします。
>
> - `bitmap_empty`関数を使用して、`table8`の第二列にデフォルト値を補充します。

`to_bitmap`関数と`bitmap_empty`関数の使用方法については、[to_bitmap](../../sql-functions/bitmap-functions/to_bitmap.md)と[bitmap_empty](../../sql-functions/bitmap-functions/bitmap_empty.md)を参照してください。

#### `skip_header`、`trim_space`、`enclose`、`escape`の設定

StarRocksのデータベース`test_db`にあるテーブル`table9`は、`col1`、`col2`、`col3`の3つの列を順に含んでいます。

データファイル`example9.csv`にも3つの列があり、`table9`の`col2`、`col1`、`col3`にそれぞれ対応しています。

`example9.csv`の全てのデータを`table9`にインポートし、ファイルの最初の5行をスキップし、列の区切り文字の前後の空白を削除し、エンクローズ文字をバックスラッシュ(`\`)に設定し、エスケープ文字をバックスラッシュ(`\`)に設定したい場合は、以下のコマンドを実行します：

```Bash
curl --location-trusted -u <username>:<password> -H "label:3875" \
    -H "Expect:100-continue" \
    -H "trim_space: true" -H "skip_header: 5" \
    -H "column_separator:," -H "enclose:\"" -H "escape:\\" \
    -H "columns: col2, col1, col3" \
    -T example9.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/tbl9/_stream_load
```

### **JSON形式のデータのインポート**

このセクションでは、JSON形式のデータをインポートする際に注意すべきいくつかのパラメータ設定について説明します。

StarRocksのデータベース`test_db`にあるテーブル`tbl1`は、以下のようなテーブル構造を持っています：

```SQL
`category` varchar(512) NULL COMMENT "",
`author` varchar(512) NULL COMMENT "",
`title` varchar(512) NULL COMMENT "",
`price` double NULL COMMENT ""
```

#### **シンプルモードでのデータインポート**

データファイル`example1.json`には以下のデータが含まれているとします：

```JSON
{"category":"C++","author":"avc","title":"C++ primer","price":895}
```

以下のコマンドを実行して`example1.json`のデータを`tbl1`にインポートできます：

```Bash
curl --location-trusted -u <username>:<password> -H "label:label6" \
    -H "Expect:100-continue" \
    -H "format: json" \
    -T example1.json -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/tbl1/_stream_load
```

> **説明**
>
> 上記の例のように、`columns`や`jsonpaths`パラメータを指定しない場合、StarRocksのテーブルの列名に基づいてJSONデータファイルのフィールドを対応させます。

スループットを向上させるために、Stream Loadは一度に複数のデータをインポートすることができます。例えば、以下のような複数のJSONデータを一度にインポートできます：

```JSON
[
    {"category":"C++","author":"avc","title":"C++ primer","price":89.5},
    {"category":"Java","author":"avc","title":"Effective Java","price":95},
    {"category":"Linux","author":"avc","title":"Linux kernel","price":195}
]
```

#### **マッチングモードでのデータインポート**

StarRocksは以下の順序でデータをマッチングおよび処理します：

1. （オプション）`strip_outer_array`パラメータに基づいて最外層の配列構造を削除します。

    > **説明**
    >
    > JSONデータの最外層が中括弧`[]`で表される配列構造の場合のみ関連し、`strip_outer_array`を`true`に設定する必要があります。

2. （オプション）`json_root`パラメータに基づいてJSONデータのルートノードをマッチングします。

    > **説明**
    >
    > JSONデータにルートノードが存在する場合のみ関連し、`json_root`パラメータを使用してルートノードを指定する必要があります。

3. `jsonpaths`パラメータに基づいてインポートするJSONデータを抽出します。

##### **JSONルートノードを指定せずにマッチングモードでデータをインポートする**

データファイル`example2.json`には以下のデータが含まれているとします：

```JSON
[
    {"category":"xuxb111","author":"1avc","title":"SayingsoftheCentury","price":895},
    {"category":"xuxb222","author":"2avc","title":"SayingsoftheCentury","price":895},
    {"category":"xuxb333","author":"3avc","title":"SayingsoftheCentury","price":895}
]
```

`jsonpaths`パラメータを指定して正確にインポートすることができます。例えば、`category`、`author`、`price`の3つのフィールドのみのデータをインポートする場合は以下のようにします：

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

> **説明**
>

> 上述例では、JSON データの最外層が中括号 [] で表される配列構造であり、配列内の各 JSON オブジェクトが一つのデータレコードを表しています。したがって、最外層の配列構造を削除するために `strip_outer_array` を `true` に設定する必要があります。インポート中に、指定されていないフィールド `title` は無視されます。

##### **JSON ルートノードを指定して、マッチングパターンでデータをインポート**

例えば、データファイル `example3.json` に以下のデータが含まれているとします:

```JSON
{
    "id": 10001,
    "RECORDS":[
        {"category":"11","title":"SayingsoftheCentury","price":895,"timestamp":1589191587},
        {"category":"22","author":"2avc","price":895,"timestamp":1589191487},
        {"category":"33","author":"3avc","title":"SayingsoftheCentury","timestamp":1589191387}
    ],
    "comments": ["3 records", "there will be 3 rows"]
}
```

`jsonpaths` を指定して正確にデータをインポートすることができます。例えば、`category`、`author`、`price` の3つのフィールドのみをインポートする場合は以下のようにします:

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

> **説明**
>
> 上述例では、JSON データの最外層が中括号 [] で表される配列構造であり、配列内の各 JSON オブジェクトが一つのデータレコードを表しています。したがって、最外層の配列構造を削除するために `strip_outer_array` を `true` に設定する必要があります。インポート中に、指定されていないフィールド `title` と `timestamp` は無視されます。さらに、`json_root` パラメータを使用して、実際にインポートする必要があるデータが `RECORDS` フィールドの値、つまり JSON 配列であることを指定しています。

