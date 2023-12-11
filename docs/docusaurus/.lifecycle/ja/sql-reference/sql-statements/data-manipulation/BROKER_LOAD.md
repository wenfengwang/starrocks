```yaml
displayed_sidebar: "Japanese"
```

# ブローカーロード

import InsertPrivNote from '../../../assets/commonMarkdown/insertPrivNote.md'

## 説明

StarRocksは、MySQLベースのローディングメソッドであるブローカーロードを提供しています。ロードジョブを提出すると、StarRocksは非同期でジョブを実行します。`SELECT * FROM information_schema.loads`を使用してジョブの結果をクエリできます。この機能はv3.1以降でサポートされています。背景情報、原則、サポートされるデータファイル形式、シングルテーブルロードとマルチテーブルロードの実行方法、およびジョブの結果の表示方法については、[HDFSからのデータロード](../../../loading/hdfs_load.md)および[クラウドストレージからのデータロード](../../../loading/cloud_storage_load.md)を参照してください。

<InsertPrivNote />

## 構文

```SQL
LOAD LABEL [<database_name>.]<label_name>
(
    data_desc[, data_desc ...]
)
WITH BROKER
(
    StorageCredentialParams
)
[PROPERTIES
(
    opt_properties
)
]
```

StarRocksでは、一部のリテラルがSQL言語によって予約されたキーワードとして使用されています。これらのキーワードを直接SQLステートメントで使用しないでください。SQLステートメントでそのようなキーワードを使用する場合は、バッククォート（`）で囲んでください。[キーワード](../../../sql-reference/sql-statements/keywords.md)を参照してください。

## パラメータ

### database_nameおよびlabel_name

`label_name`は、ロードジョブのラベルを指定します。

`database_name`は、任意で、対象テーブルが属するデータベースの名前を指定します。

各ロードジョブには、データベース全体で一意なラベルがあります。ロードジョブのラベルを使用して、ロードジョブの実行状態を表示したり、同じデータを繰り返しロードするのを防いだりできます。ロードジョブが**FINISHED**状態になると、そのラベルは再利用できなくなります。**CANCELLED**状態になったロードジョブのラベルだけが再利用できます。ほとんどのケースで、ロードジョブのラベルは、そのロードジョブを再試行して同じデータをロードするために再利用され、したがって厳密に一度だけのセマンティクスを実装します。

ラベルの名前付けの規則については、[システム制限](../../../reference/System_limit.md)を参照してください。

### data_desc

ロードするデータのバッチの説明です。各`data_desc`記述子は、データソース、ETL関数、宛先StarRocksテーブル、および宛先パーティションなどの情報を宣言します。

ブローカーロードは、同時に複数のデータファイルをロードすることをサポートしています。1つのロードジョブで、複数の`data_desc`記述子を使用してロードするデータファイルを複数宣言したり、1つの`data_desc`記述子を使用して特定のファイルパスからすべてのデータファイルをロードしたい場合に使用したりできます。ブローカーロードは、複数のデータファイルをロードする各ロードジョブのトランザクション的な原子性も確保できます。原子性とは、1つのロードジョブで複数のデータファイルをロードすることがすべて成功するか、すべて失敗するかです。あるデータファイルのロードが成功し、他のファイルのロードが失敗することはありません。

`data_desc`は、次の構文をサポートしています。

```SQL
DATA INFILE ("<file_path>"[, "<file_path>" ...])
[NEGATIVE]
INTO TABLE <table_name>
[PARTITION (<partition1_name>[, <partition2_name> ...])]
[TEMPORARY PARTITION (<temporary_partition1_name>[, <temporary_partition2_name> ...])]
[COLUMNS TERMINATED BY "<column_separator>"]
[ROWS TERMINATED BY "<row_separator>"]
[FORMAT AS "CSV | Parquet | ORC"]
[(fomat_type_options)]
[(column_list)]
[COLUMNS FROM PATH AS (<partition_field_name>[, <partition_field_name> ...])]
[SET <k1=f1(v1)>[, <k2=f2(v2)> ...]]
[WHERE predicate]
```

`data_desc`には、以下のパラメータが含まれている必要があります。

- `file_path`

   ロードする1つまたは複数のデータファイルの保存パスを指定します。

   このパラメータは、1つのデータファイルの保存パスとして指定することができます。たとえば、このパラメータを`"hdfs://<hdfs_host>:<hdfs_port>/user/data/tablename/20210411"`と指定して、HDFSサーバーの`/user/data/tablename`パスから`20210411`という名前のデータファイルをロードできます。

   `?`、`*`、`[]`、`{}`、`^`を使用して複数のデータファイルの保存パスを指定することもできます。[ワイルドカードリファレンス](https://hadoop.apache.org/docs/stable/api/org/apache/hadoop/fs/FileSystem.html#globStatus-org.apache.hadoop.fs.Path-)を参照してください。たとえば、このパラメータを`"hdfs://<hdfs_host>:<hdfs_port>/user/data/tablename/*/*"`または`"hdfs://<hdfs_host>:<hdfs_port>/user/data/tablename/dt=202104*/*"`と指定して、HDFSサーバーの`/user/data/tablename`パスからすべてのパーティションまたは`202104`パーティションのデータファイルをロードできます。

   > **注意**
   >
   > ワイルドカードを使用して中間パスを指定することもできます。

   この例では、`hdfs_host`および`hdfs_port`パラメータは以下のように記述されています。

   - `hdfs_host`: HDFSクラスタのNameNodeホストのIPアドレス。

   - `hdfs_host`: HDFSクラスタのNameNodeホストのFSポート。デフォルトのポート番号は`9000`です。

   > **注意**
   >
   > - ブローカーロードはS3またはS3Aプロトコルに従ってAWS S3にアクセスすることをサポートしています。したがって、AWS S3からデータをロードする場合は、ファイルパスとして渡すS3 URIに`s3://`または`s3a://`を含めることができます。
   > - ブローカーロードは、gsプロトコルに従ってGoogle GCSにのみアクセスすることをサポートします。したがって、Google GCSからデータをロードする場合は、ファイルパスとして渡すGCS URIに`gs://`を含める必要があります。
   > - Blob Storageからデータをロードする場合は、wasbまたはwasbsプロトコルを使用してデータへのアクセスが許可されます。:
   >   - ストレージアカウントがHTTP経由でのアクセスを許可する場合は、wasbプロトコルを使用してファイルパスを`wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`と書きます。
   >   - ストレージアカウントがHTTPS経由でのアクセスを許可する場合は、wasbsプロトコルを使用してファイルパスを`wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`と書きます。
   > - Data Lake Storage Gen1からデータをロードする場合は、adlプロトコルを使用してデータへのアクセスが許可され、ファイルパスを`adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`と書きます。
   > - Data Lake Storage Gen2からデータをロードする場合は、abfsまたはabfssプロトコルを使用してデータへのアクセスが許可されます。:
   >   - ストレージアカウントがHTTP経由でのアクセスを許可する場合は、abfsプロトコルを使用してファイルパスを`abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>`と書きます。
   >   - ストレージアカウントがHTTPS経由でのアクセスを許可する場合は、abfssプロトコルを使用してファイルパスを`abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>`と書きます。

- `INTO TABLE`

   宛先のStarRocksテーブルの名前を指定します。

`data_desc`には、以下のパラメータも含めることができます。

- `NEGATIVE`

   特定のデータバッチのロードを取り消します。これを実現するには、`NEGATIVE`キーワードを指定して同じデータバッチをロードする必要があります。

   > **注意**
   >
   > このパラメータは、StarRocksテーブルが集計テーブルであり、そのすべての値列が`sum`関数によって計算されている場合にのみ有効です。

