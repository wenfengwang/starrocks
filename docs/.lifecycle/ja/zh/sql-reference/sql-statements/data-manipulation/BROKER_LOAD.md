---
displayed_sidebar: Chinese
---

# ブローカーの負荷

## 機能

Broker LoadはMySQLプロトコルに基づく非同期インポート方法です。インポートジョブを提出すると、StarRocksは非同期でジョブを実行します。ブローカーロードジョブの結果を表示するには、`SELECT * FROM information_schema.loads`を使用します。この機能は3.1バージョンからサポートされています。Broker Loadの背景情報、基本原理、サポートされているデータファイル形式、シングルテーブルロードとマルチテーブルロードの実行方法、およびインポートジョブの結果を表示する方法については、[HDFSからのインポート](../../../loading/hdfs_load.md)と[クラウドストレージからのインポート](../../../loading/cloud_storage_load.md)を参照してください。

> **注意**
>
> Broker Load操作には、ターゲットテーブルのINSERT権限が必要です。ユーザーアカウントにINSERT権限がない場合は、[GRANT](../account-management/GRANT.md)を参照してユーザーに権限を付与してください。

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

StarRocksでは、一部のテキストはSQL言語の予約語であり、直接SQLステートメントで使用することはできません。これらの予約語をSQLステートメントで使用する場合は、バッククォート (`) で囲む必要があります。[キーワード](../keywords.md)を参照してください。

## パラメータの説明

### database_nameとlabel_name

`label_name`はインポートジョブのラベルを指定します。

`database_name`はオプションで、ターゲットのStarRocksテーブルが存在するデータベースを指定します。

各インポートジョブには、データベース内で一意のラベルが対応しています。ラベルを使用すると、対応するインポートジョブの実行状況を確認し、同じデータのインポートを防止することができます。インポートジョブのステータスが**FINISHED**の場合、そのラベルは他のインポートジョブに再利用できません。インポートジョブのステータスが**CANCELLED**の場合、そのラベルは他のインポートジョブに再利用できますが、通常は同じデータをインポートするために同じラベルを使用してインポートジョブを再試行します。

ラベルの命名規則については、[システム制限](../../../reference/System_limit.md)を参照してください。

### data_desc

インポートするデータのバッチを記述するために使用されます。各`data_desc`は、インポートするデータのソースアドレス、ETL関数、StarRocksテーブル、およびパーティションなどの情報を宣言します。

Broker Loadは、複数のデータファイルを一度にインポートすることができます。インポートジョブでは、複数の`data_desc`を使用して複数のデータファイルをインポートするか、1つの`data_desc`を使用して特定のパスのすべてのデータファイルをインポートすることができます。また、Broker Loadは、一度のインポートトランザクションのアトミック性を保証するため、一度のインポートで複数のデータファイルがすべて成功するか、すべて失敗するか、一部のインポートが成功し、一部のインポートが失敗することはありません。

`data_desc`の構文は次のとおりです。

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

`data_desc`の必須パラメータは次のとおりです。

- `file_path`

  ソースデータファイルのパスを指定します。

  特定のデータファイルをインポートする場合は、具体的なデータファイルを指定できます。たとえば、`"hdfs://<hdfs_host>:<hdfs_port>/user/data/tablename/20210411"`を指定することで、HDFSサーバー上の`/user/data/tablename`ディレクトリにある`20210411`という名前のデータファイルに一致します。

  ワイルドカードを使用して特定のパスのすべてのデータファイルをインポートすることもできます。Broker Loadは、次のワイルドカードをサポートしています：`?`、`*`、`[]`、`{}`、および`^`。詳細については、[ワイルドカードの使用ルールの参照](https://hadoop.apache.org/docs/stable/api/org/apache/hadoop/fs/FileSystem.html#globStatus-org.apache.hadoop.fs.Path-)を参照してください。たとえば、`"hdfs://<hdfs_host>:<hdfs_port>/user/data/tablename/*/*"`パスは、HDFSサーバー上の`/user/data/tablename`ディレクトリ内のすべてのパーティションのデータファイルに一致します。また、`"hdfs://<hdfs_host>:<hdfs_port>/user/data/tablename/dt=202104*/*"`パスは、HDFSサーバー上の`/user/data/tablename`ディレクトリ内のすべての`202104`パーティションのデータファイルに一致します。

  > **注意**
  >
  > - Broker Loadは、S3またはS3Aプロトコルを使用してAWS S3にアクセスすることができます。したがって、AWS S3からデータをインポートする場合は、ファイルパスに指定するターゲットファイルのS3 URIに`"s3://"`または`"s3a://"`を使用することができます。
  > - Broker Loadは、gsプロトコルを使用してGoogle GCSにアクセスすることができます。したがって、Google GCSからデータをインポートする場合は、ファイルパスに指定するターゲットファイルのGCS URIに`"gs://"`を使用する必要があります。
  > - Blob Storageからデータをインポートする場合は、wasbまたはwasbsをファイルプロトコルとして使用する必要があります：
  >   - ストレージアカウントがHTTPプロトコルでアクセス可能な場合は、wasbファイルプロトコルを使用し、ファイルパスの形式は`"wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*"`です。
  >   - ストレージアカウントがHTTPSプロトコルでアクセス可能な場合は、wasbsファイルプロトコルを使用し、ファイルパスの形式は`"wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*"`です。
  > - Azure Data Lake Storage Gen1からデータをインポートする場合は、adlをファイルプロトコルとして使用し、ファイルパスの形式は`"adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>"`です。
  > - Data Lake Storage Gen2からデータをインポートする場合は、abfsまたはabfssをファイルプロトコルとして使用する必要があります：
  >   - ストレージアカウントがHTTPプロトコルでアクセス可能な場合は、abfsファイルプロトコルを使用し、ファイルパスの形式は`"abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>"`です。
  >   - ストレージアカウントがHTTPSプロトコルでアクセス可能な場合は、abfssファイルプロトコルを使用し、ファイルパスの形式は`"abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>"`です。

- `INTO TABLE`

  ターゲットのStarRocksテーブルの名前を指定します。

`data_desc`のオプションパラメータは次のとおりです。

- `NEGATIVE`

  成功したインポートを取り消すために使用されます。成功したインポートを取り消す場合は、同じデータをインポートするために`NEGATIVE`キーワードを指定することができます。

  > **注意**
  >
  > このパラメータは、ターゲットのStarRocksテーブルが集計モデルを使用しており、すべてのValue列の集計関数が`sum`である場合にのみ適用されます。

- `PARTITION`

  データをインポートするパーティションを指定します。このパラメータを指定しない場合、デフォルトでStarRocksテーブルが存在するすべてのパーティションにインポートされます。

- `TEMPORARY_PARTITION`

  [一時パーティション](../../../table_design/Temporary_partition.md)にデータをインポートするパーティションを指定します。

- `COLUMNS TERMINATED BY`

  ソースデータファイルの列区切り記号を指定します。このパラメータを指定しない場合、デフォルトの列区切り記号は`\t`、つまりタブです。指定した列区切り記号とソースデータファイルの列区切り記号が一致していることを確認する必要があります。そうでない場合、データ品質エラーによりインポートジョブが失敗し、ジョブのステータス（`State`）が`CANCELLED`と表示されます。

  注意点として、Broker LoadはMySQLプロトコルを使用してインポートリクエストを送信しますが、StarRocksはエスケープ処理を行います。そのため、列区切り記号がタブなどの非表示文字の場合、列区切り文字の前にバックスラッシュ (\\) を追加する必要があります。たとえば、列区切り記号が`\t`の場合、`\\t`を入力する必要があります。列区切り記号が`\n`の場合、`\\n`を入力する必要があります。Apache Hive™のファイルの列区切り記号は`\x01`ですので、ソースデータファイルがHiveファイルの場合は、`\\x01`を指定する必要があります。

  > **注意**
  >
  > - StarRocksは、最大50バイトのUTF-8エンコード文字列を列区切り記号として設定することができます。一般的なカンマ(,)、タブ、パイプ(|)などが含まれます。
  > - NULL値は`\N`で表されます。たとえば、データファイルには3つの列があり、ある行の1番目の列と3番目の列のデータがそれぞれ`a`と`b`であり、2番目の列にデータがない場合、2番目の列はNULL値を表すために`\N`を使用する必要があります。つまり、`a,\N,b`と書く必要があります。`a,,b`ではなく、`a,,b`は2番目の列が空の文字列を表します。

- `ROWS TERMINATED BY`

  ソースデータファイルの行区切り記号を指定します。このパラメータを指定しない場合、デフォルトの行区切り記号は`\n`、つまり改行です。指定した行区切り記号とソースデータファイルの行区切り記号が一致していることを確認する必要があります。そうでない場合、データ品質エラーによりインポートジョブが失敗し、ジョブのステータス（`State`）が`CANCELLED`と表示されます。このパラメータは2.5.4バージョンからサポートされています。

  その他の注意事項と使用条件は、列区切り記号を指定する際に説明したものと同じです。

- `FORMAT AS`

  ソースデータファイルの形式を指定します。`CSV`、`Parquet`、`ORC`のいずれかの値を取ります。このパラメータを指定しない場合、ファイルパスパラメータで指定されたファイル拡張子（**.csv**、**.parquet**、および**.orc**）に基づいてファイル形式が判断されます。

- `format_type_options`

  `FORMAT AS`が`CSV`に設定されている場合、CSV形式のオプションを指定するために使用されます。構文は次のとおりです。

  ```JSON
  (
      key = value
      key = value
      ...
  )
  ```
  
  > **注意**
  >
  > `format_type_options`は3.0以降のバージョンでサポートされています。

  オプションの説明は以下の表を参照してください。
  
  | **パラメータ** | **説明**                                                     |
  | -------------- | ------------------------------------------------------------ |
  | skip_header    | CSVファイルの先頭の行をスキップするかどうかを指定します。値のタイプはINTEGERです。デフォルト値は`0`です。<br />一部のCSVファイルでは、先頭の行に列名や列の型などのメタデータ情報が含まれています。このパラメータを設定することで、StarRocksはデータをインポートする際にCSVファイルの先頭の行を無視することができます。たとえば、このパラメータを`1`に設定すると、StarRocksはデータをインポートする際にCSVファイルの最初の行を無視します。<br />ここでの行の区切り記号は、インポートステートメントで設定した行の区切り記号と一致している必要があります。 |

| trim_space | CSVファイル内の列区切り記号の前後の空白を削除するかどうかを指定します。値のタイプ：BOOLEAN。デフォルト値：`false`。<br />一部のデータベースは、CSVファイルにデータをエクスポートする際に、列区切り記号の前後にいくつかの空白を追加します。これらの空白は、位置によって「先行空白」と「後続空白」と呼ばれることがあります。このパラメータを設定することで、StarRocksはこれらの不要な空白を削除してデータをインポートすることができます。<br />なお、StarRocksは、`enclose`で指定された文字で囲まれたフィールド内の空白（先行空白と後続空白を含む）は削除しません。例えば、列区切り記号が縦棒（<code class="language-text">&#124;</code>）で、`enclose`で指定された文字がダブルクオート（`"`）の場合：<br /><code class="language-text">&#124;"スターロックスが大好きです」&#124;</code> <br /><code class="language-text">&#124;"愛StarRocks"&#124;</code> <br /><code class="language-text">&#124;"ラブスターロックス" &#124;</code> <br />`trim_space`を`true`に設定すると、StarRocksの処理結果は次のようになります：<br /><code class="language-text">&#124;"スターロックスが大好きです」&#124;</code> <br /><code class="language-text">&#124;"愛StarRocks"&#124;</code> <br /><code class="language-text">&#124;"スターロックスが大好きです」&#124;</code> |
| enclose | [RFC4180](https://www.rfc-editor.org/rfc/rfc4180)に基づいて、CSVファイルのフィールドを囲む文字を指定します。値のタイプ：1バイト文字。デフォルト値：`NONE`。最も一般的な`enclose`文字はシングルクオート（`'`）またはダブルクオート（`"`）です。<br />`enclose`で指定された文字で囲まれたフィールド内のすべての特殊文字（行区切り記号、列区切り記号など）は通常の文字として扱われます。さらに、StarRocksの`enclose`属性では、任意の1バイト文字を設定することができます。<br />フィールド内に`enclose`で指定された文字が含まれる場合は、同じ文字を使用して`enclose`で指定された文字をエスケープすることができます。たとえば、`enclose`をダブルクオート（`"`）に設定した場合、フィールド値`a "quoted" c`はCSVファイルでは`"a ""quoted"" c"`と書かれる必要があります。 |
| escape | エスケープに使用する文字を指定します。行区切り記号、列区切り記号、エスケープ記号、`enclose`で指定された文字など、さまざまな特殊文字をエスケープして、StarRocksがこれらの特殊文字を通常の文字として解析し、フィールド値の一部として扱うことができるようにします。値のタイプ：1バイト文字。デフォルト値：`NONE`。最も一般的な`escape`文字はスラッシュ（`\`）で、SQL文ではダブルスラッシュ（`\\`）と書かれる必要があります。<br />**注意**<br />`escape`で指定された文字は、`enclose`で指定された文字の内部および外部の両方に適用されます。<br />以下に2つの例を示します：<ul><li>`enclose`をダブルクオート（`"`）に設定し、`escape`をスラッシュ（`\`）に設定した場合、StarRocksは`"say \"Hello world\""`をフィールド値`say "Hello world"`として解析します。</li><li>列区切り記号がカンマ（`,`）の場合、`escape`をスラッシュ（`\`）に設定した場合、StarRocksは`a, b\, c`を`a`と`b, c`の2つのフィールド値として解析します。</li></ul> |

- `column_list`

  ソースデータファイルとStarRocksテーブルの列の対応関係を指定します。構文は次のとおりです：`(<column_name>[, <column_name> ...])`。`column_list`で宣言された列は、StarRocksテーブルの列と名前が一致します。

  > **注意**
  >
  > ソースデータファイルの列とStarRocksテーブルの列が順番に一致する場合、`column_list`パラメータを指定する必要はありません。

  ソースデータファイルの特定の列をスキップする場合は、`column_list`パラメータでその列をStarRocksテーブルに存在しない列名として指定するだけです。詳細については、[データ変換の実現](../../../loading/Etl_in_loading.md)を参照してください。

- `COLUMNS FROM PATH AS`

  指定されたファイルパスから1つまたは複数のパーティションフィールドの情報を抽出するために使用されます。このパラメータは、指定されたファイルパスにパーティションフィールドが存在する場合にのみ有効です。

  たとえば、ソースデータファイルのパスが`/path/col_name=col_value/file1`であり、`col_name`がStarRocksテーブルの列に対応する場合、パラメータを`col_name`に設定できます。インポート時に、StarRocksは`col_value`を`col_name`に対応する列に落とします。

  > **注意**
  >
  > このパラメータは、データをHDFSからインポートする場合にのみ使用できます。

- `SET`

  ソースデータファイルの特定の列を指定された関数で変換し、変換後の結果をStarRocksテーブルに落とすために使用されます。構文は次のとおりです：`column_name = expression`。以下に2つの例を示します：

  - StarRocksテーブルには3つの列があり、順番に`col1`、`col2`、`col3`です。ソースデータファイルには4つの列があり、最初の2つの列が順番にStarRocksテーブルの`col1`、`col2`列に対応し、後の2つの列の合計がStarRocksテーブルの`col3`列に対応します。この場合、`column_list`パラメータで`(col1,col2,tmp_col3,tmp_col4)`を宣言し、`SET (col3=tmp_col3+tmp_col4)`を使用してデータ変換を実現する必要があります。
  - StarRocksテーブルには3つの列があり、順番に`year`、`month`、`day`です。ソースデータファイルには、`yyyy-mm-dd hh:mm:ss`形式の時間データが1つだけあります。この場合、`column_list`パラメータで`(tmp_time)`を宣言し、`SET (year = year(tmp_time), month=month(tmp_time), day=day(tmp_time))`を使用してデータ変換を実現する必要があります。

- `WHERE`

  フィルタ条件を指定して、変換後のデータをフィルタリングします。WHERE句で指定されたフィルタ条件に一致するデータのみがStarRocksテーブルにインポートされます。

### WITH BROKER

v2.3以前では、インポートステートメントで`WITH BROKER "<broker_name>"`を使用して、どのブローカーを使用するかを指定する必要がありました。v2.5以降、`broker_name`を指定する必要はありませんが、`WITH BROKER`キーワードは引き続き使用されます。

> **注意**
>
> v2.3以前では、StarRocksはブローカーを使用して外部ストレージシステムにアクセスするためにブローカーが必要でした。これを「ブローカーを使用したインポート」と呼びます。ブローカーは、ファイルシステムインターフェースをカプセル化した独立したステートレスサービスです。ブローカーを使用することで、StarRocksは外部ストレージシステム上のデータファイルにアクセスして読み取り、データファイル内のデータを前処理およびインポートするための計算リソースを利用することができます。
>
> v2.5以降、StarRocksはブローカーなしで外部ストレージシステムにアクセスできるため、「ブローカーなしのインポート」と呼ばれます。
>
> StarRocksクラスタにデプロイされているブローカーは、[SHOW BROKER](../Administration/SHOW_BROKER.md)ステートメントで確認できます。ブローカーがクラスタにデプロイされていない場合は、[ブローカーノードのデプロイ](../../../deployment/deploy_broker.md)を参照してブローカーをデプロイしてください。

### StorageCredentialParams

StarRocksがストレージシステムにアクセスするための認証設定です。

#### HDFS

コミュニティ版のHDFSは、シンプル認証とKerberos認証の2つの認証方式（ブローカーのロードではデフォルトでシンプル認証が使用されます）をサポートしており、NameNodeノードのHA構成もサポートしています。ストレージシステムがコミュニティ版のHDFSの場合、次のように認証方式とHA構成を指定できます：

- 認証方式

  - シンプル認証を使用する場合は、次のように`StorageCredentialParams`を設定します：

    ```Plain
    "hadoop.security.authentication" = "simple",
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
    ```

    `StorageCredentialParams`には次のパラメータが含まれます。

    | **パラメータ名**                  | **パラメータの説明**                         |
    | ------------------------------- | -------------------------------------------- |
    | hadoop.security.authentication  | 認証方式を指定します。値の範囲：`simple`、`kerberos`。デフォルト値：`simple`。`simple`はシンプル認証を表し、認証なしを意味します。`kerberos`はKerberos認証を表します。 |
    | ユーザー名                        | NameNodeノードにアクセスするためのHDFSクラスタのユーザー名です。 |
    | パスワード                        | NameNodeノードにアクセスするためのHDFSクラスタのパスワードです。   |

  - Kerberos認証を使用する場合は、次のように`StorageCredentialParams`を設定します：

    ```Plain
    "hadoop.security.authentication" = "kerberos",
    "kerberos_principal" = "nn/zelda1@ZELDA.COM",
    "kerberos_keytab" = "/keytab/hive.keytab",
    "kerberos_keytab_content" = "YWFhYWFh"
    ```

    `StorageCredentialParams`には次のパラメータが含まれます。

    | **パラメータ名**                     | **パラメータの説明**                                                 |
    | ------------------------------- | ------------------------------------------------------------ |
    | hadoop.security.authentication  | 認証方式を指定します。値の範囲：`simple`、`kerberos`。デフォルト値：`simple`。`simple`はシンプル認証を表し、認証なしを意味します。`kerberos`はKerberos認証を表します。 |
    | kerberos_principal              | Kerberosのユーザーまたはサービス（Principal）を指定します。各PrincipalはHDFSクラスタ内で一意であり、次の3つの部分で構成されます：<ul><li>`username`または`servicename`：HDFSクラスタ内のユーザーまたはサービスの名前。</li><li>`instance`：ユーザーまたはサービスが認証されるノードが存在するサーバーの名前で、ユーザーまたはサービスがグローバルに一意であることを保証します。たとえば、HDFSクラスタには複数のDataNodeノードがあり、各ノードは独自の認証を必要とします。</li><li>`realm`：ドメイン、すべて大文字である必要があります。</li></ul>例：`nn/zelda1@ZELDA.COM`。 |
    | kerberos_keytab                 | Kerberosのキータブ（略して「keytab」）ファイルのパスを指定します。 |
    | kerberos_keytab_content         | Kerberosのkeytabファイルの内容をBase64エンコードした後の内容を指定します。このパラメータは`kerberos_keytab`パラメータと二者択一で設定します。 |

    注意点として、複数のKerberosユーザーが存在する場合、少なくとも1組の独立した[ブローカー](../../../deployment/deploy_broker.md)をデプロイし、インポートステートメントで`WITH BROKER "<broker_name>"`を使用してどのブローカーを使用するかを指定する必要があります。また、ブローカープロセスの起動スクリプトファイル**start_broker.sh**を開き、ファイルの42行付近で次の情報を変更して、ブローカープロセスが**krb5.conf**ファイルの情報を読み取るようにします：

    ```Plain
    export JAVA_OPTS="-Dlog4j2.formatMsgNoLookups=true -Xmx1024m -Dfile.encoding=UTF-8 -Djava.security.krb5.conf=/etc/krb5.conf"
    ```

    > **注意**
    >
    > - **/etc/krb5.conf**のファイルパスは、環境に合わせて変更してください。ブローカーはこのファイルを読み取るためのアクセス権を持っている必要があります。複数のブローカーをデプロイする場合、各ブローカーノードで上記の情報を変更し、ブローカーノードを再起動して設定を有効にする必要があります。
    > - StarRocksクラスタにデプロイされているブローカーは、[SHOW BROKER](../Administration/SHOW_BROKER.md)ステートメントで確認できます。

- HA構成

```
  HDFS クラスターの NameNode ノードに HA 機構を設定して、NameNode ノードの切り替えが発生した場合に StarRocks が新しく切り替わった NameNode ノードを自動的に認識できるようにすることができます。以下のシナリオが含まれます：

  - 単一の HDFS クラスターで、単一の Kerberos ユーザーが設定されている場合、Broker を使用したインポートと Broker を使用しないインポートの両方が可能です。
  
    - Broker を使用したインポートを行う場合、少なくとも一組の独立した [Broker](../../../deployment/deploy_broker.md) をデプロイして、`hdfs-site.xml` ファイルを HDFS クラスターに対応する Broker ノードの `{deploy}/conf` ディレクトリに配置する必要があります。Broker プロセスが再起動すると、`{deploy}/conf` ディレクトリが `CLASSPATH` 環境変数に追加され、Broker が HDFS クラスター内の各ノードの情報を読み取ることができます。
  
    - Broker を使用しないインポートを行う場合、`hdfs-site.xml` ファイルを各 FE ノードおよび各 BE ノードの `{deploy}/conf` ディレクトリに配置する必要があります。

  - 単一の HDFS クラスターで、複数の Kerberos ユーザーが設定されている場合、Broker を使用したインポートのみがサポートされます。少なくとも一組の独立した [Broker](../../../deployment/deploy_broker.md) をデプロイして、`hdfs-site.xml` ファイルを HDFS クラスターに対応する Broker ノードの `{deploy}/conf` ディレクトリに配置する必要があります。Broker プロセスが再起動すると、`{deploy}/conf` ディレクトリが `CLASSPATH` 環境変数に追加され、Broker が HDFS クラスター内の各ノードの情報を読み取ることができます。

  - 複数の HDFS クラスターがある場合（単一の Kerberos ユーザーであっても、複数の Kerberos ユーザーであっても）、Broker を使用したインポートのみがサポートされます。少なくとも一組の独立した [Broker](../../../deployment/deploy_broker.md) をデプロイし、以下のいずれかの方法で Broker が HDFS クラスター内の各ノードの情報を読み取るように設定する必要があります：
  
    - `hdfs-site.xml` ファイルを各 HDFS クラスターに対応する Broker ノードの `{deploy}/conf` ディレクトリに配置します。Broker プロセスが再起動すると、`{deploy}/conf` ディレクトリが `CLASSPATH` 環境変数に追加され、Broker が HDFS クラスター内の各ノードの情報を読み取ることができます。
    - Broker Load ジョブを作成する際に、以下の HA 設定を追加します：

      ```Plain
      "dfs.nameservices" = "ha_cluster",
      "dfs.ha.namenodes.ha_cluster" = "ha_n1,ha_n2",
      "dfs.namenode.rpc-address.ha_cluster.ha_n1" = "<hdfs_host>:<hdfs_port>",
      "dfs.namenode.rpc-address.ha_cluster.ha_n2" = "<hdfs_host>:<hdfs_port>",
      "dfs.client.failover.proxy.provider" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
      ```

      上記の設定におけるパラメータの説明は以下の表の通りです：

      | **パラメータ名**                         | **説明**                                                 |
      | ------------------------------------------------------- | --------------------------- |
      | dfs.nameservices                  | HDFS クラスターの名前をカスタマイズします。                                     |
      | dfs.ha.namenodes.XXX              | NameNode の名前をカスタマイズします。名前はコンマ (,) で区切り、二重引用符の中にスペースを含めることはできません。<br />ここでの `XXX` は `dfs.nameservices` でカスタマイズされた HDFS サービスの名前です。 |
      | dfs.namenode.rpc-address.XXX.NN    | NameNode の RPC アドレス情報を指定します。<br />`NN` は `dfs.ha.namenodes.XXX` でカスタマイズされた NameNode の名前を表します。 |
      | dfs.client.failover.proxy.provider | クライアントが接続する NameNode のプロバイダーを指定します。デフォルトは `org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider` です。 |

  > **説明**
  >
  > StarRocks クラスターにデプロイされている Broker は [SHOW BROKER](../Administration/SHOW_BROKER.md) ステートメントで確認できます。

#### AWS S3

ストレージシステムが AWS S3 の場合、以下のように `StorageCredentialParams` を設定してください：

- Instance Profile を使用した認証と認可

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- Assumed Role を使用した認証と認可

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAM User を使用した認証と認可

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

`StorageCredentialParams` には以下のパラメータが含まれます。

| パラメータ                        | 必須   | 説明                                                         |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | はい       | Instance Profile と Assumed Role の認証方式を指定します。値の範囲は `true` または `false`。デフォルトは `false`。 |
| aws.s3.iam_role_arn         | いいえ       | AWS S3 Bucket へのアクセス権を持つ IAM Role の ARN。Assumed Role 認証方式を使用する場合は、このパラメータを指定する必要があります。 |
| aws.s3.region               | はい       | AWS S3 Bucket が存在するリージョン。例：`us-west-1`。                |
| aws.s3.access_key           | いいえ       | IAM User の Access Key。IAM User 認証方式を使用する場合は、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | いいえ       | IAM User の Secret Key。IAM User 認証方式を使用する場合は、このパラメータを指定する必要があります。 |

AWS S3 へのアクセスに使用する認証方式の選択方法や、AWS IAM コンソールでアクセス制御ポリシーを設定する方法については、[AWS S3 への認証パラメータ](../../../integrations/authenticate_to_aws_resources.md#访问-aws-s3-的认证参数)を参照してください。

#### Google GCS

ストレージシステムが Google GCS の場合、以下のように `StorageCredentialParams` を設定してください：

- VM を使用した認証と認可

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | パラメータ                                   | デフォルト値 | 取りうる値 | 説明                                                 |
  | ------------------------------------------ | ---------- | ------------ | -------------------------------------------------------- |
  | gcp.gcs.use_compute_engine_service_account | false      | true         | Compute Engine に紐付けられた Service Account を直接使用するかどうか。 |

- Service Account を使用した認証と認可

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | パラメータ                               | デフォルト値 | 取りうる値                                                 | 説明                                                     |
  | -------------------------------------- | ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""         | "user@hello.iam.gserviceaccount.com" | Service Account を作成した際に生成された JSON ファイル内の Email。          |
  | gcp.gcs.service_account_private_key_id | ""         | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | Service Account を作成した際に生成された JSON ファイル内の Private Key ID。 |
  | gcp.gcs.service_account_private_key    | ""         | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | Service Account を作成した際に生成された JSON ファイル内の Private Key。    |

- Impersonation を使用した認証と認可

  - VM インスタンスで Service Account を模倣

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    `StorageCredentialParams` には以下のパラメータが含まれます。

    | パラメータ                                   | デフォルト値 | 取りうる値 | 説明                                                     |
    | ------------------------------------------ | ---------- | ------------ | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false      | true         | Compute Engine に紐付けられた Service Account を直接使用するかどうか。     |
    | gcp.gcs.impersonation_service_account      | ""         | "hello"      | 模倣する対象の Service Account。 |

  - ある Service Account（「Meta Service Account」と呼ばれる）を使用して、別の Service Account（「Data Service Account」と呼ばれる）を模倣

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    `StorageCredentialParams` には以下のパラメータが含まれます。

    | パラメータ                               | デフォルト値 | 取りうる値                                                 | 説明                                                     |
    | -------------------------------------- | ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""         | "user@hello.iam.gserviceaccount.com" | Meta Service Account を作成した際に生成された JSON ファイル内の Email。     |
    | gcp.gcs.service_account_private_key_id | ""         | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | Meta Service Account を作成した際に生成された JSON ファイル内の Private Key ID。 |
    | gcp.gcs.service_account_private_key    | ""         | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | Meta Service Account を作成した際に生成された JSON ファイル内の Private Key。 |
    | gcp.gcs.impersonation_service_account  | ""         | "hello"                                                      | 模倣する対象の Data Service Account。 |

#### 阿里云 OSS

ストレージシステムが阿里云 OSS の場合、以下のように `StorageCredentialParams` を設定してください：

```SQL
"fs.oss.accessKeyId" = "<oss_access_key>",
"fs.oss.accessKeySecret" = "<oss_secret_key>",
"fs.oss.endpoint" = "<oss_endpoint>"
```

`StorageCredentialParams` には以下のパラメータが含まれます。

| パラメータ名           | 説明                                                 |
| ---------------------- | ------------------------------------------------------------ |
| fs.oss.accessKeyId     | 阿里云 OSS のストレージスペースにアクセスするための AccessKey ID で、ユーザーを識別するために使用されます。           |
| fs.oss.accessKeySecret | 阿里云 OSS のストレージスペースにアクセスするための AccessKey Secret で、署名文字列を暗号化し、OSS が署名文字列を検証するためのキーです。 |
| fs.oss.endpoint        | 阿里云 OSS のストレージスペースにアクセスするためのエンドポイントです。                              |

阿里云の公式ドキュメント[ユーザー署名検証](https://help.aliyun.com/document_detail/31950.html)を参照してください。

#### 腾讯云 COS

ストレージシステムが腾讯云 COS の場合、以下のように `StorageCredentialParams` を設定してください：

```SQL
"fs.cosn.userinfo.secretId" = "<cos_access_key>",
"fs.cosn.userinfo.secretKey" = "<cos_secret_key>",
"fs.cosn.bucket.endpoint_suffix" = "<cos_endpoint>"
```

`StorageCredentialParams` には以下のパラメータが含まれます。

| パラメータ名                   | 説明                                                 |
| ------------------------------ | ------------------------------------------------------------ |
| fs.cosn.userinfo.secretId      | 腾讯云 COS のストレージスペースにアクセスするための SecretId で、API コールの身元を識別するために使用されます。  |

| fs.cosn.userinfo.secretKey     | 腾讯云 COS ストレージスペースへのアクセスに使用される SecretKey は、署名文字列の暗号化とサーバー側での署名文字列の検証に使用されるキーです。 |
| fs.cosn.bucket.endpoint_suffix | 腾讯云 COS ストレージスペースへの接続アドレスです。                              |

腾讯云公式ドキュメント[永続キーを使用して COS にアクセスする](https://cloud.tencent.com/document/product/436/68282)を参照してください。

#### 华为云 OBS

ストレージシステムが华为云 OBSの場合、次のように `StorageCredentialParams` を設定します：

```SQL
"fs.obs.access.key" = "<obs_access_key>",
"fs.obs.secret.key" = "<obs_secret_key>",
"fs.obs.endpoint" = "<obs_endpoint>"
```

`StorageCredentialParams` には、次のパラメータが含まれます。

| **パラメータ名**           | **パラメータの説明**                                                 |
| ---------------------- | ------------------------------------------------------------ |
| fs.obs.access.key      | 华为云 OBS ストレージスペースへのアクセスキー ID は、プライベートアクセスキーと関連付けられた一意の識別子です。|
| fs.obs.secret.key      | 华为云 OBS ストレージスペースへのシークレットアクセスキーは、リクエストを暗号化して署名し、送信元を識別し、リクエストが変更されるのを防ぐために使用されます。 |
| fs.obs.endpoint        | 华为云 OBS ストレージスペースへの接続アドレスです。                              |

华为云公式ドキュメント[永続アクセスキーを使用して OBS にアクセスする](https://support.huaweicloud.com/perms-cfg-obs/obs_40_0007.html)を参照してください。

> **注意**
>
> 华为云 OBS からデータをインポートする場合は、まず[依存ライブラリ](https://github.com/huaweicloud/obsa-hdfs/releases/download/v45/hadoop-huaweicloud-2.8.3-hw-45.jar)をダウンロードして **$BROKER_HOME/lib/** パスに追加し、Broker を再起動する必要があります。

#### その他の S3 互換オブジェクトストレージ

ストレージシステムが他の S3 互換オブジェクトストレージ（例：MinIO）の場合、次のように `StorageCredentialParams` を設定します：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

`StorageCredentialParams` には、次のパラメータが含まれます。

| パラメータ                            | 必須 | 説明                                                         |
| ------------------------------- | ---- | ------------------------------------------------------------ |
| aws.s3.enable_ssl               | Yes  | SSL 接続を有効にするかどうか。値の範囲：`true` または `false`。デフォルト値：`true`。MinIO の場合、`true` に設定する必要があります。 |
| aws.s3.enable_path_style_access | Yes  | パス形式の URL アクセスを有効にするかどうか。値の範囲：`true` または `false`。デフォルト値：`false`。 |
| aws.s3.endpoint                 | Yes  | S3 互換オブジェクトストレージへのエンドポイント。                         |
| aws.s3.access_key               | Yes  | IAM ユーザーのアクセスキー。                                     |
| aws.s3.secret_key               | Yes  | IAM ユーザーのシークレットキー。                                     |

#### Microsoft Azure ストレージ

##### Azure Blob Storage

ストレージシステムが Blob Storage の場合、次のように `StorageCredentialParams` を設定します：

- 共有キーによる認証と認可

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  `StorageCredentialParams` には、次のパラメータが含まれます。

  | パラメータ                   | 必須 | 説明                         |
  | -------------------------- | ---- | ---------------------------- |
  | azure.blob.storage_account | Yes  | Blob Storage アカウントのユーザー名。      |
  | azure.blob.shared_key      | Yes  | Blob Storage アカウントの共有キー。 |

- SAS トークンによる認証と認可

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.container" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  `StorageCredentialParams` には、次のパラメータが含まれます。

  | パラメータ                  | 必須 | 説明                                 |
  | ------------------------- | ---- | ------------------------------------ |
  | azure.blob.storage_account| Yes  | Blob Storage アカウントのユーザー名。              |
  | azure.blob.container      | Yes  | データを格納する Blob コンテナの名前。         |
  | azure.blob.sas_token      | Yes  | Blob Storage アカウントにアクセスするための SAS トークン。 |

##### Azure Data Lake Storage Gen1

ストレージシステムが Data Lake Storage Gen1 の場合、次のように `StorageCredentialParams` を設定します：

- マネージドサービスアイデンティティによる認証と認可

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  `StorageCredentialParams` には、次のパラメータが含まれます。

  | パラメータ                                 | 必須 | 説明                                                     |
  | ---------------------------------------- | ---- | -------------------------------------------------------- |
  | azure.adls1.use_managed_service_identity | Yes  | マネージドサービスアイデンティティ認証方式を有効にするかどうかを指定します。`true` に設定します。 |

- サービスプリンシパルによる認証と認可

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  `StorageCredentialParams` には、次のパラメータが含まれます。

  | パラメータ                      | 必須 | 説明                                                     |
  | ----------------------------- | ---- | -------------------------------------------------------- |
  | azure.adls1.oauth2_client_id  | Yes  | サービスプリンシパルのクライアント (アプリケーション) ID。               |
  | azure.adls1.oauth2_credential | Yes  | 新しいクライアント (アプリケーション) シークレット。                           |
  | azure.adls1.oauth2_endpoint   | Yes  | サービスプリンシパルまたはアプリケーションの OAuth 2.0 トークンエンドポイント (v1)。 |

##### Azure Data Lake Storage Gen2

ストレージシステムが Data Lake Storage Gen2 の場合、次のように `StorageCredentialParams` を設定します：

- マネージドアイデンティティによる認証と認可

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  `StorageCredentialParams` には、次のパラメータが含まれます。

  | パラメータ                                | 必須 | 説明                                                |
  | --------------------------------------- | ---- | --------------------------------------------------- |
  | azure.adls2.oauth2_use_managed_identity | Yes  | マネージドアイデンティティ認証方式を有効にするかどうかを指定します。`true` に設定します。 |
  | azure.adls2.oauth2_tenant_id            | Yes  | データが所属するテナントの ID。                       |
  | azure.adls2.oauth2_client_id            | Yes  | マネージドアイデンティティのクライアント (アプリケーション) ID。          |

- 共有キーによる認証と認可

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  `StorageCredentialParams` には、次のパラメータが含まれます。

  | パラメータ                    | 必須 | 説明                                   |
  | --------------------------- | ---- | -------------------------------------- |
  | azure.adls2.storage_account | Yes  | Data Lake Storage Gen2 アカウントのユーザー名。      |
  | azure.adls2.shared_key      | Yes  | Data Lake Storage Gen2 アカウントの共有キー。 |

- サービスプリンシパルによる認証と認可

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  `StorageCredentialParams` には、次のパラメータが含まれます。

  | パラメータ                           | 必須 | 説明                                                |
  | ---------------------------------- | ---- | --------------------------------------------------- |
  | azure.adls2.oauth2_client_id       | Yes  | サービスプリンシパルのクライアント (アプリケーション) ID。          |
  | azure.adls2.oauth2_client_secret   | Yes  | 新しいクライアント (アプリケーション) シークレット。                    |
  | azure.adls2.oauth2_client_endpoint | Yes  | サービスプリンシパルまたはアプリケーションの OAuth 2.0 トークンエンドポイント (v1)。 |

### opt_properties

インポートに関連するいくつかのオプションパラメータを指定するために使用され、指定されたパラメータはインポートジョブ全体に適用されます。構文は次のとおりです：

```Plain
PROPERTIES ("<key1>" = "<value1>"[, "<key2>" = "<value2>" ...])
```

パラメータの説明は次のとおりです：

- `timeout`

  インポートジョブのタイムアウト時間。単位：秒。デフォルトのタイムアウト時間は4時間です。6時間未満のタイムアウト時間を推奨します。指定した時間内にインポートジョブが完了しない場合、自動的にキャンセルされ、**CANCELLED** 状態になります。

  > **注意**
  >
  > 通常、インポートジョブのタイムアウト時間を手動で設定する必要はありません。デフォルトのタイムアウト時間でインポートジョブが完了しない場合にのみ、インポートジョブのタイムアウト時間を手動で設定することをお勧めします。

  タイムアウト時間は、次の計算式の値よりも大きいことを推奨します：

  **タイムアウト時間 > (ソースデータファイルの総サイズ x ソースデータファイルと関連するマテリアライズドビューの数)/平均インポート速度**

    > **注意**
    >
    > "平均インポート速度" は、現在の StarRocks クラスタの平均インポート速度を指します。各 StarRocks クラスタのマシン環境が異なり、クラスタで許可される同時クエリタスクの数も異なるため、StarRocks クラスタの平均インポート速度は、過去のインポート速度に基づいて推定する必要があります。

   たとえば、1 GB のデータファイルをインポートする場合で、そのデータファイルには2つのマテリアライズドビューが含まれているとします。現在の StarRocks クラスタの平均インポート速度が10 MB/sである場合、次の計算式により、タイムアウト時間が102秒であることが推奨されます：

   **(1 x 1024 x 3)/10 = 307.2（秒）**

   したがって、インポートジョブのタイムアウト時間は308秒以上であることが推奨されます。

- `max_filter_ratio`

  インポートジョブの最大許容率は、データ品質が不合格でフィルタリングされることができるデータ行の割合の最大値です。値の範囲：`0`~`1`。デフォルト値：`0` 。

  - 最大許容率を `0` に設定すると、StarRocks はインポート中にエラーのあるデータ行を無視しません。データ行にエラーがある場合、インポートジョブは失敗し、データの正確性が保証されます。
  - 最大許容率を `0` より大きく設定すると、StarRocks はインポート中にエラーのあるデータ行を無視します。これにより、インポートされたデータ行にエラーがあっても、インポートジョブは成功します。
    > **注意**
    >
    > ここでフィルタリングされるデータ行は、WHERE 句でフィルタリングされたデータ行は含まれません。

  最大許容率を `0` に設定したことでジョブが失敗した場合、[SHOW LOAD](../data-manipulation/SHOW_LOAD.md) ステートメントを使用してインポートジョブの結果情報を確認できます。その後、エラーのあるデータ行がフィルタリングされるかどうかを判断できます。フィルタリングされる場合は、結果情報の `dpp.abnorm.ALL` と `dpp.norm.ALL` を使用して、インポートジョブの最大許容率を計算し、調整してからインポートジョブを再提出できます。計算式は次のとおりです：

  **`max_filter_ratio` = [`dpp.abnorm.ALL`/(`dpp.abnorm.ALL` + `dpp.norm.ALL`)]**

  `dpp.abnorm.ALL` と `dpp.norm.ALL` の合計は、インポートする行の合計数になります。

- `log_rejected_record_num`


  指定最多允许记录多少条因数据质量不合格而过滤掉的数据行数。该参数自 3.1 版本起支持。取值范围：`0`、`-1`、大于 0 的正整数。默认值：`0`。
  
  - 取值为 `0` 表示不记录过滤掉的数据行。
  - 取值为 `-1` 表示记录所有过滤掉的数据行。
  - 取值为大于 0 的正整数（比如 `n`）表示每个 BE 节点上最多可以记录 `n` 条过滤掉的数据行。

- `load_mem_limit`

  导入作业的内存限制，最大不超过 BE 的内存限制。单位：字节。默认内存限制为 2 GB。

- `strict_mode`

  是否开启严格模式。取值范围：`true` 和 `false`。默认值：`false`。`true` 表示开启，`false` 表示关闭。<br />关于该模式的介绍，请参见[严格模式](../../../loading/load_concept/strict_mode.md)。

- `timezone`

  指定导入作业所使用的时区。默认为 `Asia/Shanghai` 时区。该参数会影响所有导入涉及的、跟时区设置有关的函数所返回的结果。受时区影响的函数有 strftime、alignment_timestamp 和 from_unixtime 等，请参见[设置时区](../../../administration/timezone.md)。导入参数 `timezone` 设置的时区对应“[设置时区](../../../administration/timezone.md)”中所述的会话级时区。

- `priority`

   指定导入作业的优先级。取值范围：`LOWEST`、`LOW`、`NORMAL`、`HIGH` 和 `HIGHEST`。默认值：`NORMAL`。Broker Load 通过 [FE 配置项](../../../administration/FE_configuration.md) `max_broker_load_job_concurrency` 指定 StarRocks 集群中可以并行执行的 Broker Load 作业的最大数量。如果某一时间段内提交的 Broker Load 作业总数超过最大数量，则超出的作业会按照优先级在队列中排队等待调度。

   已经创建成功的导入作业，如果处于 **QUEUEING** 状态或者 **LOADING** 状态，那么您可以使用 [ALTER LOAD](../data-manipulation/ALTER_LOAD.md) 语句修改该作业的优先级。

   StarRocks 自 v2.5 版本起支持为导入作业设置 `priority` 参数。

## 列映射

如果源数据文件中的列与目标表中的列按顺序一一对应，您不需要指定列映射和转换关系。

如果源数据文件中的列与目标表中的列不能按顺序一一对应，包括数量或顺序不一致，则必须通过 `COLUMNS` 参数来指定列映射和转换关系。一般包括如下两种场景：

- **列数量一致、但是顺序不一致，并且数据不需要通过函数计算、可以直接落入目标表中对应的列。** 这种场景下，您需要在 `COLUMNS` 参数中按照源数据文件中的列顺序、使用目标表中对应的列名来配置列映射和转换关系。

  例如，目标表中有三列，按顺序依次为 `col1`、`col2` 和 `col3`；源数据文件中也有三列，按顺序依次对应目标表中的 `col3`、`col2` 和 `col1`。这种情况下，需要指定 `COLUMNS(col3, col2, col1)`。

- **列数量、顺序都不一致，并且某些列的数据需要通过函数计算以后才能落入目标表中对应的列。** 这种场景下，您不仅需要在 `COLUMNS` 参数中按照源数据文件中的列顺序、使用目标表中对应的列名来配置列映射关系，还需要指定参与数据计算的函数。以下为两个示例：

  - 目标表中有三列，按顺序依次为 `col1`、`col2` 和 `col3` ；源数据文件中有四列，前三列按顺序依次对应目标表中的 `col1`、`col2` 和 `col3`，第四列在目标表中无对应的列。这种情况下，需要指定 `COLUMNS(col1, col2, col3, temp)`，其中，最后一列可随意指定一个名称（如 `temp`）用于占位即可。
  - 目标表中有三列，按顺序依次为 `year`、`month` 和 `day`。源数据文件中只有一个包含时间数据的列，格式为 `yyyy-mm-dd hh:mm:ss`。这种情况下，可以指定 `COLUMNS (col, year = year(col), month=month(col), day=day(col))`。其中，`col` 是源数据文件中所包含的列的临时命名，`year = year(col)`、`month=month(col)` 和 `day=day(col)` 用于指定从 `col` 列提取对应的数据并落入目标表中对应的列，如 `year = year(col)` 表示通过 `year` 函数提取源数据文件中 `col` 列的 `yyyy` 部分的数据并落入目标表中的 `year` 列。

有关操作示例，请参见[设置列映射关系](#设置列映射关系)。

## 相关配置项

[FE 配置项](../../../administration/FE_configuration.md#fe-配置项) `max_broker_load_job_concurrency` 指定了 StarRocks 集群中可以并行执行的 Broker Load 作业的最大数量。

StarRocks v2.4 及以前版本中，如果某一时间段内提交的 Broker Load 作业总数超过最大数量，则超出作业会按照各自的提交时间放到队列中排队等待调度。

自 StarRocks v2.5 版本起，如果某一时间段内提交的 Broker Load 作业总数超过最大数量，则超出的作业会按照作业创建时指定的优先级被放到队列中排队等待调度。参见上面介绍的可选参数 `priority`。您可以使用 [ALTER LOAD](../data-manipulation/ALTER_LOAD.md) 语句修改处于 **QUEUEING** 状态或者 **LOADING** 状态的 Broker Load 作业的优先级。

## 示例

本文以 HDFS 数据源为例，介绍各种导入配置。

### 导入 CSV 格式的数据

本小节以 CSV 格式的数据为例，重点阐述在创建导入作业的时候，如何运用各种参数配置来满足不同业务场景下的各种导入要求。

#### 设置超时时间

StarRocks 数据库 `test_db` 里的表 `table1` 包含三列，按顺序依次为 `col1`、`col2` 和 `col3`。

数据文件 `example1.csv` 也包含三列，按顺序一一对应 `table1` 中的三列。

如果要把 `example1.csv` 中所有的数据都导入到 `table1` 中，并且要求超时时间最大不超过 3600 秒，可以执行如下语句：

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

#### 设置最大容错率

StarRocks 数据库 `test_db` 里的表 `table2` 包含三列，按顺序依次为 `col1`、`col2` 和 `col3`。

数据文件 `example2.csv` 也包含三列，按顺序一一对应 `table2` 中的三列。

如果要把 `example2.csv` 中所有的数据都导入到 `table2` 中，并且要求容错率最大不超过 0.1，可以执行如下语句：

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

#### 导入指定路径下所有数据文件

StarRocks 数据库 `test_db` 里的表 `table3` 包含三列，按顺序依次为 `col1`、`col2` 和 `col3`。

HDFS 集群的 `/user/starrocks/data/input/` 路径下所有数据文件也包含三列，按顺序一一对应 `table3` 中的三列，并且列分隔符为 Hive 文件的默认列分隔符 `\x01`。

如果要把 `hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/` 路径下所有数据文件的数据都导入到 `table3` 中，可以执行如下语句：

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

#### 设置 NameNode HA 机制

StarRocks 数据库 `test_db` 里的表 `table4` 包含三列，按顺序依次为 `col1`、`col2` 和 `col3`。

数据文件 `example4.csv` 也包含三列，按顺序一一对应 `table4` 中的三列。

如果要把 `example4.csv` 中所有的数据都导入到 `table4` 中，并且要求使用 NameNode HA 机制，可以执行如下语句：

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

#### 设置 Kerberos 认证方式

StarRocks 数据库 `test_db` 里的表 `table5` 包含三列，按顺序依次为 `col1`、`col2` 和 `col3`。

数据文件 `example5.csv` 也包含三列，按顺序一一对应 `table5` 中的三列。

如果要把 `example5.csv` 中所有的数据都导入到 `table5` 中，并且要求使用 Kerberos 认证方式、提供 keytab 文件的路径，可以执行如下语句：

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

#### 撤销已导入的数据


StarRocksのデータベース`test_db`にあるテーブル`table6`は、`col1`、`col2`、`col3`の3列を含んでいます。

データファイル`example6.csv`も3列を含み、`table6`の各列に順番に対応しています。

また、Broker Loadを通じて`example6.csv`の全てのデータを`table6`にインポートしました。

インポートしたデータを取り消したい場合は、以下のステートメントを実行します：

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

#### ターゲットパーティションの設定

StarRocksのデータベース`test_db`にあるテーブル`table7`は、`col1`、`col2`、`col3`の3列を含んでいます。

データファイル`example7.csv`も3列を含み、`table7`の各列に順番に対応しています。

`example7.csv`の全てのデータを`table7`のパーティション`p1`と`p2`にインポートしたい場合は、以下のステートメントを実行します：

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

StarRocksのデータベース`test_db`にあるテーブル`table8`は、`col1`、`col2`、`col3`の3列を含んでいます。

データファイル`example8.csv`も3列を含み、`table8`の`col2`、`col1`、`col3`に対応しています。

`example8.csv`の全てのデータを`table8`にインポートしたい場合は、以下のステートメントを実行します：

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

> **説明**
>
> 上記の例では、`example8.csv`と`table8`の列が順番に対応していないため、`column_list`パラメータを使用して`example8.csv`と`table8`の間の列マッピングを設定する必要があります。

#### フィルタ条件の設定

StarRocksのデータベース`test_db`にあるテーブル`table9`は、`col1`、`col2`、`col3`の3列を含んでいます。

データファイル`example9.csv`も3列を含み、`table9`の各列に順番に対応しています。

`example9.csv`の第1列の値が`20180601`より大きいデータ行のみを`table9`にインポートしたい場合は、以下のステートメントを実行します：

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

> **説明**
>
> 上記の例では、`example9.csv`と`table9`の列が同じ数で順番に対応していますが、WHERE句を使用して列に基づくフィルタ条件を指定するため、`column_list`パラメータを使用して`example9.csv`の列に一時的な名前を付ける必要があります。

#### HLL型列を含むテーブルへのデータインポート

StarRocksのデータベース`test_db`にあるテーブル`table10`は、`id`、`col1`、`col2`、`col3`の4列を含んでおり、`col1`と`col2`はHLL型の列です。

データファイル`example10.csv`は3列を含み、第1列が`table10`の`id`列に、第2列と第3列がそれぞれHLL型の列`col1`と`col2`に対応しています。これらは関数を通じてHLL型のデータに変換され、`col1`と`col2`にそれぞれ格納されます。

`example10.csv`の全てのデータを`table10`にインポートしたい場合は、以下のステートメントを実行します：

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

> **説明**
>
> 上記の例では、`column_list`パラメータを使用して`example10.csv`の3列を`id`、`temp1`、`temp2`として一時的に命名し、関数を使用してデータ変換ルールを指定しています。これには以下が含まれます：
>
> - `hll_hash`関数を使用して`example10.csv`の`temp1`、`temp2`列をHLL型のデータに変換し、それぞれ`table10`の`col1`、`col2`列に格納します。
>
> - `empty_hll`関数を使用して、`table10`の第4列にデフォルト値を補充します。

`hll_hash`関数と`empty_hll`関数の使用方法については、[hll_hash](../../sql-functions/aggregate-functions/hll_hash.md)と[hll_empty](../../sql-functions/aggregate-functions/hll_empty.md)を参照してください。

#### ファイルパスからのパーティションフィールドの抽出

Broker Loadは、StarRocksのテーブルで定義されたフィールドタイプに基づいて、インポートするファイルパス内のパーティションフィールドを解析する機能をサポートしています。これはApache Spark™のパーティションディスカバリ（Partition Discovery）機能に似ています。

StarRocksのデータベース`test_db`にあるテーブル`table11`は、`col1`、`col2`、`col3`、`city`、`utc_date`の5列を含んでいます。

HDFSクラスタの`/user/starrocks/data/input/dir/city=beijing`パスには以下のデータファイルが含まれています：

- `/user/starrocks/data/input/dir/city=beijing/utc_date=2019-06-26/0000.csv`

- `/user/starrocks/data/input/dir/city=beijing/utc_date=2019-06-26/0001.csv`

これらのデータファイルはそれぞれ3列を含み、`table11`の`col1`、`col2`、`col3`に対応しています。

`/user/starrocks/data/input/dir/city=beijing/utc_date=*/*`パスの全てのデータファイルを`table11`にインポートし、パス内のパーティションフィールド`city`と`utc_date`の情報を`table11`の対応する列に格納したい場合は、以下のステートメントを実行します：

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

#### ファイルパスに`%3A`を含む時間パーティションフィールドの抽出

HDFSのファイルパスではコロン（:）は許可されておらず、全てのコロン（:）は`%3A`に自動的に置き換えられます。

StarRocksのデータベース`test_db`にあるテーブル`table12`は、`data_time`、`col1`、`col2`の3列を含んでおり、テーブル構造は以下の通りです：

```SQL
data_time DATETIME,
col1        INT,
col2        INT
```

HDFSクラスタの`/user/starrocks/data`パスには以下のデータファイルがあります：

- `/user/starrocks/data/data_time=2020-02-17 00%3A00%3A00/example12.csv`

- `/user/starrocks/data/data_time=2020-02-18 00%3A00%3A00/example12.csv`

`example12.csv`の全てのデータを`table12`にインポートし、指定されたパス内のパーティションフィールド`data_time`の情報を`table12`の`data_time`列に格納したい場合は、以下のステートメントを実行します：

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

上記の例では、直接抽出されたパーティションフィールド`data_time`は`%3A`を含む文字列（例：`2020-02-17 00%3A00%3A00`）であるため、`str_to_date`関数を使用して文字列をDATETIME型のデータに変換した後、`table12`の`data_time`列に格納する必要があります。

#### `format_type_options`の設定

StarRocksのデータベース`test_db`にあるテーブル`table13`は、`col1`、`col2`、`col3`の3列を含んでいます。

データファイル`example13.csv`も3列を含み、`table13`の`col2`、`col1`、`col3`に対応しています。

`example13.csv`の全てのデータを`table13`にインポートし、`example13.csv`の最初の2行をスキップし、列区切り文字の前後の空白を削除し、エンクローズ文字をダブルクォート（`"`）、エスケープ文字をバックスラッシュ（`\`）に設定したい場合は、以下のステートメントを実行します：

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

### Parquet形式のデータのインポート

このセクションでは、Parquet形式のデータをインポートする際に注意すべきいくつかのパラメータ設定について説明します。

StarRocksのデータベース`test_db`にあるテーブル`table13`は、`col1`、`col2`、`col3`の3列を含んでいます。

データファイル`example13.parquet`も3列を含み、`table13`の各列に順番に対応しています。

`example13.parquet`の全てのデータを`table13`にインポートしたい場合は、以下のステートメントを実行します：

```SQL
LOAD LABEL test_db.label13
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example13.parquet")
    INTO TABLE table13
);
    FORMAT AS "parquet"
    (col1, col2, col3)
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

> **説明**
>
> Parquet 形式のデータをインポートする際は、デフォルトでファイルの拡張子（**.parquet**）を使用してデータファイルの形式を判断します。ファイル名に拡張子が含まれていない場合は、`FORMAT AS` パラメータを使用してデータファイルの形式を `Parquet` として指定する必要があります。

### ORC 形式のデータのインポート

このセクションでは、ORC 形式のデータをインポートする際に注意すべきいくつかのパラメータ設定について説明します。

StarRocks データベース `test_db` のテーブル `table14` は、`col1`、`col2`、`col3` の順に三つの列を含んでいます。

データファイル `example14.orc` も三つの列を含んでおり、`table14` の各列と順番に対応しています。

`example14.orc` の全てのデータを `table14` にインポートするには、以下のステートメントを実行します：

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

> **説明**
>
> - ORC 形式のデータをインポートする際は、デフォルトでファイルの拡張子（**.orc**）を使用してデータファイルの形式を判断します。ファイル名に拡張子が含まれていない場合は、`FORMAT AS` パラメータを使用してデータファイルの形式を `ORC` として指定する必要があります。
>
> - StarRocks v2.3 以前のバージョンでは、データファイルに ARRAY 型の列が含まれている場合、データファイルと StarRocks のテーブルの対応する列が同名であることを確認し、SET 句内には記述しないようにしてください。

