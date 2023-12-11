```yaml
displayed_sidebar: "Japanese"
```

# STREAM LOAD

## 説明

StarRocksは、ローカルファイルシステムやストリーミングデータソースからデータをロードするためのHTTPベースのストリームロードというローディングメソッドを提供します。ロードジョブを提出すると、StarRocksは同期的にジョブを実行し、ジョブが終了した後にその結果を返します。ジョブの結果に基づいて、ジョブが成功したかどうかを判断できます。Stream Loadのアプリケーションシナリオ、制限、原則、およびサポートされるデータファイルフォーマットに関する情報については、[ローカルファイルシステムからのストリームロード](../../../loading/StreamLoad.md#loading-from-a-local-file-system-via-stream-load)を参照してください。

> **注意**
>
> - Stream Loadを使用してStarRocksテーブルにデータをロードした後、そのテーブルに作成されたマテリアライズドビューのデータも更新されます。
> - Stream Loadを使用してStarRocksテーブルにデータをロードするには、そのStarRocksテーブルにINSERT権限を持つユーザーとしてのみ可能です。INSERT権限がない場合は、[GRANT](../account-management/GRANT.md)で接続するユーザーにINSERT権限を付与する手順に従ってください。

## 構文

```Bash
curl --location-trusted -u <username>:<password> -XPUT <url>
(
    data_desc
)
[opt_properties]        
```

このトピックでは、curlを使用してStream Loadを実行する方法を説明する例を示します。curl以外にも、他のHTTP互換のツールや言語を使用してStream Loadを実行することもできます。ロード関連のパラメータは、HTTPリクエストヘッダフィールドに含まれます。これらのパラメータを入力する際には、以下の点に注意してください。

- このトピックで示されるように、チャンクトランスファーエンコーディングを使用できます。チャンクトランスファーエンコーディングを選択しない場合は、転送するコンテンツの長さを示す`Content-Length`ヘッダフィールドを入力する必要があります。これによりデータの整合性が保たれます。

  > **注意**
  >
  > curlを使用してStream Loadを実行する場合、StarRocksは自動的に`Content-Length`ヘッダフィールドを追加し、手動で入力する必要はありません。

- `Expect`ヘッダフィールドを追加し、その値を`100-continue`と指定する必要があります。これにより、ジョブリクエストが拒否された場合の不必要なデータ転送を防ぎ、リソースのオーバーヘッドを軽減できます。

StarRocksでは、SQL言語によって予約されたキーワードとして一部のリテラルが使用されています。これらのキーワードを直接SQLステートメントで使用しないでください。SQLステートメントでそのようなキーワードを使用したい場合は、バッククォート(`)で囲んでください。[Keywords](../../../sql-reference/sql-statements/keywords.md)を参照してください。

## パラメータ

### ユーザー名およびパスワード

StarRocksクラスタに接続するために使用するアカウントのユーザー名とパスワードを指定します。このパラメータは必須です。パスワードが設定されていないアカウントを使用する場合は、`<username>:`のみ入力する必要があります。

### XPUT

HTTPリクエストメソッドを指定します。このパラメータは必須です。Stream LoadではPUTメソッドのみをサポートしています。

### URL

StarRocksテーブルのURLを指定します。構文は以下の通りです。

```Plain
http://<fe_host>:<fe_http_port>/api/<database_name>/<table_name>/_stream_load
```

以下の表に、URL内のパラメータについて説明します。

| パラメータ      | 必須      | 説明                                              |
| ------------- | -------- | ------------------------------------------------- |
| fe_host       | はい     | StarRocksクラスタのFEノードのIPアドレス。<br/>**注意**<br/>ロードジョブを特定のBEノードに提出する場合、BEノードのIPアドレスを入力する必要があります。 |
| fe_http_port  | はい     | StarRocksクラスタのFEノードのHTTPポート番号。デフォルトのポート番号は`8030`です。<br/>**注意**<br/>ロードジョブを特定のBEノードに提出する場合、BEノードのHTTPポート番号を入力する必要があります。デフォルトのポート番号は`8030`です。 |
| database_name | はい     | StarRocksテーブルが属するデータベースの名前。    |
| table_name    | はい     | StarRocksテーブルの名前。                         |

> **注意**
>
> [SHOW FRONTENDS](../Administration/SHOW_FRONTENDS.md)を使用して、FEノードのIPアドレスおよびHTTPポートを表示できます。

### data_desc

ロードするデータファイルを記述します。`data_desc`記述子には、データファイルの名前、フォーマット、列セパレータ、行セパレータ、宛先パーティション、およびStarRocksテーブルに対する列マッピングを含めることができます。構文は以下の通りです。

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

`data_desc`記述子のパラメータは、一般的なパラメータ、CSVパラメータ、JSONパラメータに分類できます。

#### 一般的なパラメータ

| パラメータ    | 必須      | 説明                                              |
| ---------- | -------- | ------------------------------------------------- |
| file_path  | はい     | データファイルの保存パス。ファイル名の拡張子をオプションで含めることができます。 |
| format     | いいえ   | データファイルのフォーマット。有効な値は`CSV`および`JSON`です。デフォルト値は`CSV`です。 |
| partitions | いいえ   | データファイルをロードするパーティション。このパラメータを指定しない場合、StarRocksはデータファイルをStarRocksテーブルのすべてのパーティションにロードします。 |
| temporary_partitions| いいえ   | データファイルをロードしたい[一時パーティション](../../../table_design/Temporary_partition.md)の名前。複数の一時パーティションを指定することができます。それらはカンマ(,)で区切る必要があります。 |
| columns    | いいえ   | データファイルとStarRocksテーブルの間の列マッピング。<br/>データファイルのフィールドがStarRocksテーブルの列に順次マッピングされる場合、このパラメータを指定する必要はありません。代わりに、データ変換を実装するためにこのパラメータを使用できます。たとえば、CSVデータファイルをロードし、ファイルに`id`および`city`の2つの列が順次マッピングできる場合、`"columns: city,tmp_id, id = tmp_id * 100"`と指定できます。詳細については、このトピックの"[Column mapping](#column-mapping)"セクションを参照してください。|

#### CSVパラメータ

| パラメータ          | 必須      | 説明                                              |
| ------------------ | -------- | ------------------------------------------------- |
| column_separator   | いいえ   | データファイルでフィールドを区切るために使用される文字。このパラメータを指定しない場合、デフォルトで`\t`となります。指定する列セパレータは、ロードコマンドで指定した列セパレータと同じであることを確認してください。<br/>**注意**<br/>CSVデータの場合、テキストデリミタとして、文字数が50バイトを超えない、カンマ(,)、タブ、またはパイプ(|)などのUTF-8文字列を使用できます。 |
| row_delimiter      | いいえ   | データファイルで行を区切るために使用される文字。このパラメータを指定しない場合、デフォルトで`\n`となります。 |
| skip_header        | いいえ   | データファイルがCSV形式の場合、データファイルの最初の数行をスキップするかどうかを指定します。タイプ：整数。デフォルト値は`0`です。<br />一部のCSV形式のデータファイルでは、最初の数行が列名や列のデータ型などのメタデータを定義するために使用されます。`skip_header`パラメータを設定することで、StarRocksにデータファイルの最初の数行をロード時にスキップさせることができます。たとえば、このパラメータを`1`に設定すると、StarRocksはデータファイルのロード時に最初の行をスキップします。<br />データファイルの最初の数行は、ロードコマンドで指定した行セパレータで区切る必要があります。 |
| trim_space       | いいえ    | データファイルがCSV形式の場合、列の区切り記号の前後のスペースを削除するかどうかを指定します。タイプ：真偽値。デフォルト値： `false`。<br />一部のデータベースでは、CSV形式のデータファイルとしてデータをエクスポートする際に、列の区切り記号にスペースが追加されます。このようなスペースは、その位置に応じて先行スペースまたは後続スペースと呼ばれます。`trim_space` パラメータを設定することで、StarRocks によってデータの読み込み中にこのような不要なスペースを削除することができます。<br />なお、`enclose` で指定された文字で囲まれたフィールド内のスペース（先行スペースおよび後続スペースを含む）は、StarRocks で削除されません。たとえば、次のフィールド値ではパイプ（ <code class="language-text">&#124;</code> ）を列の区切り記号、ダブルクォーテーションマーク (`"`) を `enclose` で指定された文字として使用します：<br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> <br /><code class="language-text">&#124;" Love StarRocks "&#124;</code> <br /><code class="language-text">&#124; "Love StarRocks" &#124;</code> <br />`trim_space` を `true` に設定すると、StarRocks は前述のフィールド値を次のように処理します：<br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> <br /><code class="language-text">&#124;" Love StarRocks "&#124;</code> <br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> |
| enclose          | いいえ    | データファイルがCSV形式の場合、[RFC4180](https://www.rfc-editor.org/rfc/rfc4180) に従ってフィールド値を囲む文字を指定します。タイプ：シングルバイト文字。デフォルト値： `NONE`。最も一般的な文字はシングルクォーテーションマーク（`'`）およびダブルクォーテーションマーク（`"`）です。<br />`enclose` で指定された文字で囲まれた特殊文字（行区切りおよび列区切りを含む）は通常の記号と見なされます。RFC4180 よりも多くのことができるため、任意のシングルバイト文字を `enclose` で指定された文字として指定できます。<br />フィールド値に `enclose` で指定された文字が含まれる場合、同じ文字を使用してその `enclose` で指定された文字をエスケープすることができます。たとえば、`enclose` を `"` に設定し、フィールド値を `a "quoted" c` にする場合、フィールド値は `"a ""quoted"" c"` としてデータファイルに入力できます。 |
| escape           | いいえ    | 行区切り記号、列区切り記号、エスケープ文字、`enclose` で指定された文字などのさまざまな特殊文字をエスケープするために使用される文字を指定します。この文字は、StarRocks で一般の文字と見なされ、それが属するフィールド値の一部として解析されます。タイプ：シングルバイト文字。デフォルト値： `NONE`。最も一般的な文字はスラッシュ（ `\` ）で、SQL ステートメントではダブルスラッシュ（ `\\` ）として記述する必要があります。<br />**注記**<br />`escape` で指定された文字は、`enclose` で指定された文字の内部および外部の両方に適用されます。<br />以下に 2 つの例を示します：<ul><li>`enclose` を `"` に、`escape` を `\` に設定した場合、StarRocks は `"say \"Hello world\""` を `say "Hello world"` に解析します。</li><li>列区切り記号がコンマ（`,`）の場合、`escape` を `\` に設定した場合、StarRocks は `a, b\, c` を 2 つの別々のフィールド値である `a` と `b, c` に解析します。</li></ul> |

> **注記**
>
> - CSV データの場合、テキスト区切り記号として、カンマ（`,`）、タブ、またはパイプ（`|`）などの、50 バイトを超えない UTF-8 文字列を使用できます。
> - `\N` を使用してヌル値を示します。たとえば、データファイルが 3 つの列から構成され、そのデータファイルのレコードが最初と三番目の列にデータを保持しており、二番目の列にはデータを保持していない場合、これは、レコードを `a,\N,b` としてコンパイルする必要があります。 ここで `a,,b` は、レコードにおける二番目の列に空の文字列が保持されていることを意味します。
> - `skip_header`、`trim_space`、`enclose`、`escape` を含むフォーマットオプションは、v3.0 以降でサポートされています。

#### JSON パラメータ

| パラメータ         | 必須 | 説明                                                         |
| ----------------- | -------- | ------------------------------------------------------------ |
| jsonpaths         | いいえ    | JSON データファイルからロードするキーの名前。一致モードで JSON データをロードする場合にのみこのパラメータを指定する必要があります。このパラメータの値は JSON 形式です。[JSON データのロードにカラムマッピングを構成する](#configure-column-mapping-for-json-data-loading)を参照してください。           |
| strip_outer_array | いいえ    | 最外の配列構造を取り除くかどうかを指定します。有効な値： `true` および `false`。デフォルト値： `false`。<br/>実際のビジネスシナリオでは、JSON データには角かっこ `[]` で示される最外の配列構造が含まれる場合があります。この場合、このパラメータを `true` に設定することをお勧めします。これにより、StarRocks は最外の角かっこ `[]` を削除し、それぞれの内部配列を別々のデータレコードとしてロードします。このパラメータを `false` に設定すると、StarRocks は JSON データファイル全体を 1 つの配列として解析し、その配列を 1 つのデータレコードとしてロードします。<br/>たとえば、JSON データが `[ {"category" : 1, "author" : 2}, {"category" : 3, "author" : 4} ]` の場合、このパラメータを `true` に設定すると、`{"category" : 1, "author" : 2}` と `{"category" : 3, "author" : 4}` は別々のデータレコードとして解析され、別々の StarRocks テーブル行にロードされます。 |
| json_root         | いいえ    | JSON データファイルからロードする JSON データのルート要素。一致モードで JSON データをロードする場合にのみこのパラメータを指定する必要があります。このパラメータの値は有効な JsonPath 文字列です。デフォルトでは、このパラメータの値が空であり、JSON データファイルのすべてのデータがロードされます。詳細についてはこのトピックの"[指定されたルート要素を使用して一致モードで JSON データをロードする](#load-json-data-using-matched-mode-with-root-element-specified)"セクションを参照してください。 |
| ignore_json_size  | いいえ    | HTTP リクエスト内の JSON 本体のサイズをチェックするかどうかを指定します。<br/>**注記**<br/>デフォルトでは、HTTP リクエスト内の JSON 本体のサイズは 100 MB を超えることはできません。JSON 本体が 100 MB を超えると、"The size of this batch exceed the max size [104857600] of json type data data [8617627793]. Set ignore_json_size to skip check, although it may lead huge memory consuming." というエラーが報告されます。このエラーを防ぐために、HTTP リクエストヘッダに `"ignore_json_size:true"` を追加して、StarRocks に JSON 本体のサイズを確認しないように指示できます。 |

JSON データをロードする際に、1 つの JSON オブジェクトあたりのサイズが 4 GB を超えてはいけないことにも注意してください。JSON データファイル内の個々の JSON オブジェクトが 4 GB を超える場合、"This parser can't support a document that big." というエラーが報告されます。

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

次の表に、オプションパラメータに関する説明が示されています。

| パラメータ        | 必須 | 説明                                                         |
| ---------------- | -------- | ------------------------------------------------------------ |
| label            | いいえ    | ロードジョブのラベル。このパラメータを指定しない場合、StarRocks は自動的にロードジョブのためのラベルを生成します。<br/>StarRocks は 1 つのラベルを使用してデータバッチを複数回ロードすることを許可しません。そのため、StarRocks は同じデータが繰り返しロードされることを防ぎます。ラベルの命名規則については、[System limits](../../../reference/System_limit.md) を参照してください。<br/>デフォルトでは、StarRocks は最近 3 日間で正常に完了したロードジョブのラベルを保持します。ラベルの保持期間を変更するには、[FE parameter](../../../administration/Configuration.md) `label_keep_max_second` を使用できます。 |
| where            | いいえ    | StarRocks が事前処理されたデータをフィルタリングする基準。StarRocks は WHERE 句で指定されたフィルタ条件に一致する事前処理済みデータのみをロードします。 |
| max_filter_ratio | いいえ | 読み込みジョブの最大許容エラー許容度です。エラー許容度とは、読み込みジョブによって要求されたすべてのデータレコードの中でデータ品質が不十分であるためにフィルタリングされたデータレコードの最大割合です。有効な値：`0` から `1`。デフォルト値： `0`。<br/>デフォルト値 `0` を維持することをお勧めします。これにより、不適格なデータ レコードが検出された場合、読み込みジョブが失敗し、データの正確性が確保されます。<br/>不適格なデータ レコードを無視する場合は、このパラメータを`0`より大きな値に設定できます。このようにすると、データファイルに不適格なデータ レコードが含まれていても読み込みジョブが成功することがあります。<br/>**注意**<br/>不適格なデータ レコードには、WHERE句でフィルタリングされたデータ レコードは含まれません。 |
| log_rejected_record_num | いいえ | ロギングできる不適格データ行の最大数を指定します。このパラメータは v3.1 以降でサポートされています。有効な値：`0`、`-1`、およびゼロ以外の正の整数。デフォルト値： `0`。<ul><li>値 `0` は、フィルタリングされたデータ行はログに記録されません。</li><li>値 `-1` は、フィルタリングされたすべてのデータ行がログに記録されます。</li><li>`n` のようなゼロ以外の正の整数は、各 BE にログに記録できる最大の `n` のデータ行を指定します。</li></ul> |
| timeout          | いいえ | 読み込みジョブのタイムアウト時間です。有効な値：`1` から `259200`。単位：秒。デフォルト値： `600`。<br/>**注意**`timeout` パラメータに加えて、StarRocks クラスタ内のすべての Stream Load ジョブのタイムアウト時間を一元的に制御するために、[FEパラメータ](../../../administration/Configuration.md) `stream_load_default_timeout_second` を使用することもできます。`timeout` パラメータを指定すると、`timeout` パラメータで指定されたタイムアウト時間が優先されます。`timeout` パラメータを指定しない場合、`stream_load_default_timeout_second` パラメータで指定されたタイムアウト時間が優先されます。 |
| strict_mode      | いいえ | [strict モード](../../../loading/load_concept/strict_mode.md)を有効にするかどうかを指定します。有効な値：`true` および `false`。デフォルト値： `false`。`true` は strict モードを有効にし、`false` は strict モードを無効にします。 |
| timezone         | いいえ | 読み込みジョブで使用されるタイム ゾーンです。デフォルト値： `Asia/Shanghai`。このパラメータの値は、strftime、alignment_timestamp、および from_unixtime などの関数によって返される結果に影響を与えます。このパラメータで指定されたタイム ゾーンはセッション レベルのタイム ゾーンです。詳細については、[タイム ゾーンの構成](../../../administration/timezone.md)を参照してください。 |
| load_mem_limit   | いいえ | 読み込みジョブに割り当てることができる最大メモリ容量です。単位：バイト。読み込みジョブのデフォルトの最大メモリ サイズは 2 GB です。このパラメータの値は、各 BE に割り当てることができる最大メモリ 容量を超えることはできません。 |
| merge_condition  | いいえ | 更新が有効になる条件として使用する列の名前を指定します。ソース レコードから宛先レコードに対する更新は、指定された列の値がソース データ レコードで宛先データ レコードの値以上である場合にのみ有効になります。StarRocks は v2.5 以降で条件付き更新をサポートしています。詳細については、[読み込みを通じてデータを変更する](../../../loading/Load_to_Primary_Key_tables.md)を参照してください。<br/>**注意**<br/>指定する列はプライマリ キー列ではない必要があります。また、条件付き更新はプライマリ キー テーブルを使用するテーブルのみサポートされます。 |

## カラム マッピング

### CSV データ読み込みのためのカラム マッピングの構成

データファイルの列が StarRocks テーブルの列と同じ順序で1対1にマッピングされる場合、データファイルと StarRocks テーブルの列の間のカラム マッピングを構成する必要はありません。

データファイルの列が StarRocks テーブルの列と同じ順序で1対1にマッピングされない場合、`columns` パラメータを使用してデータファイルと StarRocks テーブルの間のカラム マッピングを構成する必要があります。これには次の2つの使用事例が含まれます:

- **列の数は同じですが、列の順序が異なります。** **また、データファイルから StarRocks テーブルの列に読み込まれるまでに関数を計算する必要はありません。**

  `columns` パラメータでは、データファイルの列が並べられた順序と同じ順序で StarRocks テーブルの列の名前を指定する必要があります。

  たとえば、StarRocks テーブルは `col1`、`col2`、`col3` の3つの列で構成されており、データファイルも3つの列で構成されており、それらは StarRocks テーブルの列 `col3`、`col2`、`col1` に対応付けられる場合、`"columns: col3, col2, col1"` を指定する必要があります。

- **列の数が異なり、列の順序が異なります。また、データファイルから StarRocks テーブルの列に読み込まれるまでに関数でデータを計算する必要があります。**

  `columns` パラメータでは、データファイルの列が並べられた順序と同じ順序で StarRocks テーブルの列の名前を指定し、データを計算するために使用したい関数を指定する必要があります。2つの例は次の通りです:

  - StarRocks テーブルは `col1`、`col2`、`col3` の3つの列で構成されており、データファイルは4つの列で構成されており、最初の3列が StarRocks テーブルの列 `col1`、`col2`、`col3` と順に対応付けられ、4番目の列は StarRocks テーブルのどの列にも対応付けられない場合、データファイルの4番目の列に一時的な名前を指定する必要があります。一時的な名前は StarRocks テーブルの列名とは異なる必要があります。たとえば、`"columns: col1, col2, col3, temp"` を指定すると、データファイルの4番目の列は一時的に `temp` という名前が付けられます。
  - StarRocks テーブルは `year`、`month`、`day` の3つの列で構成されており、データファイルは`yyyy-mm-dd hh:mm:ss` 形式の日時値が収容されている1つの列のみで構成されています。この場合、`"columns: col, year = year(col), month=month(col), day=day(col)"` を指定すると、`col` がデータファイルの列の一時的な名前であり、関数 `year = year(col)`、`month=month(col)`、`day=day(col)` が使用されてデータファイルの列 `col` からデータを抽出し、それを対応する StarRocks テーブルの列に読み込みます。たとえば、`year = year(col)` は、データファイルの列 `col` から `yyyy` のデータを抽出し、それを StarRocks テーブルの列 `year` に読み込むために使用されます。

詳細な例については、[カラム マッピングの構成](#configure-column-mapping)を参照してください。

### JSON データ読み込みのためのカラム マッピングの構成

JSON ドキュメントのキー名が StarRocks テーブルの列の名前と同じである場合、単純なモードを使用して JSON 形式のデータを読み込むことがでます。単純なモードでは、`jsonpaths` パラメータを指定する必要はありません。このモードでは、JSON 形式のデータが中括弧 `{}` によって示されるオブジェクトである必要があります。たとえば、 `{"category": 1, "author": 2, "price": "3"}` のようにです。この例では、`category`、`author`、`price` がキー名であり、これらのキーは StarRocks テーブルの `category`、`author`、`price` の列名と対応付けることができます。

JSON ドキュメントのキー名が StarRocks テーブルの列の名前と異なる場合、一致モードを使用して JSON 形式のデータを読み込むことがでます。一致モードでは、`jsonpaths` パラメータおよび `COLUMNS` パラメータを使用して、JSON ドキュメントと StarRocks テーブルとの間のカラム マッピングを指定する必要があります:

- `jsonpaths` パラメータでは、JSON キーをJSONドキュメントに配置された順に指定する必要があります。
- `COLUMNS` パラメータでは、以下のことを指定する必要があります:
  - `COLUMNS` パラメータで指定された列名が JSON キーと対応付けられます。
  - `COLUMNS` パラメータで指定された列名が StarRocks テーブルの列名と同じ名前である必要があります。

一致モードを使用して JSON 形式のデータを読み込む例については、[一致モードを使用して JSON データを読み込む](#load-json-data-using-matched-mode)を参照してください。

## 戻り値

読み込みジョブが終了すると、StarRocks は JSON 形式でジョブ結果を返します。例:

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
```JSON
{"StreamLoadPlanTimeMs": 1,
"ReadDataTimeMs": 0,
"WriteDataTimeMs": 11,
"CommitAndPublishTimeMs": 16,
}
```

次の表は、返されたジョブ結果のパラメータを説明しています。

| パラメータ             | 説明                                                     |
| --------------------  | ------------------------------------------------------------ |
| TxnId                 | ロードジョブのトランザクションID。                            |
| Label                 | ロードジョブのラベル。                                       |
| Status                | データの最終ステータス。<ul><li>`Success`: データが正常にロードされ、クエリを実行できます。</li><li>`Publish Timeout`: ロードジョブが正常に送信されましたが、データをまだクエリできません。データを再度ロードする必要はありません。</li><li>`Label Already Exists`: ロードジョブのラベルが他のロードジョブで使用されています。データは正常にロードされている可能性があります。またはロード中です。</li><li>`Fail`: データのロードに失敗しました。ロードジョブを再試行できます。</li></ul> |
| Message               | ロードジョブのステータス。ロードジョブが失敗した場合、詳細な失敗原因が返されます。 |
| NumberTotalRows       | 読み込まれたデータレコードの総数。                       |
| NumberLoadedRows      | 正常にロードされたデータレコードの総数。このパラメータは、`Status` の値が `Success` に設定されているときにのみ有効です。 |
| NumberFilteredRows    | データ品質が不十分でフィルタリングされたデータレコードの数。 |
| NumberUnselectedRows  | WHERE句によってフィルタリングされたデータレコードの数。  |
| LoadBytes             | ロードされたデータ量。単位: バイト。                    |
| LoadTimeMs            | ロードジョブにかかった時間。単位: ms。                   |
| BeginTxnTimeMs        | ロードジョブのトランザクションの実行にかかった時間。      |
| StreamLoadPlanTimeMs  | ロードジョブの実行計画を生成するのにかかった時間。         |
| ReadDataTimeMs        | ロードジョブのデータ読み取りにかかった時間。              |
| WriteDataTimeMs       | ロードジョブのデータ書き込みにかかった時間。              |
| CommitAndPublishTimeMs| ロードジョブのデータコミットおよび公開にかかった時間。    |

ロードジョブが失敗した場合、StarRocksは`ErrorURL`も返します。例：

```JSON
{"ErrorURL": "http://172.26.195.68:8045/api/_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be"}
```

`ErrorURL`は、不適格なデータレコードの詳細を取得できるURLを提供します。オプションのパラメータ `log_rejected_record_num` を使用して、記録される不適格なデータ行の最大数を指定できます。これは、ロードジョブを送信するときに設定されます。

直接`curl "url"`を実行して、フィルタリングされた不適格なデータレコードの詳細を表示できます。また、これらのデータレコードの詳細をエクスポートするには`wget "url"`を実行できます。

```Bash
wget http://172.26.195.68:8045/api/_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be
```

エクスポートされたデータレコードの詳細は、`_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be`という名前のローカルファイルに保存されます。`cat`コマンドを使用してファイルを表示できます。

次に、ロードジョブの構成を調整して、ロードジョブを再送信できます。

## 例

### CSVデータのロード

このセクションでは、CSVデータを使用して、さまざまなパラメータ設定と組み合わせを行う方法を説明します。

#### タイムアウト期間の設定

StarRocksデータベース `test_db` には、`table1`という名前のテーブルが含まれています。このテーブルには、`col1`、`col2`、`col3`の3つの列が含まれています。

データファイル `example1.csv` にも、`col1`、`col2`、`col3`の3つの列が含まれており、これらは `table1` の列 `col1`、`col2`、`col3`に順番にマッピングされます。

`example1.csv`のすべてのデータを、最大100秒以内に`table1`にロードしたい場合、次のコマンドを実行します:

```Bash
curl --location-trusted -u <username>:<password> -H "label:label1" \
    -H "Expect:100-continue" \
    -H "timeout:100" \
    -H "max_filter_ratio:0.2" \
    -T example1.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
```

#### エラートレランスの設定

StarRocksデータベース `test_db` には、`table2`という名前のテーブルが含まれています。このテーブルには、`col1`、`col2`、`col3`の3つの列が含まれています。

データファイル `example2.csv` にも、`col1`、`col2`、`col3`の3つの列が含まれており、これらは `table2` の列 `col1`、`col2`、`col3`に順番にマッピングされます。

`example2.csv`のすべてのデータを、`0.2`を最大エラートレランスとして`table2`にロードしたい場合、次のコマンドを実行します:

```Bash
curl --location-trusted -u <username>:<password> -H "label:label2" \
    -H "Expect:100-continue" \
    -H "max_filter_ratio:0.2" \
    -T example2.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table2/_stream_load
```

#### 列マッピングの設定

StarRocksデータベース `test_db` には、`table3`という名前のテーブルが含まれています。このテーブルには、`col1`、`col2`、`col3`の3つの列が含まれています。

データファイル `example3.csv` にも、`col1`、`col2`、`col3`の3つの列が含まれており、これらは `table3` の列 `col2`、`col1`、`col3`に順番にマッピングされます。

`example3.csv`のすべてのデータを`table3`にロードしたい場合、次のコマンドを実行します:

```Bash
curl --location-trusted -u <username>:<password> -H "label:label3" \
    -H "Expect:100-continue" \
    -H "columns: col2, col1, col3" \
    -T example3.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table3/_stream_load
```

> **注意**
>
> 前述の例では、`example3.csv`の列を、`table3`に配置された列と同じ順序でマッピングできないため、`columns`パラメータを使用して`example3.csv`と`table3`の列の一時的な名前を定義する必要があります。

#### フィルタ条件の設定

StarRocksデータベース `test_db` には、`table4`という名前のテーブルが含まれています。このテーブルには、`col1`、`col2`、`col3`の3つの列が含まれています。

データファイル `example4.csv` にも、`col1`、`col2`、`col3`の3つの列が含まれており、これらは `table4` の列 `col1`、`col2`、`col3`に順番にマッピングされます。

`example4.csv`の最初の列の値が`20180601`と等しいデータレコードのみを`table4`にロードしたい場合、次のコマンドを実行します:

```Bash
curl --location-trusted -u <username>:<password> -H "label:label4" \
    -H "Expect:100-continue" \
    -H "columns: col1, col2, col3]" \
    -H "where: col1 = 20180601" \
    -T example4.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table4/_stream_load
```

> **注意**
>
> 前述の例では、`example4.csv`と`table4`は順番にマッピングできる同じ数の列を持っていますが、WHERE句を使用して列ベースのフィルタ条件を指定する必要があります。したがって、`columns`パラメータを使用して`example4.csv`の列の一時的な名前を定義する必要があります。

#### 宛先パーティションの設定

StarRocksデータベース `test_db` には、`table5`という名前のテーブルが含まれています。このテーブルには、`col1`、`col2`、`col3`の3つの列が含まれています。

データファイル `example5.csv` にも、`col1`、`col2`、`col3`の3つの列が含まれており、これらは `table5` の列 `col1`、`col2`、`col3`に順番にマッピングされます。

`example5.csv`のすべてのデータを、`table5`のパーティション`p1`と`p2`にロードしたい場合、次のコマンドを実行します:

```Bash
curl --location-trusted -u <username>:<password> -H "label:label5" \
    -H "Expect:100-continue" \
    -H "partitions: p1, p2" \
    -T example5.csv -XPUT \
```Japanese
    http://<fe_host>:<fe_http_port>/api/test_db/table5/_stream_load
```

#### 厳密モードとタイムゾーンを設定する

お客様のStarRocksデータベース `test_db` には`table6`という名前のテーブルが含まれています。このテーブルには、`col1`、`col2`、`col3`という3つの列が含まれています。

あなたのデータファイル`example6.csv`にも3つの列が含まれており、これらは`table6`の`col1`、`col2`、`col3`にシーケンスに割り当てることができます。

`example6.csv`からすべてのデータを`table6`に厳密モードとタイムゾーン`Africa/Abidjan`を使用してロードしたい場合は、次のコマンドを実行してください:

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "strict_mode: true" \
    -H "timezone: Africa/Abidjan" \
    -T example6.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table6/_stream_load
```

#### HLL型の列を含むテーブルにデータをロードする

お客様のStarRocksデータベース `test_db` には`table7`という名前のテーブルが含まれています。このテーブルにはHLL型の列`col1`と`col2`が含まれています。

あなたのデータファイル`example7.csv`にも2つの列が含まれており、そのうち1つの列は`table7`の`col1`に割り当てることができ、2つ目の列は`table7`のどの列にも割り当てることができません。`example7.csv`の最初の列の値は、`col1`にロードされる前に関数を使用してHLL型のデータに変換することができます。

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
> 前述の例では、`columns`パラメータを使用して`example7.csv`の2つの列の名前をそれぞれ`temp1`と`temp2`に割り当てます。それから以下のようにデータを変換するために関数を使用します:
>
> - `hll_hash`関数は`example7.csv`の`temp1`の値をHLL型のデータに変換し、`example7.csv`の`temp1`を`table7`の`col1`にマップします。
>
> - `hll_empty`関数は`table7`の`col2`に指定されたデフォルト値を埋めるために使用されます。

関数`hll_hash`と`hll_empty`の使用方法については、[hll_hash](../../sql-functions/aggregate-functions/hll_hash.md) と [hll_empty](../../sql-functions/aggregate-functions/hll_empty.md) を参照してください。

#### BITMAP型の列を含むテーブルにデータをロードする

お客様のStarRocksデータベース `test_db` には`table8`という名前のテーブルが含まれています。このテーブルにはBITMAP型の列`col1`と`col2`が含まれています。

あなたのデータファイル`example8.csv`にも2つの列が含まれており、そのうち1つの列は`table8`の`col1`に割り当てることができ、2つ目の列は`table8`のどの列にも割り当てることができません。`example8.csv`の最初の列の値は、`col1`にロードされる前に関数を使用してBITMAP型のデータに変換することができます。

`example8.csv`からデータを`table8`にロードしたい場合は、次のコマンドを実行してください:

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "columns: temp1, temp2, col1=to_bitmap(temp1), col2=bitmap_empty()" \
    -T example8.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table8/_stream_load
```

> **注記**
>
> 前述の例では、`columns`パラメータを使用して`example8.csv`の2つの列の名前をそれぞれ`temp1`と`temp2`に割り当てます。それから以下のようにデータを変換するために関数を使用します:
>
> - `to_bitmap`関数は`example8.csv`の`temp1`の値をBITMAP型のデータに変換し、`example8.csv`の`temp1`を`table8`の`col1`にマップします。
>
> - `bitmap_empty`関数は`table8`の`col2`に指定されたデフォルト値を埋めるために使用されます。

関数`to_bitmap`と`bitmap_empty`の使用方法については、[to_bitmap](../../../sql-reference/sql-functions/bitmap-functions/to_bitmap.md) と [bitmap_empty](../../../sql-reference/sql-functions/bitmap-functions/bitmap_empty.md) を参照してください。

#### `skip_header`、`trim_space`、`enclose`、`escape`の設定

お客様のStarRocksデータベース `test_db` には`table9`という名前のテーブルが含まれています。このテーブルには、`col1`、`col2`、`col3`という3つの列が含まれています。

あなたのデータファイル`example9.csv`にも3つの列が含まれており、これらは`table13`の`col2`、`col1`、`col3`にシーケンスに割り当てることができます。

`example9.csv`からすべてのデータを`table9`にロードしたい場合は、`example9.csv`の最初の5行をスキップし、列の区切り文字の前後のスペースを削除し、`enclose`を`\`に、`escape`を`\`に設定して、次のコマンドを実行してください:

```Bash
curl --location-trusted -u <username>:<password> -H "label:3875" \
    -H "Expect:100-continue" \
    -H "trim_space: true" -H "skip_header: 5" \
    -H "column_separator:," -H "enclose:\"" -H "escape:\\" \
    -H "columns: col2, col1, col3" \
    -T example9.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/tbl9/_stream_load
```

### JSON データをロードする

このセクションでは、JSONデータをロードする際に注意する必要のあるパラメータ設定について説明します。

お客様のStarRocksデータベース `test_db` には`tbl1`という名前のテーブルが含まれており、そのスキーマは以下のとおりです:

```SQL
`category` varchar(512) NULL COMMENT "",`author` varchar(512) NULL COMMENT "",`title` varchar(512) NULL COMMENT "",`price` double NULL COMMENT ""
```

#### シンプルモードを使用してJSONデータをロードする

あなたのデータファイル`example1.json`が以下のデータを含む場合:

```JSON
{"category":"C++","author":"avc","title":"C++ primer","price":895}
```

`example1.json`からすべてのデータを`tbl1`にロードしたい場合は、次のコマンドを実行してください:

```Bash
curl --location-trusted -u <username>:<password> -H "label:label6" \
    -H "Expect:100-continue" \
    -H "format: json" \
    -T example1.json -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/tbl1/_stream_load
```

> **注記**
>
> 前述の例では、`columns`と`jsonpaths`パラメータが指定されていません。そのため、`example1.json`のキーは`tbl1`の列に名前によってマッピングされています。

スループットを向上させるために、Stream Loadは一度に複数のデータレコードをロードすることができます。例:

```JSON
[{"category":"C++","author":"avc","title":"C++ primer","price":89.5},{"category":"Java","author":"avc","title":"Effective Java","price":95},{"category":"Linux","author":"avc","title":"Linux kernel","price":195}]
```

#### 一致モードを使用してJSONデータをロードする

StarRocksは以下の手順でJSONデータを一致させて処理します:

1. (オプション) `strip_outer_array`パラメータ設定によって、最外層の配列構造を削除します。

   > **注記**
   >
   > このステップは、JSONデータの最外層が角括弧 `[]` で示される配列構造である場合のみ実行されます。`strip_outer_array`を`true`に設定する必要があります。

2. (オプション) `json_root`パラメータ設定によって、JSONデータのルート要素を一致させます。

   > **注記**
   >
   > このステップは、JSONデータにルート要素がある場合にのみ実行されます。`json_root`パラメータを使用してルート要素を指定する必要があります。

3. `jsonpaths`パラメータ設定によって指定されたJSONデータを抽出します。

##### ルート要素が指定されていない一致モードを使用してJSONデータをロードする

あなたのデータファイル`example2.json`が以下のデータを含む場合:
```JSON
{"category":"xuxb111","author":"1avc","price":895},{"category":"xuxb222","author":"2avc","price":895},{"category":"xuxb333","author":"3avc","price":895}
```

`example2.json`から`category`、`author`、`price`のみを読み込むには、次のコマンドを実行します。

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
> 前の例では、JSONデータの最も外側の層が角括弧`[]`で示される配列構造であることが示されています。配列構造は、各々がデータレコードを表す複数のJSONオブジェクトから構成されています。そのため、`strip_outer_array`を`true`に設定して最も外側の配列構造を削除する必要があります。読み込み対象外の**title**キーは読み込み中に無視されます。

##### ルート要素が指定されたマッチングモードを使用してJSONデータを読み込む

データファイル `example3.json` が以下のデータで構成されているとします。

```JSON
{"id": 10001,"RECORDS":[{"category":"11","title":"SayingsoftheCentury","price":895,"timestamp":1589191587},{"category":"22","author":"2avc","price":895,"timestamp":1589191487},{"category":"33","author":"3avc","title":"SayingsoftheCentury","timestamp":1589191387}],"comments": ["3 records", "there will be 3 rows"]}
```

`example3.json`から`category`、`author`、`price`のみを読み込むには、次のコマンドを実行します。

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
> 前の例では、JSONデータの最も外側の層が角括弧`[]`で示される配列構造であることが示されています。配列構造は、各々がデータレコードを表す複数のJSONオブジェクトから構成されています。そのため、`strip_outer_array`を`true`に設定して最も外側の配列構造を削除する必要があります。読み込み対象外の**title**と**timestamp**キーは読み込み中に無視されます。さらに、`json_root`パラメータは、JSONデータのルート要素である配列を指定するために使用されます。