- `PARTITION`

   ロードするデータを指定するパーティションを指定します。デフォルトでは、このパラメータを指定しない場合、ソースデータはStarRocksテーブルのすべてのパーティションにロードされます。

- `TEMPORARY PARTITION`

   ロードするデータを指定する[一時パーティション](../../../table_design/Temporary_partition.md)の名前を指定します。複数の一時パーティションを指定することができますが、それらはコンマ（,）で区切られている必要があります。

- `COLUMNS TERMINATED BY`

   データファイルで使用される列区切り記号を指定します。デフォルトでは、このパラメータを指定しない場合、このパラメータは`\t`（タブ）を表す`\t`にデフォルトでなります。このパラメータを使用して指定する列区切り記号は、実際にデータファイルで使用されている列区切り記号と同じである必要があります。それ以外の場合は、データ品質が不十分であり、その`State`が`CANCELLED`になるため、ロードジョブが失敗します。

   ブローカーロードジョブは、MySQLプロトコルに従って提出されます。StarRocksとMySQLは、両方ともロードリクエストでエスケープキャラクタを使用します。そのため、列区切り記号がタブなどの不可視文字の場合は、列区切り記号の前にバックスラッシュ（\）を追加する必要があります。たとえば、列区切り記号が`\t`の場合は`\\t`を入力する必要があり、列区切り記号が`\n`の場合は`\\n`を入力する必要があります。Apache Hive™ファイルでは、列区切り記号として`\x01`が使用されるため、データファイルがHiveからの場合は`\\x01`を入力する必要があります。

   > **注意**
   >
   > - CSVデータの場合、50バイトを超えないUTF-8文字列（例：コンマ（,）、タブ、パイプ（|）など）をテキストデリミタとして使用できます。
```
```markdown
  > - `NULL`の値は`\N`を使用して表されます。たとえば、データファイルが3つの列から成り、そのデータファイルのレコードが最初の列と三番目の列にデータを保持しているが、二番目の列にデータがない場合、この状況では、二番目の列に`NULL`の値を示すために`\N`を使用する必要があります。これは、レコードを`a,\N,b`のようにまとめる必要があることを意味し、`a,,b`の代わりに使うべきではありません。`a,,b` は、レコードの二番目の列が空の文字列を保持していることを表します。

- `ROWS TERMINATED BY`

  データファイルで使用される行のセパレータを指定します。このパラメータを指定しない場合、デフォルトで`\n`に設定され、これは改行を示します。このパラメータで指定する行セパレータは、実際にデータファイルで使用されているものと同じでなければなりません。そうでない場合、データ品質が不十分であるためロードジョブが失敗し、そのステートは`CANCELLED`となります。このパラメータはv2.5.4以降でサポートされています。

  行セパレータに関する使用上の注意については、前述の`COLUMNS TERMINATED BY`パラメータの使用上の注意を参照してください。

- `FORMAT AS`

  データファイルのフォーマットを指定します。有効な値: `CSV`、`Parquet`、`ORC`。このパラメータを指定しない場合、StarRocksは`file_path`パラメータで指定されたファイル名の拡張子 **.csv**、**.parquet**、または **.orc** に基づいてデータファイルのフォーマットを決定します。

