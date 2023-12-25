---
displayed_sidebar: English
---

# BROKER LOAD

import InsertPrivNote from '../../../assets/commonMarkdown/insertPrivNote.md'

## 説明

StarRocksは、MySQLベースのローディング方法であるBroker Loadを提供しています。ロードジョブを送信した後、StarRocksはジョブを非同期で実行します。`SELECT * FROM information_schema.loads`を使用してジョブ結果を照会できます。この機能はv3.1以降でサポートされています。背景情報、原理、サポートされるデータファイル形式、単一テーブルロードとマルチテーブルロードの実行方法、およびジョブ結果の確認方法については、[HDFSからデータをロードする](../../../loading/hdfs_load.md)と[クラウドストレージからデータをロードする](../../../loading/cloud_storage_load.md)を参照してください。

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

StarRocksでは、一部のリテラルがSQL言語によって予約キーワードとして使用されていることに注意してください。これらのキーワードをSQLステートメントで直接使用しないでください。このようなキーワードをSQLステートメントで使用したい場合は、バッククォート(`)で囲んでください。[キーワード](../../../sql-reference/sql-statements/keywords.md)を参照してください。

## パラメータ

### database_name と label_name

`label_name`はロードジョブのラベルを指定します。

`database_name`はオプションで、対象テーブルが属するデータベースの名前を指定します。

各ロードジョブには、データベース全体で一意のラベルがあります。ロードジョブのラベルを使用して、ロードジョブの実行状態を確認し、同じデータを繰り返しロードすることを防ぐことができます。ロードジョブが**FINISHED**状態になると、そのラベルは再利用できません。**CANCELLED**状態になったロードジョブのラベルのみが再利用可能です。通常、ロードジョブのラベルは、そのロードジョブを再試行し、同じデータをロードするために再利用され、これによってExactly-Onceセマンティクスが実現されます。

ラベルの命名規則については、[システム制限](../../../reference/System_limit.md)を参照してください。

### data_desc

ロードするデータのバッチの説明です。各`data_desc`記述子は、データソース、ETL関数、宛先StarRocksテーブル、宛先パーティションなどの情報を宣言します。

Broker Loadは、一度に複数のデータファイルをロードすることをサポートしています。1つのロードジョブで、複数の`data_desc`記述子を使用してロードするデータファイルを宣言することも、1つの`data_desc`記述子を使用してロードするすべてのデータファイルが含まれるファイルパスを宣言することもできます。Broker Loadはまた、複数のデータファイルをロードするために実行される各ロードジョブのトランザクションの原子性を保証することができます。原子性とは、1つのロードジョブで複数のデータファイルをロードする場合、すべて成功するか、すべて失敗することを意味します。一部のデータファイルのロードが成功し、他のファイルのロードが失敗することはありません。

`data_desc`は以下の構文をサポートしています：

```SQL
DATA INFILE ("<file_path>"[, "<file_path>" ...])
[NEGATIVE]
INTO TABLE <table_name>
[PARTITION (<partition1_name>[, <partition2_name> ...])]
[TEMPORARY PARTITION (<temporary_partition1_name>[, <temporary_partition2_name> ...])]
[COLUMNS TERMINATED BY "<column_separator>"]
[ROWS TERMINATED BY "<row_separator>"]
[FORMAT AS "CSV | Parquet | ORC"]
[(format_type_options)]
[(column_list)]
[COLUMNS FROM PATH AS (<partition_field_name>[, <partition_field_name> ...])]
[SET <k1=f1(v1)>[, <k2=f2(v2)> ...]]
[WHERE predicate]
```

`data_desc`には以下のパラメータが含まれる必要があります：

- `file_path`

  ロードしたい1つ以上のデータファイルの保存パスを指定します。

  このパラメータを1つのデータファイルの保存パスとして指定することができます。例えば、`"hdfs://<hdfs_host>:<hdfs_port>/user/data/tablename/20210411"`を指定して、HDFSサーバー上の`/user/data/tablename`パスから名前が`20210411`のデータファイルをロードすることができます。

  また、ワイルドカード`?`、`*`、`[]`、`{}`、`^`を使用して、複数のデータファイルの保存パスを指定することもできます。[ワイルドカードリファレンス](https://hadoop.apache.org/docs/stable/api/org/apache/hadoop/fs/FileSystem.html#globStatus-org.apache.hadoop.fs.Path-)を参照してください。例えば、`"hdfs://<hdfs_host>:<hdfs_port>/user/data/tablename/*/*"`や`"hdfs://<hdfs_host>:<hdfs_port>/user/data/tablename/dt=202104*/*"`を指定して、HDFSサーバー上の`/user/data/tablename`パス内のすべてのパーティションまたは`202104`パーティションのみからデータファイルをロードすることができます。

  > **注記**
  >
  > ワイルドカードは、中間パスを指定するためにも使用できます。

  上記の例で、`hdfs_host`と`hdfs_port`パラメータは以下のように説明されています：

  - `hdfs_host`: HDFSクラスター内のNameNodeホストのIPアドレス。
  - `hdfs_port`: HDFSクラスター内のNameNodeホストのFSポート。デフォルトのポート番号は`9000`です。

  > **注意**
  >
  > - Broker Loadは、S3またはS3Aプロトコルに従ってAWS S3にアクセスすることをサポートしています。したがって、AWS S3からデータをロードする際には、ファイルパスとして渡すS3 URIに`s3://`または`s3a://`のプレフィックスを含めることができます。
  > - Broker Loadは、gsプロトコルに従ってGoogle GCSにのみアクセスをサポートしています。したがって、Google GCSからデータをロードする場合は、ファイルパスとして渡すGCS URIに`gs://`のプレフィックスを含める必要があります。
  > - Blob Storageからデータをロードする場合は、wasbまたはwasbsプロトコルを使用してデータにアクセスする必要があります：
  >   - ストレージアカウントがHTTP経由でのアクセスを許可している場合は、wasbプロトコルを使用し、ファイルパスを`wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`として記述します。
  >   - ストレージアカウントがHTTPS経由でのアクセスを許可している場合は、wasbsプロトコルを使用し、ファイルパスを`wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`として記述します。
  > - Data Lake Storage Gen1からデータをロードする場合は、adlプロトコルを使用してデータにアクセスし、ファイルパスを`adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`として記述します。
  > - Data Lake Storage Gen2からデータをロードする場合は、abfsまたはabfssプロトコルを使用してデータにアクセスする必要があります：
  >   - ストレージアカウントがHTTP経由でのアクセスを許可している場合は、abfsプロトコルを使用し、ファイルパスを`abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>`として記述します。
  >   - ストレージアカウントがHTTPS経由でのアクセスを許可している場合は、abfssプロトコルを使用し、ファイルパスを`abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>`として記述します。

- `INTO TABLE`

  対象のStarRocksテーブルの名前を指定します。

`data_desc`には、オプションで以下のパラメータも含めることができます：

- `NEGATIVE`

  特定のデータバッチのロードを取り消します。これを実現するには、`NEGATIVE`キーワードを指定して同じデータバッチをロードする必要があります。

  > **注記**
  >
  > このパラメータは、StarRocksテーブルが集約テーブルであり、そのすべての値列が`sum`関数によって計算されている場合にのみ有効です。

- `PARTITION`

   データをロードするパーティションを指定します。デフォルトでは、このパラメータを指定しない場合、ソースデータはStarRocksテーブルのすべてのパーティションにロードされます。

- `TEMPORARY PARTITION`

  データをロードする[一時パーティション](../../../table_design/Temporary_partition.md)の名前を指定します。複数の一時パーティションを指定する場合は、コンマ(,)で区切る必要があります。

- `COLUMNS TERMINATED BY`


  データファイルで使用する列区切り文字を指定します。デフォルトでは、このパラメーターを指定しない場合、`\t`（タブ）がデフォルト値になります。このパラメーターで指定する列区切り文字は、データファイルで実際に使用される列区切り文字と同じでなければなりません。そうでない場合、データ品質が不十分でロードジョブが失敗し、その`State`は`CANCELLED`になります。

  Broker LoadジョブはMySQLプロトコルに従ってサブミットされます。StarRocksとMySQLは、ロードリクエスト内の文字をエスケープします。そのため、列区切り文字がタブなどの不可視文字の場合は、列区切り文字の前にバックスラッシュ（\）を追加する必要があります。例えば、列区切り文字が`\t`の場合は`\\t`と入力し、`\n`の場合は`\\n`と入力する必要があります。Apache Hive™ファイルは`\x01`を列区切り文字として使用するため、データファイルがHiveからのものである場合は`\\x01`と入力する必要があります。

  > **注記**
  >
  > - CSVデータでは、カンマ（,）、タブ、パイプ（|）など、50バイトを超えないUTF-8文字列をテキスト区切り文字として使用できます。
  > - Null値は`\N`を使用して表されます。例えば、データファイルが3つの列で構成されており、そのデータファイルのレコードが1列目と3列目にデータを保持しているが2列目にはデータがない場合、2列目に`\N`を使用してnull値を示す必要があります。つまり、レコードは`a,\N,b`としてコンパイルされるべきであり、`a,,b`ではありません。`a,,b`はレコードの2列目が空文字列であることを示します。

- `ROWS TERMINATED BY`

  データファイルで使用する行区切り文字を指定します。デフォルトでは、このパラメーターを指定しない場合、`\n`（改行）がデフォルト値になります。このパラメーターで指定する行区切り文字は、データファイルで実際に使用される行区切り文字と同じでなければなりません。そうでない場合、データ品質が不十分でロードジョブが失敗し、その`State`は`CANCELLED`になります。このパラメーターはv2.5.4以降でサポートされています。

  行区切り文字の使用に関する注意事項については、上記の`COLUMNS TERMINATED BY`パラメーターの使用に関する注意事項を参照してください。

- `FORMAT AS`

  データファイルの形式を指定します。有効な値は`CSV`、`Parquet`、`ORC`です。デフォルトでは、このパラメーターを指定しない場合、StarRocksは`file_path`パラメーターで指定されたファイル名拡張子（**.csv**、**.parquet**、または**.orc**）に基づいてデータファイル形式を決定します。

- `format_type_options`

  `FORMAT AS`が`CSV`に設定されている場合のCSV形式オプションを指定します。構文：

  ```JSON
  (
      key = value
      key = value
      ...
  )
  ```

  > **注記**
  >
  > `format_type_options`はv3.0以降でサポートされています。

  次の表にオプションを示します。

  | **パラメーター** | **説明**                                              |
  | ------------- | ------------------------------------------------------------ |
  | skip_header   | データファイルがCSV形式の場合、データファイルの最初の行をスキップするかどうかを指定します。タイプ: INTEGER。デフォルト値: `0`。<br />一部のCSV形式のデータファイルでは、先頭の最初の行が列名や列データ型などのメタデータを定義するために使用されます。`skip_header`パラメーターを設定することで、StarRocksはデータロード中にデータファイルの最初の行をスキップすることができます。例えば、このパラメーターを`1`に設定した場合、StarRocksはデータロード中にデータファイルの最初の行をスキップします。<br />データファイルの先頭の最初の行は、ロードステートメントで指定する行区切り文字を使用して区切られている必要があります。 |
  | trim_space    | データファイルがCSV形式の場合、列区切り文字の前後のスペースを削除するかどうかを指定します。タイプ: BOOLEAN。デフォルト値: `false`。<br />一部のデータベースでは、データをCSV形式のデータファイルとしてエクスポートする際に列区切り文字にスペースが追加されます。このようなスペースは、その位置に応じて先頭のスペースまたは末尾のスペースと呼ばれます。`trim_space`パラメーターを設定することで、StarRocksはデータロード中にこのような不要なスペースを削除できます。<br />StarRocksは、`enclose`で指定された文字のペアに囲まれたフィールド内のスペース（先頭のスペースと末尾のスペースを含む）を削除しないことに注意してください。例えば、次のフィールド値ではパイプ（<code class="language-text">&#124;</code>）を列区切り文字として、ダブルクォーテーション（`"`）を`enclose`で指定された文字として使用します：<br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> <br /><code class="language-text">&#124;" Love StarRocks "&#124;</code> <br /><code class="language-text">&#124; "Love StarRocks" &#124;</code> <br />`trim_space`を`true`に設定すると、StarRocksは上記のフィールド値を次のように処理します：<br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> <br /><code class="language-text">&#124;" Love StarRocks "&#124;</code> <br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> |
  | enclose       | データファイルがCSV形式の場合、フィールド値を囲むために使用する文字を[RFC4180](https://www.rfc-editor.org/rfc/rfc4180)に従って指定します。タイプ: 単一バイト文字。デフォルト値: `NONE`。一般的な文字はシングルクォート（`'`）とダブルクォート（`"`）です。<br />`enclose`で指定された文字で囲まれたすべての特殊文字（行区切り文字と列区切り文字を含む）は通常の記号と見なされます。StarRocksはRFC4180以上のことができ、任意の単一バイト文字を`enclose`で指定された文字として指定できます。<br />フィールド値に`enclose`で指定された文字が含まれている場合、その`enclose`で指定された文字をエスケープするために同じ文字を使用できます。例えば、`enclose`を`"`に設定し、フィールド値が`a "quoted" c`である場合、データファイルには`"a ""quoted"" c"`としてフィールド値を入力できます。 |
  | escape        | 行区切り文字、列区切り文字、エスケープ文字、`enclose`で指定された文字など、さまざまな特殊文字をエスケープするために使用される文字を指定します。これらの文字はStarRocksによって通常の文字と見なされ、それらが存在するフィールド値の一部として解析されます。タイプ: 単一バイト文字。デフォルト値: `NONE`。一般的な文字はバックスラッシュ（`\`）で、SQLステートメントではダブルバックスラッシュ（`\\`）として記述する必要があります。<br />**注記**<br />`escape`で指定された文字は、`enclose`で指定された文字のペアの内側と外側の両方に適用されます。<br />例えば、`enclose`を`"`に、`escape`を`\`に設定した場合、StarRocksは`"say \"Hello world\""`を`say "Hello world"`として解析します。列区切り文字がカンマ（`,`）で、`escape`を`\`に設定した場合、StarRocksは`a, b\, c`を`a`と`b, c`の2つの別々のフィールド値として解析します。 |

- `column_list`

  データファイルとStarRocksテーブル間の列マッピングを指定します。構文: `(<column_name>[, <column_name> ...])`。`column_list`で宣言されたカラムは、名前によってStarRocksテーブルのカラムにマッピングされます。

  > **注記**
  >
  > データファイルのカラムがStarRocksテーブルのカラムに順番にマッピングされる場合、`column_list`を指定する必要はありません。

  データファイルの特定のカラムをスキップしたい場合は、そのカラムにStarRocksテーブルのカラムとは異なる名前を一時的に付けるだけで済みます。詳細については、[ロード時のデータ変換](../../../loading/Etl_in_loading.md)を参照してください。

- `COLUMNS FROM PATH AS`

  指定したファイルパスから1つ以上のパーティションフィールドに関する情報を抽出します。このパラメーターは、ファイルパスにパーティションフィールドが含まれている場合にのみ有効です。

  例えば、データファイルがパス `/path/col_name=col_value/file1` に格納されており、`col_name` がパーティションフィールドでStarRocksテーブルの列にマッピング可能な場合、このパラメータを `col_name` として指定できます。そうすると、StarRocksはパスから `col_value` の値を抽出し、`col_name` にマッピングされたStarRocksテーブルの列にロードします。

  > **注記**
  >
  > このパラメータは、HDFSからデータをロードする際にのみ利用可能です。

- `SET`

  データファイルの列を変換するために使用したい関数を一つ以上指定します。例えば：

  - StarRocksテーブルには `col1`、`col2`、`col3` の3つの列があり、データファイルには4つの列があります。そのうち最初の2列は `col1` と `col2` に順番にマッピングされ、最後の2列の合計は `col3` にマッピングされます。この場合、`column_list` を `(col1,col2,tmp_col3,tmp_col4)` と指定し、SET句で `(col3=tmp_col3+tmp_col4)` を指定してデータ変換を実装する必要があります。
  - StarRocksテーブルには `year`、`month`、`day` の3つの列があり、データファイルには `yyyy-mm-dd hh:mm:ss` 形式の日付と時刻の値を格納する1つの列のみがあります。この場合、`column_list` を `(tmp_time)` と指定し、SET句で `(year = year(tmp_time), month=month(tmp_time), day=day(tmp_time))` を指定してデータ変換を実装する必要があります。

- `WHERE`

  ソースデータをフィルタリングする条件を指定します。StarRocksは、WHERE句で指定されたフィルタ条件を満たすソースデータのみをロードします。

### WITH BROKER

v2.3以前では、`WITH BROKER "<broker_name>"` を入力して使用したいブローカーを指定します。v2.5以降では、ブローカーを指定する必要はありませんが、`WITH BROKER` キーワードは引き続き必要です。

> **注記**
>
> v2.4以前のStarRocksは、Broker Loadジョブを実行する際に、StarRocksクラスタと外部ストレージシステム間の接続を確立するためにブローカーに依存していました。これは「ブローカーベースのローディング」と呼ばれます。ブローカーは、ファイルシステムインターフェースと統合された独立したステートレスサービスです。ブローカーを使用することで、StarRocksは外部ストレージシステムに保存されているデータファイルにアクセスし、読み取り、独自のコンピューティングリソースを使用してこれらのデータファイルのデータを前処理し、ロードすることができます。
>
> v2.5以降、StarRocksはブローカーへの依存を取り除き、「ブローカーフリーのローディング」を実装しました。
>
> [SHOW BROKER](../Administration/SHOW_BROKER.md) ステートメントを使用して、StarRocksクラスタにデプロイされているブローカーを確認できます。ブローカーがデプロイされていない場合は、[ブローカーをデプロイする](../../../deployment/deploy_broker.md)に記載されている手順に従ってブローカーをデプロイできます。

### StorageCredentialParams

StarRocksがストレージシステムにアクセスするために使用する認証情報です。

#### HDFS

オープンソースのHDFSは、シンプル認証とKerberos認証の2つの認証方法をサポートしています。Broker Loadはデフォルトでシンプル認証を使用します。オープンソースのHDFSは、NameNodeのHAメカニズムの設定もサポートしています。ストレージシステムとしてオープンソースのHDFSを選択する場合、認証設定とHA設定を次のように指定できます：

- 認証設定

  - シンプル認証を使用する場合、`StorageCredentialParams` を次のように設定します：

    ```Plain
    "hadoop.security.authentication" = "simple",
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
    ```

    以下の表は `StorageCredentialParams` のパラメーターの説明です。

    | パラメーター                       | 説明                                                  |
    | ------------------------------- | ------------------------------------------------------------ |
    | hadoop.security.authentication  | 認証方法。有効な値: `simple` と `kerberos`。デフォルト値: `simple`。`simple` は認証なしを意味するシンプル認証を表し、`kerberos` はKerberos認証を表します。 |
    | username                        | HDFSクラスタのNameNodeにアクセスするためのアカウントのユーザー名。 |
    | password                        | HDFSクラスタのNameNodeにアクセスするためのアカウントのパスワード。 |

  - Kerberos認証を使用する場合、`StorageCredentialParams` を次のように設定します：

    ```Plain
    "hadoop.security.authentication" = "kerberos",
    "kerberos_principal" = "nn/zelda1@ZELDA.COM",
    "kerberos_keytab" = "/keytab/hive.keytab",
    "kerberos_keytab_content" = "YWFhYWFh"
    ```

    以下の表は `StorageCredentialParams` のパラメーターの説明です。

    | パラメーター                       | 説明                                                  |
    | ------------------------------- | ------------------------------------------------------------ |
    | hadoop.security.authentication  | 認証方法。有効な値: `simple` と `kerberos`。デフォルト値: `simple`。`simple` は認証なしを意味するシンプル認証を表し、`kerberos` はKerberos認証を表します。 |
    | kerberos_principal              | 認証されるKerberosプリンシパル。各プリンシパルは、HDFSクラスタ全体で一意であることを保証するために、次の3部分で構成されます：<ul><li>`username` または `servicename`：プリンシパルの名前。</li><li>`instance`：HDFSクラスタで認証されるノードをホストするサーバーの名前。サーバー名は、例えばHDFSクラスタがそれぞれが独立して認証される複数のDataNodeで構成されている場合に、プリンシパルが一意であることを確認するのに役立ちます。</li><li>`realm`：レルムの名前。レルム名は大文字でなければなりません。例：`nn/zelda1@ZELDA.COM`。</li></ul> |
    | kerberos_keytab                 | Kerberosキータブファイルの保存パス。 |
    | kerberos_keytab_content         | KerberosキータブファイルのBase64エンコードされた内容。`kerberos_keytab` または `kerberos_keytab_content` のいずれかを指定できます。 |

    複数のKerberosユーザーを設定している場合、少なくとも1つの独立した[ブローカーグループ](../../../deployment/deploy_broker.md)がデプロイされていることを確認し、ロードステートメントで `WITH BROKER "<broker_name>"` を入力して使用するブローカーグループを指定する必要があります。さらに、ブローカーの起動スクリプトファイル **start_broker.sh** を開き、ファイルの42行目を変更してブローカーが **krb5.conf** ファイルを読み取ることができるようにします。例：

    ```Plain
    export JAVA_OPTS="-Dlog4j2.formatMsgNoLookups=true -Xmx1024m -Dfile.encoding=UTF-8 -Djava.security.krb5.conf=/etc/krb5.conf"
    ```

    > **注記**
    >
    > - 上記の例では、`/etc/krb5.conf` は **krb5.conf** ファイルの実際の保存パスに置き換えることができます。ブローカーがそのファイルを読み取る権限を持っていることを確認してください。ブローカーグループが複数のブローカーで構成されている場合は、各ブローカーノードで **start_broker.sh** ファイルを変更し、ブローカーノードを再起動して変更を有効にする必要があります。
    > - [SHOW BROKER](../Administration/SHOW_BROKER.md) ステートメントを使用して、StarRocksクラスタにデプロイされているブローカーを確認できます。

- HA設定

  HDFSクラスタのNameNodeに対するHAメカニズムを設定できます。これにより、NameNodeが別のノードに切り替わった場合、StarRocksは新しいNameNodeとして機能するノードを自動的に識別できます。これには以下のシナリオが含まれます：

  - Kerberosユーザーが1人だけ設定されている単一のHDFSクラスタからデータをロードする場合、ロードベースのローディングとロードフリーローディングの両方がサポートされています。
  
    - ロードベースのローディングを実行するには、少なくとも1つの独立した[ブローカーグループ](../../../deployment/deploy_broker.md)がデプロイされていることを確認し、`hdfs-site.xml` ファイルをHDFSクラスタにサービスを提供するブローカーノードの `{deploy}/conf` パスに配置します。StarRocksはブローカーの起動時に `{deploy}/conf` パスを環境変数 `CLASSPATH` に追加し、ブローカーがHDFSクラスタノードに関する情報を読み取ることができるようにします。
  
    - ロードフリーローディングを実行するには、`hdfs-site.xml` ファイルを各FEノードと各BEノードの `{deploy}/conf` パスに配置します。
  
  - 複数のKerberosユーザーが設定された単一のHDFSクラスターからデータをロードする場合、ブローカーベースのロードのみがサポートされます。少なくとも1つの独立した[ブローカーグループ](../../../deployment/deploy_broker.md)がデプロイされていることを確認し、`hdfs-site.xml`ファイルをHDFSクラスターにサービスを提供するブローカーノードの`{deploy}/conf`パスに配置します。StarRocksはブローカーの起動時に`{deploy}/conf`パスを環境変数`CLASSPATH`に追加し、ブローカーがHDFSクラスターノードに関する情報を読み取れるようにします。

  - 複数のHDFSクラスターからデータをロードする場合（1人または複数のKerberosユーザーが設定されているかどうかに関わらず）、ブローカーベースのロードのみがサポートされます。これらのHDFSクラスターごとに少なくとも1つの独立した[ブローカーグループ](../../../deployment/deploy_broker.md)がデプロイされていることを確認し、以下のいずれかのアクションを実行して、ブローカーがHDFSクラスターノードに関する情報を読み取れるようにします。

    - 各HDFSクラスターにサービスを提供するブローカーノードの`{deploy}/conf`パスに`hdfs-site.xml`ファイルを配置します。StarRocksはブローカーの起動時に`{deploy}/conf`パスを環境変数`CLASSPATH`に追加し、ブローカーがそのHDFSクラスター内のノードに関する情報を読み取れるようにします。

    - ジョブ作成時に以下のHA設定を追加します。

      ```Plain
      "dfs.nameservices" = "ha_cluster",
      "dfs.ha.namenodes.ha_cluster" = "ha_n1,ha_n2",
      "dfs.namenode.rpc-address.ha_cluster.ha_n1" = "<hdfs_host>:<hdfs_port>",
      "dfs.namenode.rpc-address.ha_cluster.ha_n2" = "<hdfs_host>:<hdfs_port>",
      "dfs.client.failover.proxy.provider" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
      ```

      次の表は、HA設定のパラメーターを説明しています。

      | パラメーター                          | 説明                                                  |
      | ---------------------------------- | ------------------------------------------------------------ |
      | dfs.nameservices                   | HDFSクラスターの名前。                                |
      | dfs.ha.namenodes.XXX               | HDFSクラスター内のNameNodeの名前。複数のNameNode名を指定する場合は、コンマ(`,`)で区切ります。`XXX`は`dfs.nameservices`で指定したHDFSクラスター名です。 |
      | dfs.namenode.rpc-address.XXX.NN    | HDFSクラスター内のNameNodeのRPCアドレス。`NN`は`dfs.ha.namenodes.XXX`で指定したNameNode名です。 |
      | dfs.client.failover.proxy.provider | クライアントが接続するNameNodeのプロバイダー。デフォルト値は`org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider`です。 |

  > **注記**
  >
  > StarRocksクラスターにデプロイされているブローカーを確認するには、[SHOW BROKER](../Administration/SHOW_BROKER.md)ステートメントを使用できます。

#### AWS S3

ストレージシステムとしてAWS S3を選択する場合、以下のいずれかのアクションを実行します。

- インスタンスプロファイルベースの認証方法を選択する場合、`StorageCredentialParams`を次のように設定します。

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- アサムドロールベースの認証方法を選択する場合、`StorageCredentialParams`を次のように設定します。

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAMユーザーベースの認証方法を選択する場合、`StorageCredentialParams`を次のように設定します。

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

以下の表は、`StorageCredentialParams`で設定する必要があるパラメータを説明しています。

| パラメーター                   | 必須 | 説明                                                  |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | はい      | インスタンスプロファイルとアサムドロールの認証方法を有効にするかどうかを指定します。有効な値は`true`と`false`です。デフォルト値は`false`です。 |
| aws.s3.iam_role_arn         | いいえ       | AWS S3バケットに権限を持つIAMロールのARNです。AWS S3へのアクセスにアサムドロールを認証方法として選択する場合、このパラメータを指定する必要があります。 |
| aws.s3.region               | はい      | AWS S3バケットが存在するリージョンです。例：`us-west-1`。 |
| aws.s3.access_key           | いいえ       | IAMユーザーのアクセスキーです。AWS S3へのアクセスにIAMユーザーを認証方法として選択する場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | いいえ       | IAMユーザーのシークレットキーです。AWS S3へのアクセスにIAMユーザーを認証方法として選択する場合、このパラメータを指定する必要があります。 |

AWS S3へのアクセスに認証方法を選択し、AWS IAMコンソールでアクセス制御ポリシーを設定する方法については、[AWS S3へのアクセス認証パラメータ](../../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

#### Google GCS

ストレージシステムとしてGoogle GCSを選択する場合、以下のいずれかのアクションを実行します。

- VMベースの認証方法を選択する場合、`StorageCredentialParams`を次のように設定します。

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  以下の表は、`StorageCredentialParams`で設定する必要があるパラメータを説明しています。

  | **パラメーター**                              | **デフォルト値** | **値の例** | **説明**                                              |
  | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | false             | true                  | Compute Engineにバインドされているサービスアカウントを直接使用するかどうかを指定します。 |

- サービスアカウントベースの認証方法を選択する場合、`StorageCredentialParams`を次のように設定します。

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  以下の表は、`StorageCredentialParams`で設定する必要があるパラメータを説明しています。

  | **パラメーター**                          | **デフォルト値** | **値の例**                                        | **説明**                                              |
  | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""                | "user@hello.iam.gserviceaccount.com" | サービスアカウント作成時に生成されたJSONファイル内のメールアドレスです。 |
  | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | サービスアカウント作成時に生成されたJSONファイル内のプライベートキーIDです。 |
  | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n"  | サービスアカウント作成時に生成されたJSONファイル内のプライベートキーです。 |

- インパーソネーションベースの認証方法を選択する場合、`StorageCredentialParams`を次のように設定します。

  - VMインスタンスがサービスアカウントをインパーソネートする場合：

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    以下の表は、`StorageCredentialParams`で設定する必要があるパラメータを説明しています。

    | **パラメーター**                              | **デフォルト値** | **値の例** | **説明**                                              |
    | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false             | true                  | Compute Engineにバインドされているサービスアカウントを直接使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""                | "hello"               | インパーソネートするサービスアカウントです。            |

  - サービスアカウント（メタサービスアカウントとして命名）が別のサービスアカウント（データサービスアカウントとして命名）をインパーソネートする場合：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_private_key_id>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_private_key>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    以下の表は、`StorageCredentialParams`で設定する必要があるパラメータを説明しています。

    | **パラメーター**                          | **デフォルト値** | **値の例**                                        | **説明**                                              |
    | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | メタサービスアカウントの作成時に生成されたJSONファイル内のメールアドレス。 |
    | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | メタサービスアカウントの作成時に生成されたJSONファイル内のプライベートキーID。 |
    | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n"  | メタサービスアカウントの作成時に生成されたJSONファイル内のプライベートキー。 |
    | gcp.gcs.impersonation_service_account  | ""                | "hello"                                                      | 偽装したいデータサービスアカウント。       |

#### その他のS3互換ストレージシステム

MinIOなどの他のS3互換ストレージシステムを選択する場合は、`StorageCredentialParams` を次のように構成します：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下の表は、`StorageCredentialParams` で設定する必要があるパラメーターを説明しています。

| パラメーター                        | 必須 | 説明                                                  |
| -------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                | はい      | SSL接続を有効にするかどうかを指定します。有効な値: `true` と `false`。デフォルト値: `true`。 |
| aws.s3.enable_path_style_access  | はい      | パス形式のURLアクセスを有効にするかどうかを指定します。有効な値: `true` と `false`。デフォルト値: `false`。MinIOの場合は、値を `true` に設定する必要があります。 |
| aws.s3.endpoint                  | はい      | AWS S3ではなく、S3互換ストレージシステムに接続するために使用されるエンドポイント。 |
| aws.s3.access_key                | はい      | IAMユーザーのアクセスキー。 |
| aws.s3.secret_key                | はい      | IAMユーザーのシークレットキー。 |

#### Microsoft Azure ストレージ

##### Azure Blob Storage

ストレージシステムとしてBlob Storageを選択した場合、以下のいずれかのアクションを実行します：

- 共有キー認証方法を選択する場合、`StorageCredentialParams` を次のように構成します：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  以下の表は、`StorageCredentialParams` で設定する必要があるパラメーターを説明しています。

  | **パラメーター**              | **必須** | **説明**                              |
  | -------------------------- | ------------ | -------------------------------------------- |
  | azure.blob.storage_account | はい          | Blob Storageアカウントのユーザー名。   |
  | azure.blob.shared_key      | はい          | Blob Storageアカウントの共有キー。 |

- SASトークン認証方法を選択する場合、`StorageCredentialParams` を次のように構成します：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.container" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  以下の表は、`StorageCredentialParams` で設定する必要があるパラメーターを説明しています。

  | **パラメーター**             | **必須** | **説明**                                              |
  | ------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.blob.storage_account| はい          | Blob Storageアカウントのユーザー名。                   |
  | azure.blob.container      | はい          | データを格納するblobコンテナの名前。        |
  | azure.blob.sas_token      | はい          | Blob Storageアカウントへのアクセスに使用されるSASトークン。 |

##### Azure Data Lake Storage Gen1

ストレージシステムとしてData Lake Storage Gen1を選択した場合、以下のいずれかのアクションを実行します：

- 管理されたサービスID認証方法を選択する場合、`StorageCredentialParams` を次のように構成します：

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  以下の表は、`StorageCredentialParams` で設定する必要があるパラメーターを説明しています。

  | **パラメーター**                            | **必須** | **説明**                                              |
  | ---------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | はい          | 管理されたサービスID認証方法を有効にするかどうかを指定します。値を `true` に設定します。 |

- サービスプリンシパル認証方法を選択する場合、`StorageCredentialParams` を次のように構成します：

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  以下の表は、`StorageCredentialParams` で設定する必要があるパラメーターを説明しています。

  | **パラメーター**                 | **必須** | **説明**                                              |
  | ----------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | はい          | サービスプリンシパルまたはアプリケーションのクライアント（アプリケーション）ID。 |
  | azure.adls1.oauth2_credential | はい          | 作成された新しいクライアント（アプリケーション）シークレットの値。    |
  | azure.adls1.oauth2_endpoint   | はい          | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）。 |

##### Azure Data Lake Storage Gen2

ストレージシステムとしてData Lake Storage Gen2を選択した場合、以下のいずれかのアクションを実行します：

- マネージドアイデンティティ認証方法を選択する場合、`StorageCredentialParams` を次のように構成します：

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  以下の表は、`StorageCredentialParams` で設定する必要があるパラメーターを説明しています。

  | **パラメーター**                           | **必須** | **説明**                                              |
  | --------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | はい          | マネージドアイデンティティ認証方法を有効にするかどうかを指定します。値を `true` に設定します。 |
  | azure.adls2.oauth2_tenant_id            | はい          | アクセスしたいテナントのID。          |
  | azure.adls2.oauth2_client_id            | はい          | マネージドアイデンティティのクライアント（アプリケーション）ID。         |

- 共有キー認証方法を選択する場合、`StorageCredentialParams` を次のように構成します：

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  以下の表は、`StorageCredentialParams` で設定する必要があるパラメーターを説明しています。

  | **パラメーター**               | **必須** | **説明**                                              |
  | --------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.storage_account | はい          | Data Lake Storage Gen2ストレージアカウントのユーザー名。 |
  | azure.adls2.shared_key      | はい          | Data Lake Storage Gen2ストレージアカウントの共有キー。 |

- サービスプリンシパル認証方法を選択する場合、`StorageCredentialParams` を次のように構成します：

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  以下の表は、`StorageCredentialParams` で設定する必要があるパラメーターを説明しています。

  | **パラメーター**                      | **必須** | **説明**                                              |
  | ---------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | はい          | サービスプリンシパルのクライアント（アプリケーション）ID。        |
  | azure.adls2.oauth2_client_secret   | はい          | 作成された新しいクライアント（アプリケーション）シークレットの値。    |
  | azure.adls2.oauth2_client_endpoint | はい          | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）。 |

### opt_properties

ロードジョブ全体に適用されるいくつかのオプションパラメーターを指定します。構文：

```Plain
PROPERTIES ("<key1>" = "<value1>"[, "<key2>" = "<value2>" ...])
```

以下のパラメーターがサポートされています：

- `timeout`

  ロードジョブのタイムアウト期間を指定します。単位は秒です。デフォルトのタイムアウト期間は4時間です。6時間未満のタイムアウト期間を指定することを推奨します。ロードジョブがタイムアウト期間内に終了しない場合、StarRocksはロードジョブをキャンセルし、そのステータスは**CANCELLED**になります。

  > **注記**
  >
  > ほとんどの場合、タイムアウト期間を設定する必要はありません。ロードジョブがデフォルトのタイムアウト期間内に終了できない場合のみ、タイムアウト期間を設定することを推奨します。

  タイムアウト期間を推定するには、以下の式を使用します：

  **タイムアウト期間 > (ロードするデータファイルの合計サイズ x ロードするデータファイルの数とデータファイルに作成されたマテリアライズドビューの数) / 平均ロード速度**

  > **注記**
  >
  > 「平均ロード速度」とは、StarRocksクラスタ全体の平均ロード速度のことです。平均ロード速度は、サーバー構成とクラスターで許可される同時クエリタスクの最大数によって、クラスターごとに異なります。平均ロード速度は、過去のロードジョブのロード速度に基づいて推測することができます。

  例えば、平均ロード速度が10 MB/sのStarRocksクラスタに、2つのマテリアライズドビューが作成された1 GBのデータファイルをロードする場合、データロードに必要な時間は約102秒です。

  (1 x 1024 x 3)/10 = 307.2 (秒)

  この例では、タイムアウト期間を308秒以上に設定することを推奨します。

- `max_filter_ratio`

  ロードジョブの最大エラー許容率を指定します。最大エラー許容率とは、データ品質が不十分なためにフィルタリングされる行の最大割合です。有効な値: `0`～`1`。デフォルト値: `0`。

  - このパラメータを`0`に設定すると、StarRocksはロード中に不適格な行を無視しません。その結果、ソースデータに不適格な行が含まれている場合、ロードジョブは失敗します。これにより、StarRocksにロードされたデータの正確性が保証されます。

  - このパラメータを`0`より大きい値に設定すると、StarRocksはロード中に不適格な行を無視することができます。その結果、ソースデータに不適格な行が含まれていても、ロードジョブが成功する可能性があります。

    > **注記**
    >
    > データ品質が不十分なためにフィルタリングされた行には、WHERE句によってフィルタリングされた行は含まれません。

  最大エラー許容率が`0`に設定されているためにロードジョブが失敗した場合、[SHOW LOAD](../../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)を使用してジョブ結果を表示し、不適格な行をフィルタリングできるかどうかを判断します。不適格な行をフィルタリングできる場合は、ジョブ結果で返された`dpp.abnorm.ALL`と`dpp.norm.ALL`の値に基づいて最大エラー許容率を計算し、最大エラー許容率を調整してロードジョブを再実行してください。最大エラー許容率の計算式は以下の通りです。

  `max_filter_ratio` = [`dpp.abnorm.ALL` / (`dpp.abnorm.ALL` + `dpp.norm.ALL`)]

  `dpp.abnorm.ALL`と`dpp.norm.ALL`の合計は、ロードされる行の総数です。

- `log_rejected_record_num`

  ログに記録される不適格データ行の最大数を指定します。このパラメータはv3.1以降でサポートされています。有効な値: `0`、`-1`、および0以外の正の整数。デフォルト値: `0`。
  
  - 値`0`は、フィルタリングされたデータ行がログに記録されないことを指定します。
  - 値`-1`は、フィルタリングされたすべてのデータ行がログに記録されることを指定します。
  - 0以外の正の整数（例えば`n`）は、各BEでフィルタリングされた最大`n`行のデータ行がログに記録されることを指定します。

- `load_mem_limit`

  ロードジョブに割り当てられるメモリの最大量を指定します。単位: バイト。デフォルトのメモリ制限は2 GBです。

- `strict_mode`

  [厳格モード](../../../loading/load_concept/strict_mode.md)を有効にするかどうかを指定します。有効な値: `true`および`false`。デフォルト値: `false`。`true`は厳格モードを有効にすることを指定し、`false`は厳格モードを無効にすることを指定します。

- `timezone`

  ロードジョブのタイムゾーンを指定します。デフォルト値: `Asia/Shanghai`。タイムゾーンの設定は、strftime、alignment_timestamp、from_unixtimeなどの関数によって返される結果に影響します。詳細については、[タイムゾーンの設定](../../../administration/timezone.md)を参照してください。`timezone`パラメータで指定されるタイムゾーンはセッションレベルのタイムゾーンです。

- `priority`

  ロードジョブの優先順位を指定します。有効な値: `LOWEST`、`LOW`、`NORMAL`、`HIGH`、および`HIGHEST`。デフォルト値: `NORMAL`。Broker Loadは[FEパラメータ](../../../administration/FE_configuration.md#fe-configuration-items) `max_broker_load_job_concurrency`を提供し、StarRocksクラスタ内で同時に実行できるBroker Loadジョブの最大数を決定します。指定された期間内に提出されたBroker Loadジョブの数が最大数を超える場合、余剰なジョブは優先順位に基づいてスケジュールされるのを待機します。

  [ALTER LOAD](../../../sql-reference/sql-statements/data-manipulation/ALTER_LOAD.md)ステートメントを使用して、`QUEUEING`または`LOADING`状態の既存のロードジョブの優先順位を変更することができます。

  StarRocksはv2.5以降、Broker Loadジョブの`priority`パラメータを設定することができます。

## 列マッピング

データファイルの列がStarRocksテーブルの列に一対一で順番にマッピングできる場合、データファイルとStarRocksテーブル間の列マッピングを設定する必要はありません。

データファイルの列がStarRocksテーブルの列に一対一で順番にマッピングできない場合、`columns`パラメータを使用してデータファイルとStarRocksテーブル間の列マッピングを設定する必要があります。これには以下の2つのユースケースが含まれます：

- **列数は同じですが、列の順序が異なります。** **また、データファイルからのデータは、対応するStarRocksテーブルの列にロードされる前に関数によって計算する必要はありません。**

  `columns`パラメータでは、データファイルの列の配置と同じ順序でStarRocksテーブルの列の名前を指定する必要があります。

  例えば、StarRocksテーブルには`col1`、`col2`、`col3`の順に3つの列があり、データファイルにも3つの列があり、それらがStarRocksテーブルの`col3`、`col2`、`col1`の順にマッピングできる場合、`"columns: col3, col2, col1"`と指定する必要があります。

- **列数も列の順序も異なり、データファイルからのデータは対応するStarRocksテーブルの列にロードされる前に関数によって計算する必要があります。**

  `columns`パラメータでは、データファイルの列の配置と同じ順序でStarRocksテーブルの列の名前を指定し、データを計算するために使用する関数も指定する必要があります。以下に2つの例を示します：

  - StarRocksテーブルには`col1`、`col2`、`col3`の順に3つの列があり、データファイルには4つの列があり、最初の3つの列はStarRocksテーブルの`col1`、`col2`、`col3`に順番にマッピングできますが、4番目の列はどのStarRocksテーブルの列にもマッピングできません。この場合、データファイルの4番目の列に一時的な名前を指定し、その一時的な名前はStarRocksテーブルの列名と異なるものである必要があります。例えば、`"columns: col1, col2, col3, temp"`と指定し、データファイルの4番目の列に一時的な名前`temp`を付けます。
  - StarRocksテーブルには`year`、`month`、`day`の順に3つの列があり、データファイルには`yyyy-mm-dd hh:mm:ss`形式の日付と時刻の値を格納する1つの列のみがあります。この場合、`"columns: col, year = year(col), month = month(col), day = day(col)"`と指定し、`col`はデータファイルの列の一時名とし、関数`year = year(col)`、`month = month(col)`、`day = day(col)`を使用してデータファイルの列`col`からデータを抽出し、対応するStarRocksテーブルの列にロードします。例えば、`year = year(col)`はデータファイルの列`col`から`yyyy`のデータを抽出し、StarRocksテーブルの`year`列にロードするために使用されます。

詳細な例については、[列マッピングの設定](#configure-column-mapping)を参照してください。

## 関連する設定項目

[FE設定項目](../../../administration/FE_configuration.md#fe-configuration-items)の`max_broker_load_job_concurrency`は、StarRocksクラスター内で同時に実行できるBroker Loadジョブの最大数を指定します。

StarRocks v2.4以前では、特定の期間内に送信されたBroker Loadジョブの総数が最大数を超えた場合、超過したジョブは送信時間に基づいてキューに入れられ、スケジュールされます。

StarRocks v2.5以降では、特定の期間内に送信されたBroker Loadジョブの総数が最大数を超えると、超過したジョブは優先度に基づいてキューに入れられ、スケジュールされます。ジョブの優先度を指定するには、上述の`priority`パラメータを使用します。[ALTER LOAD](../data-manipulation/ALTER_LOAD.md)を使用して、**QUEUEING**または**LOADING**状態の既存のジョブの優先順位を変更できます。

## 例

このセクションでは、HDFSを例にして、さまざまなロード設定について説明します。

### CSVデータのロード

このセクションでは、CSVを例にして、さまざまなロード要件を満たすために使用できるさまざまなパラメータ設定について説明します。

#### タイムアウト期間の設定

StarRocksデータベース`test_db`には`table1`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3列で構成されています。

データファイル`example1.csv`も3列で構成されており、`table1`の`col1`、`col2`、`col3`に順番にマッピングされます。

`example1.csv`からのすべてのデータを最大3600秒以内に`table1`にロードしたい場合は、次のコマンドを実行します：

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

#### 最大エラー許容数の設定

StarRocksデータベース`test_db`には`table2`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3列で構成されています。

データファイル`example2.csv`も3列で構成されており、`table2`の`col1`、`col2`、`col3`に順番にマッピングされます。

`example2.csv`からのすべてのデータを最大エラー許容数`0.1`で`table2`にロードしたい場合は、次のコマンドを実行します：

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

#### ファイルパスからのすべてのデータファイルのロード

StarRocksデータベース`test_db`には`table3`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3列で構成されています。

HDFSクラスターの`/user/starrocks/data/input/`パスに格納されているすべてのデータファイルもそれぞれ3列で構成されており、`table3`の`col1`、`col2`、`col3`に順番にマッピングされます。これらのデータファイルで使用される列区切り文字は`\x01`です。

`hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/`パスに格納されているこれらのすべてのデータファイルから`table3`にデータをロードしたい場合は、次のコマンドを実行します：

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

StarRocksデータベース`test_db`には`table4`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3列で構成されています。

データファイル`example4.csv`も3列で構成されており、`table4`の`col1`、`col2`、`col3`にマッピングされます。

NameNode用に構成されたHAメカニズムを使用して`example4.csv`からのすべてのデータを`table4`にロードしたい場合は、次のコマンドを実行します：

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
    "dfs.client.failover.proxy.provider.my_ha" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
);
```

#### Kerberos認証の設定

StarRocksデータベース`test_db`には`table5`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3列で構成されています。

データファイル`example5.csv`も3列で構成されており、`table5`の`col1`、`col2`、`col3`に順番にマッピングされます。

Kerberos認証が構成され、キータブファイルパスが指定された状態で`example5.csv`からのすべてのデータを`table5`にロードしたい場合は、次のコマンドを実行します：

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

#### データロードの取り消し

StarRocksデータベース`test_db`には`table6`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3列で構成されています。

データファイル`example6.csv`も3列で構成されており、`table6`の`col1`、`col2`、`col3`に順番にマッピングされます。

Broker Loadジョブを実行して`example6.csv`からのすべてのデータを`table6`にロードしました。

ロードしたデータを取り消したい場合は、次のコマンドを実行します：

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

StarRocksデータベース`test_db`には`table7`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3列で構成されています。

データファイル`example7.csv`も3列で構成されており、`table7`の`col1`、`col2`、`col3`に順番にマッピングされます。

`example7.csv`からのすべてのデータを`table7`の2つのパーティション`p1`と`p2`にロードしたい場合は、次のコマンドを実行します：

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

StarRocksデータベース`test_db`には`table8`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3列で構成されています。

データファイル`example8.csv`も3列で構成されており、`table8`の`col2`、`col1`、`col3`に順番にマッピングされます。

`example8.csv`からのすべてのデータを`table8`にロードしたい場合は、次のコマンドを実行します：

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

> **注記**
>
> 上記の例では、`example8.csv`の列を`table8`の列と同じ順序でマッピングすることはできません。したがって、`example8.csv`と`table8`の間の列マッピングを設定するためには、`column_list`を使用する必要があります。

#### フィルタ条件の設定

StarRocksデータベース`test_db`には`table9`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3列で構成されています。
また、データファイル `example9.csv` は3つの列で構成され、それぞれ `col1`, `col2`, `col3` に順番にマッピングされます `table9`。

最初の列の値が `20180601` より大きいデータレコードのみを `example9.csv` から `table9` にロードする場合は、次のコマンドを実行します。

```SQL
LOAD LABEL test_db.label9
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example9.csv")
    INTO TABLE table9
    (col1, col2, col3)
    WHERE col1 > 20180601
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

> **注記**
>
> 上記の例では、`example9.csv` の列は `table9` の列と同じ順序でマッピングできますが、列ベースのフィルター条件を指定するには WHERE 句を使用する必要があります。したがって、`example9.csv` と `table9` の間のカラムマッピングを構成するには `column_list` を使用する必要があります。

#### HLL型の列を含むテーブルへのデータロード

StarRocks データベース `test_db` には `table10` という名前のテーブルが含まれています。このテーブルは、`id`, `col1`, `col2`, `col3` の順に4つの列で構成されています。`col1` と `col2` はHLL型の列として定義されています。

データファイル `example10.csv` は3つの列で構成されており、最初の列は `table10` の `id` に、2番目と3番目の列は `col1` と `col2` に順番にマッピングされます。`example10.csv` の2番目と3番目の列の値は、`table10` の `col1` と `col2` にロードされる前に関数を使用してHLL型データに変換できます。

`example10.csv` から `table10` へすべてのデータをロードする場合は、次のコマンドを実行します。

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
        col3 = hll_empty()
     )
 )
 WITH BROKER
 (
     "username" = "<hdfs_username>",
     "password" = "<hdfs_password>"
 );
```

> **注記**
>
> 上記の例では、`example10.csv` の3つの列は `column_list` を使用して順番に `id`, `temp1`, `temp2` と名付けられています。次に、関数を使用してデータを以下のように変換します：
>
> - `hll_hash` 関数は、`example10.csv` の `temp1` と `temp2` の値をHLL型データに変換し、`table10` の `col1` と `col2` にマッピングするために使用されます。
>
> - `hll_empty` 関数は、`table10` の `col3` に指定されたデフォルト値を入力するために使用されます。

関数 `hll_hash` と `hll_empty` の使用方法については、[hll_hash](../../sql-functions/aggregate-functions/hll_hash.md) と [hll_empty](../../sql-functions/aggregate-functions/hll_empty.md) を参照してください。

#### ファイルパスからパーティションフィールドの値を抽出する

Broker Load は、宛先 StarRocks テーブルの列定義に基づいて、ファイルパスに含まれる特定のパーティションフィールドの値を解析することをサポートしています。StarRocks のこの機能は、Apache Spark™ のパーティションディスカバリ機能に似ています。

StarRocks データベース `test_db` には `table11` という名前のテーブルが含まれています。このテーブルは、`col1`, `col2`, `col3`, `city`, `utc_date` の順に5つの列で構成されています。

HDFSクラスターのファイルパス `/user/starrocks/data/input/dir/city=beijing` には、以下のデータファイルが含まれています：

- `/user/starrocks/data/input/dir/city=beijing/utc_date=2019-06-26/0000.csv`
- `/user/starrocks/data/input/dir/city=beijing/utc_date=2019-06-26/0001.csv`

これらのデータファイルはそれぞれ3つの列で構成され、`col1`, `col2`, `col3` に順番にマッピングされます `table11`。

ファイルパス `/user/starrocks/data/input/dir/city=beijing/utc_date=*/*` からすべてのデータファイルのデータを `table11` にロードし、同時にファイルパスに含まれるパーティションフィールド `city` と `utc_date` の値を抽出して `table11` の `city` と `utc_date` にロードする場合は、次のコマンドを実行します。

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

#### `%3A` を含むファイルパスからパーティションフィールドの値を抽出する

HDFSでは、ファイルパスにコロン (:) を含めることはできません。すべてのコロン (:) は `%3A` に変換されます。

StarRocks データベース `test_db` には `table12` という名前のテーブルが含まれています。このテーブルは、`data_time`, `col1`, `col2` の順に3つの列で構成されています。テーブルスキーマは以下の通りです：

```SQL
data_time DATETIME,
col1      INT,
col2      INT
```

HDFSクラスターのファイルパス `/user/starrocks/data` には、以下のデータファイルが含まれています：

- `/user/starrocks/data/data_time=2020-02-17 00%3A00%3A00/example12.csv`
- `/user/starrocks/data/data_time=2020-02-18 00%3A00%3A00/example12.csv`

`example12.csv` から `table12` へすべてのデータをロードし、同時にファイルパスからパーティションフィールド `data_time` の値を抽出して `table12` の `data_time` にロードする場合は、次のコマンドを実行します。

```SQL
LOAD LABEL test_db.label12
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/*/example12.csv")
    INTO TABLE table12
    COLUMNS TERMINATED BY ","
    FORMAT AS "csv"
    (col1, col2)
    COLUMNS FROM PATH AS (data_time)
    SET (data_time = str_to_date(data_time, '%Y-%m-%d %H%%3A%i%%3A%s'))
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

> **注記**
>
> 上記の例では、パーティションフィールド `data_time` から抽出された値は `%3A` を含む文字列、例えば `2020-02-17 00%3A00%3A00` です。したがって、`table12` の `data_time` にロードする前に、`str_to_date` 関数を使用して文字列をDATETIME型データに変換する必要があります。

#### `format_type_options` の設定

StarRocks データベース `test_db` には `table13` という名前のテーブルが含まれています。このテーブルは、`col1`, `col2`, `col3` の順に3つの列で構成されています。

データファイル `example13.csv` も3つの列で構成され、それぞれ `col2`, `col1`, `col3` に順番にマッピングされます `table13`。

`example13.csv` の最初の2行をスキップして、列セパレータの前後のスペースを削除し、`enclose` を `"` と `escape` を `\` に設定して `table13` にすべてのデータをロードする場合は、次のコマンドを実行します。

```SQL
LOAD LABEL test_db.label13
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/*/example13.csv")
    INTO TABLE table13
    COLUMNS TERMINATED BY ","
    FORMAT AS "CSV"
    (
        skip_header = 2,
        trim_space = TRUE,
        enclose = "\"",
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

### Parquetデータのロード

このセクションでは、Parquetデータをロードする際に注意が必要ないくつかのパラメータ設定について説明します。

StarRocks データベース `test_db` には `table13` という名前のテーブルが含まれています。このテーブルは、`col1`, `col2`, `col3` の順に3つの列で構成されています。

データファイル `example13.parquet` も3つの列で構成され、それぞれ `col1`, `col2`, `col3` に順番にマッピングされます `table13`。

`example13.parquet` から `table13` へすべてのデータをロードする場合は、次のコマンドを実行します。

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

> **注記**
>
> デフォルトでは、Parquet データを読み込む際に、StarRocks はファイル名に拡張子 **.parquet** が含まれているかどうかに基づいてデータファイル形式を決定します。ファイル名に拡張子 **.parquet** が含まれていない場合は、`FORMAT AS` を使用してデータファイル形式を `Parquet` として指定する必要があります。

### ORC データの読み込み

このセクションでは、ORC データをロードする際に注意すべきいくつかのパラメータ設定について説明します。

StarRocks データベース `test_db` には `table14` という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3` の3つの列で構成されています。

また、データファイル `example14.orc` にも3つの列が含まれており、これらは `table14` の `col1`、`col2`、`col3` に順番にマッピングされます。

`example14.orc` から `table14` へすべてのデータをロードしたい場合は、次のコマンドを実行します。

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

> **注記**
>
> - デフォルトでは、ORC データをロードする際に、StarRocks はファイル名に拡張子 **.orc** が含まれているかどうかに基づいてデータファイル形式を決定します。ファイル名に拡張子 **.orc** が含まれていない場合は、`FORMAT AS` を使用してデータファイル形式を `ORC` として指定する必要があります。
>
> - StarRocks v2.3 以前では、データファイルに ARRAY 型の列が含まれている場合、ORC データファイルの列の名前が StarRocks テーブルのマッピング列と同じであること、および SET 句で列を指定することはできないことを確認する必要があります。

