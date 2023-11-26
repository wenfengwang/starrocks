---
displayed_sidebar: "Japanese"
---

# ストリームロード

## 説明

StarRocksでは、HTTPベースのストリームロードというロード方法を提供しており、ローカルファイルシステムやストリーミングデータソースからデータをロードするのに役立ちます。ロードジョブを送信すると、StarRocksは同期的にジョブを実行し、ジョブが完了した後にジョブの結果を返します。ジョブの結果に基づいてジョブが成功したかどうかを判断することができます。ストリームロードのアプリケーションシナリオ、制限、原則、およびサポートされているデータファイル形式についての詳細については、[ローカルファイルシステムからのストリームロード](../../../loading/StreamLoad.md#loading-from-a-local-file-system-via-stream-load)を参照してください。

> **注意**
>
> - Stream Loadを使用してStarRocksテーブルにデータをロードした後、そのテーブルに作成されたマテリアライズドビューのデータも更新されます。
> - StarRocksテーブルにデータをロードするには、StarRocksテーブルに対するINSERT権限を持つユーザーとしてのみデータをロードできます。INSERT権限がない場合は、[GRANT](../account-management/GRANT.md)でINSERT権限を付与するための手順に従ってください。

## 構文

```Bash
curl --location-trusted -u <username>:<password> -XPUT <url>
(
    data_desc
)
[opt_properties]        
```

このトピックでは、curlを例に挙げて、ストリームロードを使用してデータをロードする方法を説明します。curl以外にも、他のHTTP互換のツールや言語を使用してストリームロードを実行することもできます。ロード関連のパラメータは、HTTPリクエストヘッダフィールドに含まれます。これらのパラメータを入力する際には、次の点に注意してください。

- このトピックで示されているように、チャンク化された転送エンコーディングを使用することができます。チャンク化された転送エンコーディングを選択しない場合は、転送するコンテンツの長さを示す`Content-Length`ヘッダフィールドを入力する必要があります。これにより、データの整合性が確保されます。

  > **注意**
  >
  > curlを使用してストリームロードを実行する場合、StarRocksは自動的に`Content-Length`ヘッダフィールドを追加しますので、手動で入力する必要はありません。

- `Expect`ヘッダフィールドを追加し、その値を`100-continue`と指定します。これにより、ジョブリクエストが拒否された場合に不要なデータ転送を防ぎ、リソースのオーバーヘッドを削減するのに役立ちます。

StarRocksでは、いくつかのリテラルがSQL言語によって予約されたキーワードとして使用されます。SQL文でこれらのキーワードを直接使用しないでください。SQL文でこのようなキーワードを使用する場合は、バッククォート（`）で囲んでください。詳細については、[キーワード](../../../sql-reference/sql-statements/keywords.md)を参照してください。

## パラメータ

### usernameとpassword

StarRocksクラスタに接続するために使用するアカウントのユーザー名とパスワードを指定します。このパラメータは必須です。パスワードが設定されていないアカウントを使用する場合は、`<username>:`のみを入力する必要があります。

### XPUT

HTTPリクエストメソッドを指定します。このパラメータは必須です。ストリームロードはPUTメソッドのみをサポートしています。

### url

StarRocksテーブルのURLを指定します。構文:

```Plain
http://<fe_host>:<fe_http_port>/api/<database_name>/<table_name>/_stream_load
```

以下の表は、URLのパラメータについて説明しています。

| パラメータ     | 必須 | 説明                                                         |
| ------------- | ---- | ------------------------------------------------------------ |
| fe_host       | Yes  | StarRocksクラスタのFEノードのIPアドレス。<br/>**注意**<br/>ロードジョブを特定のBEノードに送信する場合は、BEノードのIPアドレスを入力する必要があります。 |
| fe_http_port  | Yes  | StarRocksクラスタのFEノードのHTTPポート番号。デフォルトのポート番号は`8030`です。<br/>**注意**<br/>ロードジョブを特定のBEノードに送信する場合は、BEノードのHTTPポート番号を入力する必要があります。デフォルトのポート番号は`8030`です。 |
| database_name | Yes  | StarRocksテーブルが所属するデータベースの名前。               |
| table_name    | Yes  | StarRocksテーブルの名前。                                    |

> **注意**
>
> [SHOW FRONTENDS](../Administration/SHOW_FRONTENDS.md)を使用して、FEノードのIPアドレスとHTTPポートを表示することができます。

### data_desc

ロードするデータファイルに関する情報を記述します。`data_desc`ディスクリプタには、データファイルの名前、形式、列セパレータ、行セパレータ、宛先パーティション、およびStarRocksテーブルに対する列マッピングなどが含まれます。構文:

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

`data_desc`ディスクリプタのパラメータは、次の3つのタイプに分類できます: 一般的なパラメータ、CSVパラメータ、およびJSONパラメータ。

#### 一般的なパラメータ

| パラメータ  | 必須 | 説明                                                         |
| ---------- | ---- | ------------------------------------------------------------ |
| file_path  | Yes  | データファイルの保存パス。ファイル名の拡張子を省略することもできます。 |
| format     | No   | データファイルの形式。有効な値: `CSV`および`JSON`。デフォルト値: `CSV`。 |
| partitions | No   | データファイルをロードするパーティション。このパラメータを指定しない場合、StarRocksはデータファイルをStarRocksテーブルのすべてのパーティションにロードします。 |
| temporary_partitions|  No   | データファイルをロードする[一時パーティション](../../../table_design/Temporary_partition.md)の名前。複数の一時パーティションを指定することができますが、カンマ（,）で区切る必要があります。|
| columns    | No   | データファイルとStarRocksテーブルの間の列マッピング。<br/>データファイルのフィールドがStarRocksテーブルの列に順番にマッピングできる場合、このパラメータを指定する必要はありません。代わりに、このパラメータを使用してデータ変換を実装することができます。たとえば、CSV形式のデータファイルをロードし、ファイルが2つの列からなり、これらの列がStarRocksテーブルの2つの列`id`と`city`に順番にマッピングできる場合、`"columns: city,tmp_id, id = tmp_id * 100"`と指定することができます。詳細については、このトピックの"[列マッピング](#column-mapping)"セクションを参照してください。 |

#### CSVパラメータ

| パラメータ        | 必須 | 説明                                                         |
| ---------------- | ---- | ------------------------------------------------------------ |
| column_separator | No   | データファイルでフィールドを区切るために使用される文字。このパラメータを指定しない場合、このパラメータのデフォルト値は`\t`であり、タブを示します。<br/>このパラメータで指定する列セパレータがデータファイルで使用されている列セパレータと同じであることを確認してください。<br/>**注意**<br/>CSVデータの場合、50バイトを超えないUTF-8文字列（カンマ（,）、タブ、またはパイプ（\|）など）をテキストデリミタとして使用することができます。 |
| row_delimiter    | No   | データファイルで行を区切るために使用される文字。このパラメータを指定しない場合、このパラメータのデフォルト値は`\n`です。 |
| skip_header      | No   | データファイルがCSV形式の場合、データファイルの最初の数行をスキップするかどうかを指定します。タイプ: INTEGER。デフォルト値: `0`。<br />一部のCSV形式のデータファイルでは、最初の数行が列名や列のデータ型などのメタデータを定義するために使用されます。`skip_header`パラメータを設定することで、データロード中にデータファイルの最初の数行をスキップすることができます。たとえば、このパラメータを`1`に設定すると、データロード中にデータファイルの最初の行がスキップされます。<br />データファイルの最初の数行は、ロードコマンドで指定した行セパレータで区切る必要があります。 |
| trim_space       | No   | CSV形式のデータファイルの場合、データファイルから列セパレータの前後のスペースを削除するかどうかを指定します。タイプ: BOOLEAN。デフォルト値: `false`。<br />一部のデータベースでは、CSV形式のデータファイルをエクスポートする際に、列セパレータにスペースが追加されます。このようなスペースは、その位置に応じて先行スペースまたは後続スペースと呼ばれます。`trim_space`パラメータを設定することで、データロード中にこのような不要なスペースを削除することができます。<br />なお、StarRocksは、`enclose`で指定された文字で囲まれたフィールド内のスペース（先行スペースおよび後続スペースを含む）を削除しません。たとえば、次のフィールド値は、パイプ（<code class="language-text">&#124;</code>）を列セパレータとし、ダブルクォーテーションマーク（`"`）を`enclose`で指定された文字として使用しています:<br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> <br /><code class="language-text">&#124;" Love StarRocks "&#124;</code> <br /><code class="language-text">&#124; "Love StarRocks" &#124;</code> <br />`trim_space`を`true`に設定すると、StarRocksは次のように前のフィールド値を処理します:<br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> <br /><code class="language-text">&#124;" Love StarRocks "&#124;</code> <br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> |
| enclose          | No   | [RFC4180](https://www.rfc-editor.org/rfc/rfc4180)に従って、データファイルのフィールド値を囲むために使用される文字を指定します。タイプ: 半角文字。デフォルト値: `NONE`。最も一般的な文字はシングルクォーテーションマーク（`'`）とダブルクォーテーションマーク（`"`）です。<br />`enclose`で指定された文字で囲まれたすべての特殊文字（行セパレータや列セパレータを含む）は通常のシンボルと見なされます。StarRocksはRFC4180以上のことができるため、`enclose`で指定された文字を任意の半角文字として指定することができます。<br />フィールド値に`enclose`で指定された文字が含まれる場合は、同じ文字を使用してその`enclose`で指定された文字をエスケープすることができます。たとえば、`enclose`を`"`に設定し、フィールド値が`a "quoted" c`である場合、フィールド値を`"a ""quoted"" c"`としてデータファイルに入力することができます。 |
| escape           | No   | 行セパレータ、列セパレータ、エスケープ文字、および`enclose`で指定された文字など、さまざまな特殊文字をエスケープするために使用される文字を指定します。タイプ: 半角文字。デフォルト値: `NONE`。最も一般的な文字はスラッシュ（`\`）ですが、SQL文ではスラッシュを2重スラッシュ（`\\`）として記述する必要があります。<br />**注意**<br />`escape`で指定された文字は、`enclose`で指定された文字のペアの内側と外側の両方に適用されます。<br />2つの例は次のとおりです:<ul><li>`enclose`を`"`に設定し、`escape`を`\`に設定すると、StarRocksは`"say \"Hello world\""`を`say "Hello world"`に解析します。</li><li>列セパレータがカンマ（`,`）であると仮定します。`escape`を`\`に設定すると、StarRocksは`a, b\, c`を2つの別々のフィールド値、`a`と`b, c`に解析します。</li></ul> |

> **注意**
>
> - CSVデータの場合、50バイトを超えないUTF-8文字列（カンマ（,）、タブ、またはパイプ（|）など）をテキストデリミタとして使用することができます。
> - Null値は`\N`を使用して示されます。たとえば、データファイルが3つの列からなり、そのデータファイルのレコードが最初の列と3番目の列にデータを保持しているが、2番目の列にデータを保持していない場合、2番目の列にはnull値を示すために`\N`を使用する必要があります。これは、レコードを`a,\N,b`としてコンパイルする必要があることを意味します。`a,,b`は、レコードの2番目の列に空の文字列が保持されていることを示します。
> - `skip_header`、`trim_space`、`enclose`、および`escape`などの形式オプションは、v3.0以降でサポートされています。

#### JSONパラメータ

| パラメータ         | 必須 | 説明                                                         |
| ----------------- | ---- | ------------------------------------------------------------ |
| jsonpaths         | No   | JSONデータファイルからロードするキーの名前。マッチングモードでJSONデータをロードする場合にのみ、このパラメータを指定する必要があります。このパラメータの値はJSON形式です。[JSONデータロードのための列マッピングの設定](#configure-column-mapping-for-json-data-loading)を参照してください。           |
| strip_outer_array | No   | 最外部の配列構造を削除するかどうかを指定します。有効な値: `true`および`false`。デフォルト値: `false`。<br/>実際のビジネスシナリオでは、JSONデータには角括弧`[]`で示される最外部の配列構造が存在する場合があります。この場合、`strip_outer_array`を`true`に設定することをお勧めします。これにより、StarRocksは最外部の角括弧`[]`を削除し、各内部配列を個別のデータレコードとしてロードします。`strip_outer_array`を`false`に設定すると、StarRocksはJSONデータ全体を1つの配列として解析し、配列を単一のデータレコードとしてロードします。<br/>たとえば、JSONデータが`[ {"category" : 1, "author" : 2}, {"category" : 3, "author" : 4} ]`の場合、`strip_outer_array`を`true`に設定すると、`{"category" : 1, "author" : 2}`および`{"category" : 3, "author" : 4}`が個別のデータレコードとして解析され、個別のStarRocksテーブル行にロードされます。 |
| json_root         | No   | JSONデータファイルからロードするJSONデータのルート要素。マッチングモードでJSONデータをロードする場合にのみ、このパラメータを指定する必要があります。このパラメータの値は有効なJsonPath文字列です。デフォルトでは、このパラメータの値は空であり、JSONデータファイルのすべてのデータがロードされます。詳細については、このトピックの"[ルート要素を指定してマッチングモードでJSONデータをロードする](#load-json-data-using-matched-mode-with-root-element-specified)"セクションを参照してください。 |
| ignore_json_size  | No   | HTTPリクエストのJSON本文のサイズをチェックするかどうかを指定します。<br/>**注意**<br/>デフォルトでは、HTTPリクエストのJSON本文のサイズは100MBを超えることはできません。JSON本文のサイズが100MBを超える場合、エラーメッセージ「The size of this batch exceed the max size [104857600] of json type data data [8617627793]. Set ignore_json_size to skip check, although it may lead huge memory consuming.」が報告されます。このエラーを防ぐために、HTTPリクエストヘッダに`"ignore_json_size:true"`を追加して、StarRocksにJSON本文のサイズをチェックしないように指示することができます。 |

JSONデータをロードする際には、JSONオブジェクトごとのサイズが4GBを超えてはなりません。JSONデータファイルの個々のJSONオブジェクトが4GBを超える場合、「This parser can't support a document that big.」というエラーが報告されます。

### opt_properties

ロードジョブ全体に適用されるいくつかのオプションパラメータを指定します。構文:

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

以下の表は、返されるジョブ結果のパラメータについて説明しています。

| パラメータ              | 説明                                                         |
| ---------------------- | ------------------------------------------------------------ |
| TxnId                  | ロードジョブのトランザクションID。                          |
| Label                  | ロードジョブのラベル。                                       |
| Status                 | データのロードの最終的なステータス。<ul><li>`Success`: データが正常にロードされ、クエリを実行できます。</li><li>`Publish Timeout`: ロードジョブは正常に送信されましたが、データはまだクエリできません。データのロードを再試行する必要はありません。</li><li>`Label Already Exists`: ロードジョブのラベルは他のロードジョブで使用されています。データは正常にロードされている可能性があります。</li><li>`Fail`: データのロードに失敗しました。ロードジョブを再試行することができます。</li></ul> |
| Message                | ロードジョブのステータス。ロードジョブが失敗した場合、詳細な失敗原因が返されます。 |
| NumberTotalRows        | 読み取られたデータレコードの総数。                          |
| NumberLoadedRows       | 正常にロードされたデータレコードの総数。このパラメータは、`Status`が`Success`の場合にのみ有効です。 |
| NumberFilteredRows     | データ品質が不十分なためにフィルタリングされたデータレコードの数。 |
| NumberUnselectedRows   | WHERE句によってフィルタリングされたデータレコードの数。     |
| LoadBytes              | ロードされたデータの量。単位: バイト。                      |
| LoadTimeMs             | ロードジョブにかかった時間。単位: ミリ秒。                   |
| BeginTxnTimeMs         | ロードジョブのトランザクションを実行するのにかかった時間。    |
| StreamLoadPlanTimeMs   | ロードジョブの実行計画を生成するのにかかった時間。            |
| ReadDataTimeMs         | データを読み取るのにかかった時間。                            |
| WriteDataTimeMs        | データを書き込むのにかかった時間。                            |
| CommitAndPublishTimeMs | データをコミットおよび公開するのにかかった時間。              |

ロードジョブが失敗した場合、StarRocksは`ErrorURL`も返します。例:

```JSON
{"ErrorURL": "http://172.26.195.68:8045/api/_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be"}
```

`ErrorURL`は、フィルタリングされた不適格なデータレコードの詳細を取得するためのURLを提供します。ロードジョブを送信する際に設定したオプションパラメータ`log_rejected_record_num`で、フィルタリングされた不適格なデータ行の最大数を指定することができます。

`curl "url"`を実行してフィルタリングされた不適格なデータレコードの詳細を直接表示することができます。また、`wget "url"`を実行してこれらのデータレコードの詳細をエクスポートすることもできます:

```Bash
```
wget http://172.26.195.68:8045/api/_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be
```

エクスポートされたデータレコードの詳細は、`_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be`という名前のローカルファイルに保存されます。`cat`コマンドを使用してファイルを表示することができます。

その後、ロードジョブの設定を調整し、ロードジョブを再度送信できます。

## 例

### CSVデータのロード

このセクションでは、CSVデータを例として使用して、さまざまなパラメータ設定と組み合わせを使用して、さまざまなロード要件を満たす方法を説明します。

#### タイムアウト期間の設定

StarRocksデータベース`test_db`には、`table1`という名前のテーブルがあります。テーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

データファイル`example1.csv`も3つの列で構成されており、これらの列は`table1`の`col1`、`col2`、`col3`に順番にマッピングできます。

`example1.csv`のすべてのデータを最大100秒以内に`table1`にロードする場合、次のコマンドを実行します。

```Bash
curl --location-trusted -u <username>:<password> -H "label:label1" \
    -H "Expect:100-continue" \
    -H "timeout:100" \
    -H "max_filter_ratio:0.2" \
    -T example1.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
```

#### エラートレランスの設定

StarRocksデータベース`test_db`には、`table2`という名前のテーブルがあります。テーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

データファイル`example2.csv`も3つの列で構成されており、これらの列は`table2`の`col1`、`col2`、`col3`に順番にマッピングできます。

`example2.csv`のすべてのデータを最大エラートレランス`0.2`で`table2`にロードする場合、次のコマンドを実行します。

```Bash
curl --location-trusted -u <username>:<password> -H "label:label2" \
    -H "Expect:100-continue" \
    -H "max_filter_ratio:0.2" \
    -T example2.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table2/_stream_load
```

#### 列マッピングの設定

StarRocksデータベース`test_db`には、`table3`という名前のテーブルがあります。テーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

データファイル`example3.csv`も3つの列で構成されており、これらの列は`table3`の`col2`、`col1`、`col3`に順番にマッピングできます。

`example3.csv`のすべてのデータを`table3`にロードする場合、次のコマンドを実行します。

```Bash
curl --location-trusted -u <username>:<password>  -H "label:label3" \
    -H "Expect:100-continue" \
    -H "columns: col2, col1, col3" \
    -T example3.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table3/_stream_load
```

> **注意**
>
> 上記の例では、`example3.csv`の列は、`table3`の列の並び順と同じ順序でマッピングすることはできません。そのため、`columns`パラメータを使用して`example3.csv`と`table3`の列のマッピングを設定する必要があります。

#### フィルタ条件の設定

StarRocksデータベース`test_db`には、`table4`という名前のテーブルがあります。テーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

データファイル`example4.csv`も3つの列で構成されており、これらの列は`table4`の`col1`、`col2`、`col3`に順番にマッピングできます。

`example4.csv`の最初の列の値が`20180601`と等しいデータレコードのみを`table4`にロードする場合、次のコマンドを実行します。

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
> 上記の例では、`example4.csv`と`table4`はマッピング可能な列の数が同じですが、列ベースのフィルタ条件を指定するためにWHERE句を使用する必要があります。そのため、`columns`パラメータを使用して`example4.csv`の列に一時的な名前を定義する必要があります。

#### 宛先パーティションの設定

StarRocksデータベース`test_db`には、`table5`という名前のテーブルがあります。テーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

データファイル`example5.csv`も3つの列で構成されており、これらの列は`table5`の`col1`、`col2`、`col3`に順番にマッピングできます。

`example5.csv`のすべてのデータを`table5`のパーティション`p1`と`p2`にロードする場合、次のコマンドを実行します。

```Bash
curl --location-trusted -u <username>:<password>  -H "label:label5" \
    -H "Expect:100-continue" \
    -H "partitions: p1, p2" \
    -T example5.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table5/_stream_load
```

#### 厳密モードとタイムゾーンの設定

StarRocksデータベース`test_db`には、`table6`という名前のテーブルがあります。テーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

データファイル`example6.csv`も3つの列で構成されており、これらの列は`table6`の`col1`、`col2`、`col3`に順番にマッピングできます。

`example6.csv`のすべてのデータを`table6`にロードする際に、厳密モードとタイムゾーン`Africa/Abidjan`を使用する場合、次のコマンドを実行します。

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "strict_mode: true" \
    -H "timezone: Africa/Abidjan" \
    -T example6.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table6/_stream_load
```

#### HLL型の列を含むテーブルにデータをロードする

StarRocksデータベース`test_db`には、`table7`という名前のテーブルがあります。テーブルは、`col1`と`col2`の2つのHLL型の列で構成されています。

データファイル`example7.csv`も2つの列で構成されており、最初の列は`table7`の`col1`にマッピングできますが、2番目の列は`table7`のいずれの列にもマッピングできません。`example7.csv`の最初の列の値は、ロードされる前に関数を使用してHLL型のデータに変換することができます。

`example7.csv`のデータを`table7`にロードする場合、次のコマンドを実行します。

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "columns: temp1, temp2, col1=hll_hash(temp1), col2=hll_empty()" \
    -T example7.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table7/_stream_load
```

> **注意**
>
> 上記の例では、`example7.csv`の2つの列は、`columns`パラメータを使用して`temp1`と`temp2`という名前に設定されます。その後、次のようにデータを変換するために関数が使用されます。
>
> - `hll_hash`関数は、`example7.csv`の`temp1`の値をHLL型のデータに変換し、`example7.csv`の`temp1`を`table7`の`col1`にマッピングします。
>
> - `hll_empty`関数は、指定されたデフォルト値を`table7`の`col2`に埋め込みます。

関数`hll_hash`および`hll_empty`の使用方法については、[hll_hash](../../sql-functions/aggregate-functions/hll_hash.md)および[hll_empty](../../sql-functions/aggregate-functions/hll_empty.md)を参照してください。

#### BITMAP型の列を含むテーブルにデータをロードする

StarRocksデータベース`test_db`には、`table8`という名前のテーブルがあります。テーブルは、`col1`と`col2`の2つのBITMAP型の列で構成されています。

データファイル`example8.csv`も2つの列で構成されており、最初の列は`table8`の`col1`にマッピングできますが、2番目の列は`table8`のいずれの列にもマッピングできません。`example8.csv`の最初の列の値は、関数を使用して変換することができます。

`example8.csv`のデータを`table8`にロードする場合、次のコマンドを実行します。

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "columns: temp1, temp2, col1=to_bitmap(temp1), col2=bitmap_empty()" \
    -T example8.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table8/_stream_load
```

> **注意**
>
> 上記の例では、`example8.csv`の2つの列は、`columns`パラメータを使用して`temp1`と`temp2`という名前に設定されます。その後、次のようにデータを変換するために関数が使用されます。
>
> - `to_bitmap`関数は、`example8.csv`の`temp1`の値をBITMAP型のデータに変換し、`example8.csv`の`temp1`を`table8`の`col1`にマッピングします。
>
> - `bitmap_empty`関数は、指定されたデフォルト値を`table8`の`col2`に埋め込みます。

関数`to_bitmap`および`bitmap_empty`の使用方法については、[to_bitmap](../../../sql-reference/sql-functions/bitmap-functions/to_bitmap.md)および[bitmap_empty](../../../sql-reference/sql-functions/bitmap-functions/bitmap_empty.md)を参照してください。

#### `skip_header`、`trim_space`、`enclose`、および`escape`の設定

StarRocksデータベース`test_db`には、`table9`という名前のテーブルがあります。テーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

データファイル`example9.csv`も3つの列で構成されており、これらの列は`table13`の`col2`、`col1`、`col3`に順番にマッピングできます。

`example9.csv`のすべてのデータを`table9`にロードする場合、最初の5行をスキップし、列セパレータの前後のスペースを削除し、`enclose`を`\`、`escape`を`\`に設定する意図で、次のコマンドを実行します。

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

StarRocksデータベース`test_db`には、`tbl1`という名前のテーブルがあり、そのスキーマは次のようになっています。

```SQL
`category` varchar(512) NULL COMMENT "",`author` varchar(512) NULL COMMENT "",`title` varchar(512) NULL COMMENT "",`price` double NULL COMMENT ""
```

#### シンプルモードを使用してJSONデータをロードする

データファイル`example1.json`が次のデータを含んでいるとします。

```JSON
{"category":"C++","author":"avc","title":"C++ primer","price":895}
```

`example1.json`のすべてのデータを`tbl1`にロードする場合、次のコマンドを実行します。

```Bash
curl --location-trusted -u <username>:<password> -H "label:label6" \
    -H "Expect:100-continue" \
    -H "format: json" \
    -T example1.json -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/tbl1/_stream_load
```

> **注意**
>
> 上記の例では、`columns`および`jsonpaths`パラメータは指定されていません。そのため、`example1.json`のキーは名前によって`tbl1`の列にマッピングされます。

スループットを向上させるために、Stream Loadは一度に複数のデータレコードをロードすることができます。例:

```JSON
[{"category":"C++","author":"avc","title":"C++ primer","price":89.5},{"category":"Java","author":"avc","title":"Effective Java","price":95},{"category":"Linux","author":"avc","title":"Linux kernel","price":195}]
```

#### マッチングモードを使用してJSONデータをロードする

StarRocksは、JSONデータをマッチングおよび処理するために次の手順を実行します。

1. (オプション) `strip_outer_array`パラメータ設定によって指示されたように、最外層の配列構造を削除します。

   > **注意**
   >
   > この手順は、JSONデータの最外層が角かっこ`[]`で囲まれた配列構造である場合にのみ実行されます。`strip_outer_array`を`true`に設定する必要があります。

2. (オプション) `json_root`パラメータ設定によって指示されたように、JSONデータのルート要素をマッチングします。

   > **注意**
   >
   > この手順は、JSONデータにルート要素がある場合にのみ実行されます。`json_root`パラメータを使用してルート要素を指定する必要があります。

3. `jsonpaths`パラメータ設定によって指定されたJSONデータを抽出します。

##### ルート要素が指定されていないマッチングモードでJSONデータをロードする

データファイル`example2.json`が次のデータを含んでいるとします。

```JSON
[{"category":"xuxb111","author":"1avc","title":"SayingsoftheCentury","price":895},{"category":"xuxb222","author":"2avc","title":"SayingsoftheCentury","price":895},{"category":"xuxb333","author":"3avc","title":"SayingsoftheCentury","price":895}]
```

`example2.json`から`category`、`author`、および`price`のみをロードする場合、次のコマンドを実行します。

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
> 上記の例では、JSONデータの最外層は角かっこ`[]`で囲まれた配列構造です。配列構造は、各データレコードを表す複数のJSONオブジェクトで構成されています。そのため、`strip_outer_array`を`true`に設定して最外層の配列構造を削除する必要があります。ロードしない`title`キーはロード中に無視されます。

##### ルート要素が指定されたマッチングモードでJSONデータをロードする

データファイル`example3.json`が次のデータを含んでいるとします。

```JSON
{"id": 10001,"RECORDS":[{"category":"11","title":"SayingsoftheCentury","price":895,"timestamp":1589191587},{"category":"22","author":"2avc","price":895,"timestamp":1589191487},{"category":"33","author":"3avc","title":"SayingsoftheCentury","timestamp":1589191387}],"comments": ["3 records", "there will be 3 rows"]}
```

`example3.json`から`category`、`author`、および`price`のみをロードする場合、次のコマンドを実行します。

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
> 上記の例では、JSONデータの最外層は角かっこ`[]`で囲まれた配列構造です。配列構造は、各データレコードを表す複数のJSONオブジェクトで構成されています。そのため、`strip_outer_array`を`true`に設定して最外層の配列構造を削除する必要があります。ロードしない`title`および`timestamp`キーはロード中に無視されます。また、`json_root`パラメータを使用してJSONデータのルート要素（配列）を指定するために使用されます。