- `format_type_options`

  `FORMAT AS`が`CSV`に設定されている場合のCSVフォーマットオプションを指定します。構文:

  ```JSON
  (
      key = value
      key = value
      ...
  )
  ```

  > **注意**
  >
  > `format_type_options` はv3.0以降でサポートされています。

  次の表はオプションを説明します。

  | **パラメータ** | **説明**                                              |
  | ------------- | ------------------------------------------------------------ |
  | skip_header   | CSVフォーマットのデータファイルの最初の行をスキップするかどうかを指定します。タイプ: INTEGER。デフォルト値: `0`。<br />いくつかのCSV形式のデータファイルでは、最初の行は列名や列のデータ型などのメタデータを定義するために使用されます。 `skip_header`パラメータを設定することで、データのロード中にStarRocksがデータファイルの最初の行をスキップすることができます。たとえば、このパラメータを`1`に設定すると、StarRocks はデータのロード中にデータファイルの最初の行をスキップします。<br />データファイルの最初の行は、ロードステートメントで指定した行セパレータで区切られている必要があります |
  | trim_space    | CSVフォーマットのデータファイルに含まれる列区切り記号の前後のスペースを削除するかどうかを指定します。タイプ: BOOLEAN。デフォルト値: `false`。<br />一部のデータベースでは、CSVフォーマットのデータファイルとしてデータをエクスポートする際に列区切り記号にスペースが追加されます。このような余分なスペースは、場所によっては前置スペースまたは後置スペースと呼ばれます。`trim_space`パラメータを設定することで、StarRocksがデータのロード中にこのような不要なスペースを削除できます。<br />なお、StarRocksは`enclose`で指定された文字で囲まれたフィールド内のスペース（前置スペースおよび後置スペースを含む）は削除しません。たとえば、以下のフィールド値は、パイプ（<code class="language-text">&#124;</code>）を列区切り記号として、二重引用符（`"`）を`enclose`で指定された文字として使用しています:<br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> <br /><code class="language-text">&#124;" Love StarRocks "&#124;</code> <br /><code class="language-text">&#124; "Love StarRocks" &#124;</code> <br />`trim_space`を`true`に設定すると、StarRocksは前置のフィールド値を次のように処理します:<br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> <br /><code class="language-text">&#124;" Love StarRocks "&#124;</code> <br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> |
  | enclose       | データファイルがCSV形式の場合、[RFC4180](https://www.rfc-editor.org/rfc/rfc4180)に従ってフィールド値を囲むのに使用される文字を指定します。タイプ: single-byte character。デフォルト値: `NONE`。最も一般的な文字は単一引用符（`'`）と二重引用符（`"`）です。<br />`enclose`で指定された文字で囲まれたすべての特殊文字（行セパレータや列セパレータを含む）は通常の記号と見なされます。StarRocksはRFC4180以上のことができるため、`enclose`で指定された文字を任意の単一バイト文字として指定できます。<br />フィールド値に`enclose`で指定された文字が含まれる場合、その `enclose` で指定された文字をエスケープすることができます。たとえば、 `enclose` を `"` に設定し、フィールド値が `a "quoted" c` の場合、データファイルには `"a ""quoted"" c"` と入力します。 |
  | escape        | 行セパレータ、列セパレータ、エスケープ文字、`enclose`で指定された文字などの様々な特殊文字をエスケープするために使用される文字を指定します。タイプ: single-byte character。デフォルト値: `NONE`。最も一般的な文字はスラッシュ(`\`)であり、SQLステートメントでは二重スラッシュ(`\\`)として記述する必要があります。<br />**注意**<br />`escape`で指定された文字は、`enclose`で指定された文字の組の内外の両方に適用されます。<br />2つの例は以下の通りです:<ul><li>`enclose`を`"`に、`escape`を`\`に設定した場合、StarRocks は `"say \"Hello world\""` を `say "Hello world"` に解析します。</li><li>列セパレータがカンマ(`,`)であるとします。`escape`を`\`に設定すると、StarRocks は `a, b\, c` を独立した2つのフィールド値、`a` および `b, c` に解析します。</li></ul> |

- `column_list`

  データファイルとStarRocksテーブルの列のマッピングを指定します。構文: `(<column_name>[, <column_name> ...])`。`column_list`で宣言された列は、名前によってStarRocksテーブルの列にマッピングされます。

  > **注意**
  >
  > データファイルの列がStarRocksテーブルの列にシーケンスに従ってマッピングされる場合、`column_list`を指定する必要はありません。

  データファイルの特定の列をスキップしたい場合、その列を一時的にStarRocksテーブルの列とは異なる名前で命名するだけです。詳細については、[Transform data at loading](../../../loading/Etl_in_loading.md)を参照してください。

- `COLUMNS FROM PATH AS`

  指定したファイルパスから1つ以上のパーティションフィールドの情報を抽出します。このパラメータは、ファイルパスにパーティションフィールドが含まれている場合にのみ有効です。

  たとえば、データファイルがパス `/path/col_name=col_value/file1` に格納されている場合、ここで `col_name` がパーティションフィールドであり、StarRocksテーブルの列にマッピングできる場合、このパラメータを `col_name` として指定します。そのようにして、StarRocksはパスから `col_name` にマッピングされるStarRocksテーブルの列に `col_value` の値を抽出してロードします。

  > **注意**
  >
  > このパラメータはHDFSからデータをロードする場合のみ利用可能です。

- `SET`

  データファイルの列を変換するために使用する関数を1つ以上指定します。例:

  - StarRocksテーブルは、`col1`、`col2`、`col3` の3つの列から成ります。データファイルは4つの列から成り、そのうち最初の2つの列が順にStarRocksテーブルの `col1` および `col2` にマッピングされ、最後の2つの列の合計がStarRocksテーブル `col3` にマッピングされます。この場合、`column_list`を `(col1,col2,tmp_col3,tmp_col4)` として指定し、SET句に `(col3=tmp_col3+tmp_col4)` を指定してデータ変換を実装する必要があります。
  - StarRocksテーブルは、`year`、`month`、`day` の3つの列から成ります。データファイルは、`yyyy-mm-dd hh:mm:ss`形式の日付と時刻値を含む列のみで構成されています。この場合、`column_list`を `(tmp_time)` として指定し、SET句に `(year = year(tmp_time), month=month(tmp_time), day=day(tmp_time))` を指定してデータ変換を実装する必要があります。

- `WHERE`

  ソースデータをフィルタリングする条件を指定します。WHERE句で指定したフィルタ条件に合致するソースデータのみがStarRocksにロードされます。

### WITH BROKER

v2.3およびそれ以前では、使用したいブローカーを指定するには `WITH BROKER "<broker_name>"` と入力します。v2.5以降、ブローカーを指定する必要はなくなりましたが、`WITH BROKER` キーワード自体は引き続き保持する必要があります。

> **注意**
>
> v2.4およびそれ以前のバージョンでは、StarRocksはブローカーに依存しており、ブローカーのロードジョブを実行する際に、StarRocksクラスタと外部ストレージシステムの接続を確立します。これを「ブローカーベースのロード」と呼びます。ブローカーはファイルシステムインターフェースに統合された独立した状態を持つサービスであり、StarRocksはこれにより、外部ストレージシステムに保存されているデータファイルにアクセスし、それらのデータファイルのデータを事前処理してロードすることができます。

> v2.5以降、StarRocksはブローカーに対する依存関係を取り除き、「ブローカーフリーのロード」を実装しています。

> [SHOW BROKER](../Administration/SHOW_BROKER.md)ステートメントを使用して、StarRocksクラスタに展開されているブローカーを確認できます。ブローカーが展開されていない場合は、[ブローカーを展開](../../../deployment/deploy_broker.md)する手順に従ってブローカーを展開できます。

### StorageCredentialParams

StarRocksがストレージシステムにアクセスするために使用する認証情報。

#### HDFS

オープンソースのHDFSは、シンプル認証とKerberos認証の2つの認証方法をサポートしています。ブローカーロードでは、デフォルトでシンプル認証が使用されます。また、オープンソースのHDFSは、NameNodeのためのHAメカニズムの構成もサポートしています。ストレージシステムとしてオープンソースのHDFSを選択した場合、認証構成とHA構成を次のように指定できます:

- 認証構成

  - シンプル認証を使用する場合、「StorageCredentialParams」を次のように構成します:

    ```Plain
    "hadoop.security.authentication" = "simple",
    "username" = "<hdfsユーザー名>",
    "password" = "<hdfsパスワード>"
    ```

    次の表に、`StorageCredentialParams`のパラメータについて説明します。

    | パラメータ                      | 説明                                               |
    | ---------------------------- | ------------------------------------------------- |
    | hadoop.security.authentication | 認証方法。有効値: `simple`および`kerberos`。デフォルト値: `simple`。`simple`はシンプル認証（つまり認証なし）を表し、`kerberos`はKerberos認証を表します。 |
    | username                     | HDFSクラスタのNameNodeにアクセスするために使用するアカウントのユーザー名。 |
    | password                     | HDFSクラスタのNameNodeにアクセスするために使用するアカウントのパスワード。 |

  - Kerberos認証を使用する場合、「StorageCredentialParams」を次のように構成します:

    ```Plain
    "hadoop.security.authentication" = "kerberos",
    "kerberos_principal" = "nn/zelda1@ZELDA.COM",
    "kerberos_keytab" = "/keytab/hive.keytab",
    "kerberos_keytab_content" = "YWFhYWFh"
    ```

    次の表に、`StorageCredentialParams`のパラメータについて説明します。

    | パラメータ                      | 説明                                               |
    | ---------------------------- | ------------------------------------------------- |
    | hadoop.security.authentication | 認証方法。有効値: `simple`および`kerberos`。デフォルト値: `simple`。`simple`はシンプル認証（つまり認証なし）を表し、`kerberos`はKerberos認証を表します。 |
    | kerberos_principal           | 認証されるKerberosプリンシパル。各プリンシパルは、HDFSクラスタ全体で一意であることを確実にするため、以下の3つの部分で構成されています:<ul><li>`username`または`servicename`: プリンシパルの名前。</li><li>`instance`: HDFSクラスタ内で認証されるノードをホストするサーバーの名前。サーバー名は、各自が独立して認証される複数のDataNodeを持つHDFSクラスタの例えば一意であるように助けます。</li><li>`realm`: レルムの名前。レルム名は大文字である必要があります。 例: `nn/[zelda1@ZELDA.COM](mailto:zelda1@ZELDA.COM)`。</li></ul> |
    | kerberos_keytab              | Kerberosキータブファイルの保存パス。 |
    | kerberos_keytab_content      | KerberosキータブファイルのBase64エンコードされた内容。`kerberos_keytab`または`kerberos_keytab_content`のいずれかを指定できます。 |

    複数のKerberosユーザを構成した場合、少なくとも1つの独立した[ブローカーグループ](../../../deployment/deploy_broker.md)が展開されるようにしてください。ロードステートメントでは、使用するブローカーグループを指定するために `WITH BROKER "<broker_name>"` を入力する必要があります。また、ブローカーの起動スクリプトファイル **start_broker.sh** を開き、ファイルの42行目を変更してブローカーが **krb5.conf** ファイルを読み取るようにします。例:

    ```Plain
    export JAVA_OPTS="-Dlog4j2.formatMsgNoLookups=true -Xmx1024m -Dfile.encoding=UTF-8 -Djava.security.krb5.conf=/etc/krb5.conf"
    ```

    > **注記**
    
    > - 上記の例では`/etc/krb5.conf`を実際の **krb5.conf** ファイルの保存パスに置き換えることができます。ブローカーがそのファイルを読み取る権限を持つことを確認してください。もしブローカーグループが複数のブローカーで構成されている場合は、各ブローカーノードで **start_broker.sh** ファイルを変更してからブローカーノードを再起動して変更を有効にしてください。
    > - [SHOW BROKER](../Administration/SHOW_BROKER.md)ステートメントを使用して、StarRocksクラスタに展開されているブローカーを確認できます。

- HA構成

  HDFSクラスタのNameNodeにHAメカニズムを構成できます。これにより、NameNodeが別のノードに切り替わった場合でも、StarRocksは新しいNameNodeとして機能するノードを自動的に特定できます。次のシナリオがこれに含まれます。

  - 1つのKerberosユーザが構成された単一のHDFSクラスタからデータをロードする場合、ロードベースのロードとロードフリーのロードの両方をサポートしています。

    - ロードベースのロードを行う場合は、少なくとも1つの独立した[ブローカーグループ](../../../deployment/deploy_broker.md)が展開されるようにしてください。HDFSクラスタを提供するブローカーノードの`{deploy}/conf`パスに`hdfs-site.xml`ファイルを配置します。StarRocksはブローカー起動時に`{deploy}/conf`パスを環境変数`CLASSPATH`に追加し、ブローカーがHDFSクラスタノードの情報を読むことを可能にします。

    - ロードフリーのロードを行う場合は、`hdfs-site.xml`ファイルを各FEノードおよび各BEノードの`{deploy}/conf`パスに配置します。

  - 複数のKerberosユーザが構成された単一のHDFSクラスタからデータをロードする場合、ブローカーベースのロードのみがサポートされます。少なくとも1つの独立した[ブローカーグループ](../../../deployment/deploy_broker.md)が展開されるようにしてください。HDFSクラスタを提供するブローカーノードの`{deploy}/conf`パスに`hdfs-site.xml`ファイルを配置します。StarRocksはブローカー起動時に`{deploy}/conf`パスを環境変数`CLASSPATH`に追加し、ブローカーがHDFSクラスタノードの情報を読むことを可能にします。

  - 複数のHDFSクラスタからデータをロードする場合（1つまたは複数のKerberosユーザが構成されているかどうかに関係なく）、ブローカーベースのロードのみがサポートされます。これらのHDFSクラスタごとに少なくとも1つの独立した[ブローカーグループ](../../../deployment/deploy_broker.md)が展開されていることを確認し、ブローカーがHDFSクラスタノードの情報を読むことを可能にするために次のいずれかのアクションを実行してください:

    - `hdfs-site.xml`ファイルを各HDFSクラスタを提供するブローカーノードの`{deploy}/conf`パスに配置します。StarRocksはブローカー起動時に`{deploy}/conf`パスを環境変数`CLASSPATH`に追加し、そのHDFSクラスタのノードに関する情報をブローカーが読むことを可能にします。

    - 次のHA構成をジョブ作成時に追加します:

      ```Plain
      "dfs.nameservices" = "ha_cluster",
      "dfs.ha.namenodes.ha_cluster" = "ha_n1,ha_n2",
      "dfs.namenode.rpc-address.ha_cluster.ha_n1" = "<hdfsホスト>:<hdfsポート>",
      "dfs.namenode.rpc-address.ha_cluster.ha_n2" = "<hdfsホスト>:<hdfsポート>",
      "dfs.client.failover.proxy.provider" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
      ```

      次の表に、HA構成のパラメータについて説明します。

      | パラメータ                          | 説明                                    |
      | ---------------------------------- | -------------------------------------- |
      | dfs.nameservices                   | HDFSクラスタの名前。                     |
      | dfs.ha.namenodes.XXX               | HDFSクラスタのNameNodeの名前。複数のNameNode名を指定する場合は、カンマ（`,`）で区切ります。`xxx`は`dfs.nameservices`で指定したHDFSクラスタ名です。 |
      | dfs.namenode.rpc-address.XXX.NN    | HDFSクラスタのNameNodeのRPCアドレス。`NN`は`dfs.ha.namenodes.XXX`で指定したNameNode名です。 |
      | dfs.client.failover.proxy.provider | クライアントが接続するNameNodeのプロバイダ。デフォルト値: `org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider`。 |

  > **注記**
  >
  > [SHOW BROKER](../Administration/SHOW_BROKER.md)ステートメントを使用して、StarRocksクラスタに展開されているブローカーを確認できます。

#### AWS S3

ストレージシステムとしてAWS S3を選択する場合、次のいずれかのアクションを実行できます:

- インスタンスプロファイルベースの認証メソッドを選択するには、`StorageCredentialParams`を次のように構成します：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- サービスアカウントベースの認証メソッドを選択するには、`StorageCredentialParams`を次のように構成します：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAMユーザーベースの認証メソッドを選択するには、`StorageCredentialParams`を次のように構成します：

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

以下の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。

| パラメータ                   | 必須   | 説明                                                         |
| --------------------------- | ------ | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | Yes    | クレデンシャルメソッドにインスタンスプロファイルと仮定される役割を有効にするかどうかを指定します。有効な値は「true」と「false」です。デフォルト値は「false」です。 |
| aws.s3.iam_role_arn         | No     | AWS S3バケットに権限を持つIAMロールのARN。AWS S3へのアクセスの認証メソッドとして仮定される役割を選択した場合、このパラメータを指定する必要があります。 |
| aws.s3.region               | Yes    | AWS S3バケットが存在するリージョン。例：`us-west-1`。        |
| aws.s3.access_key           | No     | IAMユーザーのアクセスキー。AWS S3へのアクセスの認証メソッドとしてIAMユーザーを選択した場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | No     | IAMユーザーのシークレットキー。AWS S3へのアクセスの認証メソッドとしてIAMユーザーを選択した場合、このパラメータを指定する必要があります。 |

AWS S3へのアクセスの認証メソッドを選択する方法とAWS IAMコンソールでアクセス制御ポリシーを構成する方法については、[AWS S3へのアクセスの認証パラメータ](../../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

#### Google GCS

ストレージシステムとしてGoogle GCSを選択する場合、次のいずれかの操作を実行してください：

- VMベースの認証メソッドを選択するには、`StorageCredentialParams`を次のように構成します：

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  以下の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。

  | **パラメータ**                             | **デフォルト値** | **値の例** | **説明**                                                     |
  | ---------------------------------------- | ---------------- | ----------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | false            | true        | Compute Engineにバインドされたサービスアカウントを直接使用するかどうかを指定します。 |

- サービスアカウントベースの認証メソッドを選択するには、`StorageCredentialParams`を次のように構成します：

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  以下の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。

  | **パラメータ**                         | **デフォルト値** | **値の例**                                                    | **説明**                                                     |
  | ------------------------------------ | ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email        | ""               | "user@hello.iam.gserviceaccount.com"                        | サービスアカウント作成時に生成されたJSONファイル内の電子メールアドレス。 |
  | gcp.gcs.service_account_private_key_id | ""               | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"               | サービスアカウント作成時に生成されたJSONファイル内のプライベートキーID。 |
  | gcp.gcs.service_account_private_key  | ""               | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | サービスアカウント作成時に生成されたJSONファイル内のプライベートキー。 |

- 偽装ベースの認証メソッドを選択するには、`StorageCredentialParams`を次のように構成します：

  - VMインスタンスにサービスアカウントを偽装させる：

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    以下の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。

    | **パラメータ**                             | **デフォルト値** | **値の例** | **説明**                                                     |
    | ---------------------------------------- | ---------------- | ----------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false            | true        | Compute Engineにバインドされたサービスアカウントを直接使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""               | "hello"     | 偽装したいサービスアカウント。                                |

  - メタサービスアカウントとして名前が付けられたサービスアカウントが別のサービスアカウント（データサービスアカウントとして名前が付けられた）を偽装する：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    以下の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。

    | **パラメータ**                         | **デフォルト値** | **値の例**                                                    | **説明**                                                     |
    | ------------------------------------ | ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email        | ""               | "user@hello.iam.gserviceaccount.com"                        | メタサービスアカウント作成時に生成されたJSONファイル内の電子メールアドレス。 |
    | gcp.gcs.service_account_private_key_id | ""               | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"               | メタサービスアカウント作成時に生成されたJSONファイル内のプライベートキーID。 |
    | gcp.gcs.service_account_private_key  | ""               | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | メタサービスアカウント作成時に生成されたJSONファイル内のプライベートキー。 |
    | gcp.gcs.impersonation_service_account  | ""               | "hello"                                                       | 偽装したいデータサービスアカウント。                        |

#### その他のS3互換ストレージシステム

MinIOなどの他のS3互換ストレージシステムを選択する場合、`StorageCredentialParams`を次のように構成してください：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。

| パラメータ                        | 必須   | 説明                                                     |
| -------------------------------- | ------ | -------------------------------------------------------- |
| aws.s3.enable_ssl                | Yes    | SSL接続を有効にするかどうかを指定します。有効な値は「true」と「false」です。デフォルト値は「true」です。 |
| aws.s3.enable_path_style_access  | Yes    | パス形式のURLアクセスを有効にするかどうかを指定します。有効な値は「true」と「false」です。MinIOの場合、値を「true」に設定する必要があります。 |
| aws.s3.endpoint                  | Yes    | AWS S3の代わりにS3互換ストレージシステムに接続するために使用するエンドポイント。 |
| aws.s3.access_key                | Yes    | IAMユーザーのアクセスキー。                             |
| aws.s3.secret_key                | Yes    | IAMユーザーのシークレットキー。                         |

#### Microsoft Azure Storage

##### Azure Blob Storage

Blob Storageをストレージシステムとして選択する場合、次のいずれかの操作を実行してください：

- 共有キー認証メソッドを選択するには、`StorageCredentialParams`を次のように構成します：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  以下の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。

  | **パラメータ**              | **必須** | **説明**                         |
  | -------------------------- | -------- | -------------------------------- |
  | azure.blob.storage_account | Yes      | Blob Storageアカウントのユーザー名。 |
  | azure.blob.shared_key      | Yes      | Blob Storageアカウントの共有キー。 |

- SASトークン認証メソッドを選択するには、`StorageCredentialParams`を次のように構成します：

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
```plaintext
      "azure.blob.container_name" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"

  以下の表は、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **Parameter**             | **Required** | **Description**                                              |
  | ------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.blob.account_name   | Yes          | Blob Storageアカウントのユーザー名。                         |
  | azure.blob.container_name | Yes          | データを格納するブロブコンテナの名前。                       |
  | azure.blob.sas_token      | Yes          | Blob Storageアカウントにアクセスするために使用されるSASトークン。 |

##### Azure Data Lake Storage Gen1

ストレージシステムとしてData Lake Storage Gen1を選択する場合、次のいずれかのアクションを実行します:

- Managed Service Identity認証メソッドを選択するには、`StorageCredentialParams`を次のように構成します:

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  以下の表は、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **Parameter**                            | **Required** | **Description**                                              |
  | ---------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | Yes          | Managed Service Identity認証メソッドを有効にするかどうかを指定します。値を`true`に設定します。 |

- Service Principal認証メソッドを選択するには、`StorageCredentialParams`を次のように構成します:

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  以下の表は、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **Parameter**                 | **Required** | **Description**                                              |
  | ----------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | Yes          | クライアント（アプリケーション）ID。                         |
  | azure.adls1.oauth2_credential | Yes          | 作成された新しいクライアント（アプリケーション）シークレットの値。 |
  | azure.adls1.oauth2_endpoint   | Yes          | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）。 |

##### Azure Data Lake Storage Gen2

ストレージシステムとしてData Lake Storage Gen2を選択する場合、次のいずれかのアクションを実行します:

- Managed Identity認証メソッドを選択するには、`StorageCredentialParams`を次のように構成します:

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  以下の表は、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **Parameter**                           | **Required** | **Description**                                              |
  | --------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | Yes          | Managed Identity認証メソッドを有効にするかどうかを指定します。値を`true`に設定します。 |
  | azure.adls2.oauth2_tenant_id            | Yes          | アクセスするテナントのID。                                  |
  | azure.adls2.oauth2_client_id            | Yes          | 管理対象アイデンティティのクライアント（アプリケーション）ID。     |

- Shared Key認証メソッドを選択するには、`StorageCredentialParams`を次のように構成します:

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  以下の表は、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **Parameter**               | **Required** | **Description**                                              |
  | --------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.storage_account | Yes          | Data Lake Storage Gen2ストレージアカウントのユーザー名。       |
  | azure.adls2.shared_key      | Yes          | Data Lake Storage Gen2ストレージアカウントの共有キー。         |

- Service Principal認証メソッドを選択するには、`StorageCredentialParams`を次のように構成します:

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  以下の表は、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **Parameter**                      | **Required** | **Description**                                              |
  | ---------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | Yes          | サービスプリンシパルのクライアント（アプリケーション）ID。          |
  | azure.adls2.oauth2_client_secret   | Yes          | 作成された新しいクライアント（アプリケーション）シークレットの値。 |
  | azure.adls2.oauth2_client_endpoint | Yes          | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）。 |

### opt_properties

ロードジョブ全体に適用されるいくつかのオプションパラメータを指定します。構文:

```Plain
PROPERTIES ("<key1>" = "<value1>"[, "<key2>" = "<value2>" ...])
```

以下のパラメータがサポートされます:

- `timeout`

  ロードジョブのタイムアウト期間を指定します。単位: 秒。デフォルトのタイムアウト期間は4時間です。6時間より短いタイムアウト期間を指定することをお勧めします。ロードジョブがタイムアウト期間内に完了しない場合、StarRocksはロードジョブをキャンセルし、ロードジョブのステータスが**CANCELLED**になります。

  > **注意**
  >
  > ほとんどの場合、タイムアウト期間を設定する必要はありません。デフォルトのタイムアウト期間内にロードジョブが完了しない場合のみ、タイムアウト期間を設定することをお勧めします。

  タイムアウト期間は次の数式を使用して推定します:

  **タイムアウト期間 > (ロードするデータファイルの総サイズ x ロードするデータファイルの総数およびそれらのデータファイルで作成されたマテリアライズドビューの総数)/平均ロード速度**

  > **注意**
  >
  > 「平均ロード速度」は、クラスタ全体の平均ロード速度です。平均ロード速度は、サーバー構成やクラスタに許可される最大並列クエリタスク数に応じて、各クラスタごとに異なります。過去のロードジョブのロード速度を基に、平均ロード速度を推定できます。

  たとえば、平均ロード速度が10 MB/sのStarRocksクラスタに1GBのデータファイルをロードし、そのデータファイルに2つのマテリアライズドビューが作成されているとします。データロードにかかる時間は約102秒です。

  (1 x 1024 x 3)/10 = 307.2 (秒)

  この例では、タイムアウト期間を308秒より大きな値に設定することをお勧めします。

- `max_filter_ratio`

  ロードジョブの最大エラートレランスを指定します。最大エラートレランスは、データ品質が不十分であるためにフィルタリングされる行の最大パーセンテージです。有効な値: `0` ~ `1`。デフォルト値は`0`です。

  - このパラメータを`0`に設定すると、StarRocksはロード中に資格のない行を無視しません。したがって、元のデータに資格のない行が含まれている場合、ロードジョブは失敗します。これにより、StarRocksにロードされるデータの正確性が確保されます。

  - このパラメータを`0`より大きい値に設定すると、StarRocksはロード中に資格のない行を無視できます。したがって、元のデータに資格のない行が含まれている場合でも、ロードジョブは成功することがあります。

    > **注意**
    >
    > データ品質が不十分であるためにフィルタリングされる行には、WHERE句によってフィルタリングされる行は含まれません。

  `max_filter_ratio`を`0`に設定したためにロードジョブが失敗した場合は、[SHOW LOAD](../../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)を使用してジョブ結果を表示し、資格のない行をフィルタリングすることができるかどうかを判断します。資格のない行をフィルタリングできる場合は、`dpp.abnorm.ALL`および`dpp.norm.ALL`の戻り値に基づいて最大エラートレランスを計算し、最大エラートレランスを調整してロードジョブを再送信します。最大エラートレランスを計算するための数式は次のとおりです:

  `max_filter_ratio` = [`dpp.abnorm.ALL`/(`dpp.abnorm.ALL` + `dpp.norm.ALL`)]

  `dpp.abnorm.ALL`および`dpp.norm.ALL`の戻り値の合計は、ロードする行の総数です。

- `log_rejected_record_num`

  ログに記録できる最大の資格のないデータ行数を指定します。このパラメータはv3.1以降でサポートされています。有効な値: `0`、`-1`、および正の整数。デフォルト値は`0`です。
  
  - 値`0`は、フィルタリングされたデータ行をログに記録しません。
  - 値`-1`は、フィルタリングされたすべてのデータ行をログに記録します。
  - `n`などの正の非ゼロの整数は、フィルタリングされたデータ行を各BEで最大で`n`行までログに記録できることを指定します。

- `load_mem_limit`
```
最大のメモリ量

バイト単位で指定される、ロードジョブに提供することができるメモリの最大量を指定します。デフォルトのメモリ制限は2GBです。

- `strict_mode`

  [ストリクトモード](../../../loading/load_concept/strict_mode.md)を有効にするかどうかを指定します。有効な値は`true`と`false`です。デフォルト値は`false`です。`true`はストリクトモードを有効にし、`false`はストリクトモードを無効にします。

- `timezone`

  ロードジョブのタイムゾーンを指定します。デフォルト値は`Asia/Shanghai`です。タイムゾーンの設定はstrftime、alignment_timestamp、from_unixtimeなどの関数によって返される結果に影響を与えます。詳細は[タイムゾーンの設定](../../../administration/timezone.md)を参照してください。`timezone`パラメータで指定されたタイムゾーンは、セッションレベルのタイムゾーンです。

- `priority`

  ロードジョブの優先度を指定します。有効な値は`LOWEST`、`LOW`、`NORMAL`、`HIGH`、`HIGHEST`です。デフォルト値は`NORMAL`です。ブローカーロードは[FEパラメータ](../../../administration/Configuration.md#fe-configuration-items)`max_broker_load_job_concurrency`を提供し、あなたのStarRocksクラスタ内で同時に実行できるブローカーロードジョブの最大数を決定します。指定された時間内に提出されたブローカーロードジョブの数が最大数を超えると、余分なジョブはその優先度に基づいてスケジュールされるまで待機します。

  `QUEUEING`または`LOADING`状態の既存のロードジョブの優先度を変更するには[ALTER LOAD](../../../sql-reference/sql-statements/data-manipulation/ALTER_LOAD.md)ステートメントを使用できます。

  StarRocksではv2.5以降、ブローカーロードジョブの`priority`パラメータを設定することができます。

## カラムマッピング

データファイルのカラムがStarRocksテーブルのカラムと1対1のシーケンスでマッピングできる場合、データファイルとStarRocksテーブルの間のカラムマッピングを構成する必要はありません。

データファイルのカラムがStarRocksテーブルのカラムと1対1のシーケンスでマッピングできない場合、`columns`パラメータを使用してデータファイルとStarRocksテーブルの間のカラムマッピングを構成する必要があります。次の2つのユースケースがこれに該当します:

- **同じ数のカラムがありますが、異なるカラム順序です。** **また、データファイルのデータがStarRocksテーブルのカラムにロードされる前に関数によって計算される必要がありません。**

  `columns`パラメータでは、データファイルのカラムが配置されている順序と同じようにStarRocksテーブルのカラムの名前を指定する必要があります。

  例えば、StarRocksテーブルは`col1`、`col2`、`col3`の3つのカラムで構成されており、データファイルも3つのカラムで構成されており、これらはStarRocksテーブルのカラム`col3`、`col2`、`col1`と対応付けることができます。この場合、`"columns: col3, col2, col1"`を指定する必要があります。

- **異なる数のカラムと異なるカラム順序です。また、データファイルのデータがロードされる前に関数によって計算される必要があります。**

  `columns`パラメータでは、データファイルのカラムが配置されている順序と同じようにStarRocksテーブルのカラムの名前を指定し、データを計算するために使用する関数を指定する必要があります。次の2つの例があります:

  - StarRocksテーブルは`col1`、`col2`、`col3`の3つのカラムで構成されており、データファイルは4つのカラムで構成されており、そのうち最初の3つのカラムがStarRocksテーブルのカラム`col1`、`col2`、`col3`にそれぞれ対応付けられ、4番目のカラムはどのStarRocksテーブルのカラムにも対応付けられていません。この場合、データファイルの4列目に一時的な名前を指定する必要があり、その一時的な名前はStarRocksテーブルのカラム名とは異なる必要があります。例えば、`"columns: col1, col2, col3, temp"`を指定することがあります。ここでデータファイルの4番目の列は一時的に`temp`という名前になります。
  - StarRocksテーブルは`year`、`month`、`day`の3つのカラムで構成されており、データファイルは`yyyy-mm-dd hh:mm:ss`形式の日付と時刻値を収容するただ1つのカラムで構成されています。この場合、`"columns: col, year = year(col), month=month(col), day=day(col)"`を指定することができます。ここで`col`はデータファイルの列の一時的な名前であり、`year = year(col)`、`month=month(col)`、`day=day(col)`という関数は、データファイルの列`col`からデータを抽出し、それをStarRocksテーブルのカラムにロードするために使用されます。例えば、`year = year(col)`はデータファイルの列`col`から`yyyy`データを抽出し、それをStarRocksテーブルの列`year`にロードするために使用されます。

詳細な例については、[カラムマッピングの構成](#configure-column-mapping)を参照してください。

## 関連する設定項目

[FE設定項目](../../../administration/Configuration.md#fe-configuration-items)`max_broker_load_job_concurrency`は、StarRocksクラスタ内で同時に実行できるブローカーロードジョブの最大数を指定します。

StarRocks v2.4以前では、特定の時間内に提出されたブローカーロードジョブの総数が最大数を超えると、余分なジョブはその提出時間に基づいてキューイングされてスケジュールされます。

StarRocks v2.5以降、特定の時間内に提出されたブローカーロードジョブの総数が最大数を超えると、余分なジョブはその優先度に基づいてキューイングおよびスケジュールされます。上記で説明した`priority`パラメータを使用してジョブの優先度を指定することができます。**QUEUEING**または**LOADING**状態のジョブの優先度を変更するには[ALTER LOAD](../data-manipulation/ALTER_LOAD.md)を使用できます。

## 例

このセクションではHDFSを使用して、さまざまなロード構成を説明します。

### CSVデータのロード

このセクションでは、CSVを使用して多様なロード要件に対処するために使用できる様々なパラメータ構成を説明します。

#### タイムアウト期間の設定

あなたのStarRocksデータベース`test_db`には`table1`という名前のテーブルがあります。テーブルはシーケンスで`col1`、`col2`、`col3`の3つのカラムで構成されています。

あなたのデータファイル`example1.csv`も同様に、`col1`、`col2`、`col3`をシーケンスにマッピングされた3つのカラムで構成されています。

`example1.csv`からすべてのデータを3600秒以内に`table1`にロードしたい場合、次のコマンドを実行します:

```SQL
LOAD LABEL test_db.label1
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example1.csv")
    INTO TABLE table1
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 最大エラートレランスの設定

あなたのStarRocksデータベース`test_db`には`table2`という名前のテーブルがあります。テーブルはシーケンスで`col1`、`col2`、`col3`の3つのカラムで構成されています。

あなたのデータファイル`example2.csv`も同様に、`col1`、`col2`、`col3`をシーケンスにマッピングされた3つのカラムで構成されています。

`example2.csv`からすべてのデータを`table2`に最大エラートレランス`0.1`としてロードしたい場合、次のコマンドを実行します:

```SQL
LOAD LABEL test_db.label2
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example2.csv")
    INTO TABLE table2

)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
)
PROPERTIES
(
    "max_filter_ratio" = "0.1"
);
```

#### ファイルパスからすべてのデータファイルをロードする

あなたのStarRocksデータベース`test_db`には`table3`という名前のテーブルがあります。テーブルはシーケンスで`col1`、`col2`、`col3`の3つのカラムで構成されています。

あなたのHDFSクラスタの`/user/starrocks/data/input/`パスに保存されているすべてのデータファイルも、それぞれ`col1`、`col2`、`col3`にマッピングされた3つのカラムで構成されています。これらのデータファイルで使用される列の区切り記号は`\x01`です。

これらのデータファイルに含まれるデータをすべて`table3`にロードしたい場合、次のコマンドを実行します:

```SQL
LOAD LABEL test_db.label3
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/*")
    INTO TABLE table3
    COLUMNS TERMINATED BY "\\x01"
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

#### NameNode HAメカニズムの設定
あなたのStarRocksデータベース`test_db`には、`table4`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

また、データファイル`example4.csv`も3つの列で構成されており、`table4`の`col1`、`col2`、`col3`にそれぞれマッピングされています。

HAメカニズムがNameNodeに構成されている状態で、`example4.csv`のすべてのデータを`table4`にロードしたい場合は、次のコマンドを実行してください。

```SQL
LOAD LABEL test_db.label4
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example4.csv")
    INTO TABLE table4
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>",
    "dfs.nameservices" = "my_ha",
    "dfs.ha.namenodes.my_ha" = "my_namenode1, my_namenode2",
    "dfs.namenode.rpc-address.my_ha.my_namenode1" = "nn1_host:rpc_port",
    "dfs.namenode.rpc-address.my_ha.my_namenode2" = "nn2_host:rpc_port",
    "dfs.client.failover.proxy.provider" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
);
```

#### ケルベロス認証の設定

あなたのStarRocksデータベース`test_db`には、`table5`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

また、データファイル`example5.csv`も3つの列で構成されており、順に`table5`の`col1`、`col2`、`col3`にマッピングされています。

キーtabファイルパスが指定された状態でKerberos認証が構成されている`table5`に`example5.csv`のすべてのデータをロードしたい場合は、次のコマンドを実行してください。

```SQL
LOAD LABEL test_db.label5
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example5.csv")
    INTO TABLE table5
    COLUMNS TERMINATED BY "\t"
)
WITH BROKER
(
    "hadoop.security.authentication" = "kerberos",
    "kerberos_principal" = "starrocks@YOUR.COM",
    "kerberos_keytab" = "/home/starRocks/starRocks.keytab"
);
```

#### データの取り消し

あなたのStarRocksデータベース`test_db`には、`table6`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

また、データファイル`example6.csv`も3つの列で構成されており、順に`table6`の`col1`、`col2`、`col3`にマッピングされています。

Broker Loadジョブを実行して`example6.csv`のすべてのデータを`table6`にロードした場合は、次のコマンドを実行してください。

```SQL
LOAD LABEL test_db.label6
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example6.csv")
    NEGATIVE
    INTO TABLE table6
    COLUMNS TERMINATED BY "\t"
)
WITH BROKER
(
    "hadoop.security.authentication" = "kerberos",
    "kerberos_principal" = "starrocks@YOUR.COM",
    "kerberos_keytab" = "/home/starRocks/starRocks.keytab"
);
```

#### 宛先パーティションの指定

あなたのStarRocksデータベース`test_db`には、`table7`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

また、データファイル`example7.csv`も3つの列で構成されており、順に`table7`の`col1`、`col2`、`col3`にマッピングされています。

`example7.csv`のすべてのデータを`table7`の`p1`と`p2`の2つのパーティションにロードしたい場合は、次のコマンドを実行してください。

```SQL
LOAD LABEL test_db.label7
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example7.csv")
    INTO TABLE table7
    PARTITION (p1, p2)
    COLUMNS TERMINATED BY ","
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

#### 列マッピングの設定

あなたのStarRocksデータベース`test_db`には、`table8`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

また、データファイル`example8.csv`も3つの列で構成されており、順に`table8`の`col2`、`col1`、`col3`にマッピングされています。

`example8.csv`のすべてのデータを`table8`にロードしたい場合は、次のコマンドを実行してください。

```SQL
LOAD LABEL test_db.label8
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example8.csv")
    INTO TABLE table8
    COLUMNS TERMINATED BY ","
    (col2, col1, col3)
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

> **注意**
>
> 前述の例の場合、`example8.csv`の列は`table8`の列と同じ順序でマッピングすることができません。したがって、`column_list`を使用して、`example8.csv`と`table8`の間の列マッピングを構成する必要があります。

#### フィルタ条件の設定

あなたのStarRocksデータベース`test_db`には、`table9`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

また、データファイル`example9.csv`も3つの列で構成されており、順に`table9`の`col1`、`col2`、`col3`にマッピングされています。

`example9.csv`から`col1`の値が`20180601`より大きいデータレコードのみを`table9`にロードしたい場合は、次のコマンドを実行してください。

```SQL
LOAD LABEL test_db.label9
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example9.csv")
    INTO TABLE table9
    (col1, col2, col3)
    where col1 > 20180601
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

> **注意**
>
> 前述の例の場合、`example9.csv`の列は`table9`の列と同じ順序でマッピングすることができますが、列ベースのフィルタ条件を指定するためにWHERE句を使用する必要があります。したがって、`column_list`を使用して、`example9.csv`と`table9`の間の列マッピングを構成する必要があります。

#### HLLタイプの列を含むテーブルにデータをロード

あなたのStarRocksデータベース`test_db`には、`table10`という名前のテーブルが含まれています。このテーブルは、`id`、`col1`、`col2`、`col3`の4つの列で構成されています。`col1`と`col2`はHLLタイプの列として定義されています。

また、データファイル`example10.csv`は3つの列で構成されており、最初の列が`table10`の`id`に、2番目と3番目の列が`table10`の`col1`と`col2`に順にマッピングされています。`example10.csv`の2番目と3番目の列の値は、`table10`の`col1`と`col2`にロードされる前に関数を使用してHLLタイプのデータに変換することができます。

`example10.csv`のすべてのデータを`table10`にロードしたい場合は、次のコマンドを実行してください。

```SQL
LOAD LABEL test_db.label10
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example10.csv")
    INTO TABLE table10
    COLUMNS TERMINATED BY ","
    (id, temp1, temp2)
    SET
    (
        col1 = hll_hash(temp1),
        col2 = hll_hash(temp2),
        col3 = empty_hll()
     )
 )
 WITH BROKER
 (
     "username" = "<hdfs_username>",
     "password" = "<hdfs_password>"
 );
```

> **注意**
>
> 前述の例の場合、`example10.csv`の3つの列は`column_list`を使用してそれぞれ`id`、`temp1`、`temp2`と名前が付けられています。その後、次のように関数を使用してデータを変換しています。
>
> - `hll_hash`関数を使用して`example10.csv`の`temp1`と`temp2`の値をHLLタイプのデータに変換し、`example10.csv`の`temp1`と`temp2`を`table10`の`col1`と`col2`にマッピングしています。
- `hll_empty` 関数は、`table10` の `col3` に指定したデフォルト値を埋めるために使用されます。

`hll_hash` 関数と `hll_empty` 関数の使用方法については、[hll_hash](../../sql-functions/aggregate-functions/hll_hash.md) と [hll_empty](../../sql-functions/aggregate-functions/hll_empty.md) を参照してください。

#### ファイルパスからパーティションフィールドの値を抽出

Broker Load は、宛先の StarRocks テーブルの列定義を基に、ファイルパスに含まれる特定のパーティションフィールドの値の解析をサポートしています。StarRocks のこの機能は、Apache Spark™ の Partition Discovery 機能に類似しています。

あなたの StarRocks データベース `test_db` には `table11` という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`、`city`、そして `utc_date` という 5 つの列で構成されています。

あなたの HDFS クラスターのファイルパス `/user/starrocks/data/input/dir/city=beijing` には、以下のデータファイルが含まれています:

- `/user/starrocks/data/input/dir/city=beijing/utc_date=2019-06-26/0000.csv`

- `/user/starrocks/data/input/dir/city=beijing/utc_date=2019-06-26/0001.csv`

これらのデータファイルは、それぞれ `col1`、`col2`、`col3` に順番にマップされる 3 つの列で構成されています。

もし、`/user/starrocks/data/input/dir/city=beijing/utc_date=*/*` のファイルパスから全データファイルを `table11` にロードし、同時にファイルパスに含まれるパーティションフィールド `city` と `utc_date` の値を抽出し、抽出した値を `table11` の `city` と `utc_date` にロードしたい場合は、次のコマンドを実行してください:

```SQL
LOAD LABEL test_db.label11
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/dir/city=beijing/*/*")
    INTO TABLE table11
    FORMAT AS "csv"
    (col1, col2, col3)
    COLUMNS FROM PATH AS (city, utc_date)
    SET (uniq_id = md5sum(k1, city))
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

#### `%3A` を含むファイルパスからのパーティションフィールドの値を抽出

HDFS では、ファイルパスにコロン（:）を含めることができません。すべてのコロン（:）は `%3A` に変換されます。

あなたの StarRocks データベース `test_db` には `table12` という名前のテーブルが含まれています。このテーブルは、`data_time`、`col1`、`col2` という 3 つの列で構成されています。テーブルのスキーマは以下のとおりです:

```SQL
data_time DATETIME,
col1        INT,
col2        INT
```

あなたの HDFS クラスターのファイルパス `/user/starrocks/data` には、以下のデータファイルが含まれています:

- `/user/starrocks/data/data_time=2020-02-17 00%3A00%3A00/example12.csv`

- `/user/starrocks/data/data_time=2020-02-18 00%3A00%3A00/example12.csv`

もし、`example12.csv` からすべてのデータを `table12` にロードし、同時にファイルパスからパーティションフィールド `data_time` の値を抽出し、抽出した値を `table12` の `data_time` にロードしたい場合は、次のコマンドを実行してください:

```SQL
LOAD LABEL test_db.label12
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/*/example12.csv")
    INTO TABLE table12
    COLUMNS TERMINATED BY ","
    FORMAT AS "csv"
    (col1,col2)
    COLUMNS FROM PATH AS (data_time)
    SET (data_time = str_to_date(data_time, '%Y-%m-%d %H%%3A%i%%3A%s'))
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

> **注意**
>
> 上記の例では、`data_time` のパーティションフィールドから抽出された値は、`2020-02-17 00%3A00%3A00` のように `%3A` を含む文字列です。そのため、これらの値を `table8` の `data_time` にロードする前に、`str_to_date` 関数を使って文字列を DATETIME 型のデータに変換する必要があります。

#### `format_type_options` の設定

あなたの StarRocks データベース `test_db` には `table13` という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3` という 3 つの列で構成されています。

あなたのデータファイル `example13.csv` もまた、`table13` の `col2`、`col1`、`col3` の順にマップされる 3 つの列で構成されています。

もし、`example13.csv` からすべてのデータを `table13` にロードし、`example13.csv` の最初の 2 行をスキップし、列区切り記号の前後の空白を削除し、`enclose` を `\`、`escape` を `\` に設定する場合は、次のコマンドを実行してください:

```SQL
LOAD LABEL test_db.label13
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/*/example13.csv")
    INTO TABLE table13
    COLUMNS TERMINATED BY ","
    FORMAT AS "CSV"
    (
        skip_header = 2
        trim_space = TRUE
        enclose = "\""
        escape = "\\"
    )
    (col2, col1, col3)
)
WITH BROKER
(
    "username" = "hdfs_username",
    "password" = "hdfs_password"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

### Parquet データのロード

このセクションでは、Parquet データのロード時に注意すべきいくつかのパラメータ設定について説明します。

あなたの StarRocks データベース `test_db` には `table13` という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3` という 3 つの列で構成されています。

あなたのデータファイル `example13.parquet` もまた、`table13` の `col1`、`col2`、`col3` の順にマップされる 3 つの列で構成されています。

もし、`example13.parquet` からすべてのデータを `table13` にロードしたい場合は、次のコマンドを実行してください:

```SQL
LOAD LABEL test_db.label13
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example13.parquet")
    INTO TABLE table13
    FORMAT AS "parquet"
    (col1, col2, col3)
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

> **注意**
>
> デフォルトでは、Parquet データをロードする際、StarRocks はファイル名に拡張子 **.parquet** が含まれているかどうかに基づいてデータファイル形式を決定します。ファイル名に拡張子 **.parquet** が含まれていない場合は、`FORMAT AS` を使用してデータファイル形式を `Parquet` に指定する必要があります。

### ORC データのロード

このセクションでは、ORC データのロード時に注意すべきいくつかのパラメータ設定について説明します。

あなたの StarRocks データベース `test_db` には `table14` という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3` という 3 つの列で構成されています。

あなたのデータファイル `example14.orc` もまた、`table14` の `col1`、`col2`、`col3` の順にマップされる 3 つの列で構成されています。

もし、`example14.orc` からすべてのデータを `table14` にロードしたい場合は、次のコマンドを実行してください:

```SQL
LOAD LABEL test_db.label14
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example14.orc")
    INTO TABLE table14
    FORMAT AS "orc"
    (col1, col2, col3)
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

> **注意**
>
> - デフォルトでは、ORC データをロードする際、StarRocks はファイル名に拡張子 **.orc** が含まれているかどうかに基づいてデータファイル形式を決定します。ファイル名に拡張子 **.orc** が含まれていない場合は、`FORMAT AS` を使用してデータファイル形式を `ORC` に指定する必要があります。
> - StarRocks v2.3 以前では、データファイルに ARRAY 型の列が含まれている場合、ORC データファイルの列名が StarRocks テーブルでのマッピング列名と同じであることを確認し、列を SET 句で指定してはいけません。