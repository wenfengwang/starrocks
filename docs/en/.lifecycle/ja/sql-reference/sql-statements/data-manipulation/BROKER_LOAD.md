---
displayed_sidebar: "Japanese"
---

# BROKER LOAD

import InsertPrivNote from '../../../assets/commonMarkdown/insertPrivNote.md'

## 説明

StarRocksでは、MySQLベースのロード方法であるBroker Loadを提供しています。ロードジョブを送信すると、StarRocksは非同期でジョブを実行します。ジョブの結果をクエリするには、`SELECT * FROM information_schema.loads`を使用できます。この機能はv3.1以降でサポートされています。バックグラウンド情報、原理、サポートされているデータファイル形式、単一テーブルのロードと複数テーブルのロードの方法、ジョブの結果を表示する方法の詳細については、[HDFSからデータをロードする](../../../loading/hdfs_load.md)および[クラウドストレージからデータをロードする](../../../loading/cloud_storage_load.md)を参照してください。

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

StarRocksでは、一部のリテラルがSQL言語によって予約キーワードとして使用されています。SQL文でこれらのキーワードを直接使用しないでください。SQL文でこのようなキーワードを使用する場合は、バッククォート（`）で囲んでください。[キーワード](../../../sql-reference/sql-statements/keywords.md)を参照してください。

## パラメータ

### database_nameとlabel_name

`label_name`はロードジョブのラベルを指定します。

`database_name`は、宛先テーブルが所属するデータベースの名前をオプションで指定します。

各ロードジョブには、データベース全体で一意のラベルがあります。ロードジョブのラベルを使用して、ロードジョブの実行状態を表示し、同じデータを繰り返しロードするのを防ぐことができます。ロードジョブが**FINISHED**状態に入ると、そのラベルは再利用できなくなります。**CANCELLED**状態に入ったロードジョブのラベルのみが再利用できます。ほとんどの場合、ロードジョブのラベルは、そのロードジョブを再試行して同じデータをロードするために再利用され、Exactly-Onceセマンティクスが実現されます。

ラベルの命名規則については、[システム制限](../../../reference/System_limit.md)を参照してください。

### data_desc

ロードするデータのバッチの説明です。各`data_desc`ディスクリプタは、データソース、ETL関数、宛先のStarRocksテーブル、および宛先のパーティションなどの情報を宣言します。

Broker Loadは、一度に複数のデータファイルをロードすることをサポートしています。1つのロードジョブで、複数の`data_desc`ディスクリプタを使用して、ロードしたい複数のデータファイルを宣言するか、1つの`data_desc`ディスクリプタを使用して、その中のすべてのデータファイルをロードしたいファイルパスを宣言することができます。Broker Loadは、複数のデータファイルをロードするために実行される各ロードジョブのトランザクション的な原子性を保証することもできます。原子性とは、1つのロードジョブで複数のデータファイルをロードする場合、すべてが成功するか、すべてが失敗するかのいずれかです。一部のデータファイルのロードが成功し、他のファイルのロードが失敗することはありません。

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

`data_desc`には、次のパラメータが含まれている必要があります。

- `file_path`

  ロードしたい1つ以上のデータファイルの保存パスを指定します。

  このパラメータを1つのデータファイルの保存パスとして指定することができます。たとえば、このパラメータを`"hdfs://<hdfs_host>:<hdfs_port>/user/data/tablename/20210411"`と指定して、HDFSサーバーのパス`/user/data/tablename`から名前が`20210411`のデータファイルをロードすることができます。

  ワイルドカード`?`、`*`、`[]`、`{}`、または`^`を使用して、複数のデータファイルの保存パスを指定することもできます。[ワイルドカードリファレンス](https://hadoop.apache.org/docs/stable/api/org/apache/hadoop/fs/FileSystem.html#globStatus-org.apache.hadoop.fs.Path-)を参照してください。たとえば、このパラメータを`"hdfs://<hdfs_host>:<hdfs_port>/user/data/tablename/*/*"`または`"hdfs://<hdfs_host>:<hdfs_port>/user/data/tablename/dt=202104*/*"`と指定して、HDFSサーバーのパス`/user/data/tablename`のすべてのパーティションまたは`202104`パーティションのデータファイルをロードすることができます。

  > **注意**
  >
  > ワイルドカードは中間パスを指定するためにも使用できます。

  前述の例では、`hdfs_host`および`hdfs_port`パラメータは次のように説明されています。

  - `hdfs_host`：HDFSクラスタのNameNodeホストのIPアドレスです。

  - `hdfs_host`：HDFSクラスタのNameNodeホストのFSポートです。デフォルトのポート番号は`9000`です。

  > **注意**
  >
  > - Broker Loadは、S3またはS3Aプロトコルに従ってAWS S3にアクセスすることをサポートしています。したがって、AWS S3からデータをロードする場合は、ファイルパスとして渡すS3 URIのプレフィックスとして`s3://`または`s3a://`を含めることができます。
  > - Broker Loadは、gsプロトコルに従ってGoogle GCSにアクセスすることのみをサポートしています。したがって、Google GCSからデータをロードする場合は、ファイルパスとして渡すGCS URIのプレフィックスとして`gs://`を含める必要があります。
  > - Blob Storageからデータをロードする場合は、wasbまたはwasbsプロトコルを使用してデータにアクセスする必要があります：
  >   - ストレージアカウントがHTTP経由でアクセスを許可する場合は、wasbプロトコルを使用し、ファイルパスを`wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`として書きます。
  >   - ストレージアカウントがHTTPS経由でアクセスを許可する場合は、wasbsプロトコルを使用し、ファイルパスを`wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`として書きます。
  > - Data Lake Storage Gen1からデータをロードする場合は、adlプロトコルを使用してデータにアクセスし、ファイルパスを`adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`として書きます。
  > - Data Lake Storage Gen2からデータをロードする場合は、abfsまたはabfssプロトコルを使用してデータにアクセスする必要があります：
  >   - ストレージアカウントがHTTP経由でアクセスを許可する場合は、abfsプロトコルを使用し、ファイルパスを`abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>`として書きます。
  >   - ストレージアカウントがHTTPS経由でアクセスを許可する場合は、abfssプロトコルを使用し、ファイルパスを`abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>`として書きます。

- `INTO TABLE`

  宛先のStarRocksテーブルの名前を指定します。

`data_desc`は、次のオプションのパラメータを含めることもできます。

- `NEGATIVE`

  特定のデータバッチのロードを取り消します。これを実現するには、`NEGATIVE`キーワードを指定して同じデータバッチをロードする必要があります。

  > **注意**
  >
  > このパラメータは、StarRocksテーブルが集計テーブルであり、そのすべての値列が`sum`関数によって計算される場合にのみ有効です。

- `PARTITION`

   データをロードするパーティションを指定します。このパラメータを指定しない場合、デフォルトでソースデータはStarRocksテーブルのすべてのパーティションにロードされます。

- `TEMPORARY PARTITION`

  データをロードする[一時パーティション](../../../table_design/Temporary_partition.md)の名前を指定します。複数の一時パーティションを指定する場合は、カンマ（,）で区切る必要があります。

- `COLUMNS TERMINATED BY`

  データファイルで使用される列セパレータを指定します。このパラメータを指定しない場合、デフォルトで`\t`（タブ）が指定されます。このパラメータで指定する列セパレータは、実際にデータファイルで使用されている列セパレータと同じである必要があります。そうでない場合、データ品質が不十分でロードジョブが失敗し、その`State`が`CANCELLED`になります。

  Broker LoadジョブはMySQLプロトコルに従って送信されます。StarRocksとMySQLは、ロードリクエストでエスケープ文字を使用します。したがって、列セパレータがタブなどの見えない文字の場合は、列セパレータの前にバックスラッシュ（\）を追加する必要があります。たとえば、列セパレータが`\t`の場合は`\\t`を入力し、列セパレータが`\n`の場合は`\\n`を入力する必要があります。Apache Hive™ファイルでは、列セパレータとして`\x01`が使用されるため、データファイルがHiveからの場合は`\\x01`を入力する必要があります。

  > **注意**
  >
  > - CSVデータの場合、50バイトを超えないUTF-8文字列（コンマ（,）、タブ、またはパイプ（|）など）をテキストデリミタとして使用できます。
  > - ヌル値は`\N`を使用して示されます。たとえば、データファイルは3つの列から構成され、そのデータファイルのレコードは最初の列と3番目の列にデータを保持していますが、2番目の列にはデータがありません。この場合、ヌル値を示すために2番目の列に`\N`を使用する必要があります。これは、レコードを`a,\N,b`としてコンパイルする必要があります。`a,,b`は、レコードの2番目の列が空の文字列を保持していることを示します。

- `ROWS TERMINATED BY`

  データファイルで使用される行セパレータを指定します。このパラメータを指定しない場合、デフォルトで`\n`（改行）が指定されます。このパラメータで指定する行セパレータは、実際にデータファイルで使用されている行セパレータと同じである必要があります。そうでない場合、データ品質が不十分でロードジョブが失敗し、その`State`が`CANCELLED`になります。このパラメータはv2.5.4以降でサポートされています。

  行セパレータの使用方法についての使用上の注意については、前述の`COLUMNS TERMINATED BY`パラメータの使用上の注意を参照してください。

- `FORMAT AS`

  データファイルの形式を指定します。有効な値は`CSV`、`Parquet`、`ORC`です。このパラメータを指定しない場合、StarRocksは`file_path`パラメータで指定されたファイル名拡張子**.csv**、**.parquet**、または**.orc**に基づいてデータファイルの形式を判断します。

- `format_type_options`

  `FORMAT AS`が`CSV`に設定されている場合、CSV形式のオプションを指定します。構文:

  ```JSON
  (
      key = value
      key = value
      ...
  )
  ```

  > **注意**
  >
  > `format_type_options`はv3.0以降でサポートされています。

  次の表に、オプションの説明を示します。

  | **パラメータ** | **説明**                                                    |
  | ------------- | ------------------------------------------------------------ |
  | skip_header   | CSV形式のデータファイルの場合、データファイルの最初の行をスキップするかどうかを指定します。タイプ: INTEGER。デフォルト値: `0`。<br />一部のCSV形式のデータファイルでは、先頭の行が列名や列のデータ型などのメタデータを定義するために使用されます。`skip_header`パラメータを設定することで、データロード中にStarRocksがデータファイルの最初の行をスキップできるようにすることができます。たとえば、このパラメータを`1`に設定すると、StarRocksはデータロード中にデータファイルの最初の行をスキップします。<br />データファイルの先頭の行は、ロードステートメントで指定した行セパレータを使用して区切られている必要があります。 |
  | trim_space    | CSV形式のデータファイルの場合、列セパレータの前後にあるスペースを削除するかどうかを指定します。タイプ: BOOLEAN。デフォルト値: `false`。<br />一部のデータベースでは、CSV形式のデータファイルとしてデータをエクスポートする際に、列セパレータにスペースが追加されます。このようなスペースは、その位置に応じて先行スペースまたは後続スペースと呼ばれます。`trim_space`パラメータを設定することで、データロード中にStarRocksがこのような不要なスペースを削除できるようにすることができます。<br />StarRocksは、`enclose`で指定された文字で囲まれたフィールド内のスペース（先行スペースと後続スペースを含む）を削除しません。たとえば、次のフィールド値は、パイプ（<code class="language-text">&#124;</code>）を列セパレータとし、ダブルクォーテーションマーク（`"`）を`enclose`で指定された文字として使用しています:<br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> <br /><code class="language-text">&#124;" Love StarRocks "&#124;</code> <br /><code class="language-text">&#124; "Love StarRocks" &#124;</code> <br />`trim_space`を`true`に設定すると、StarRocksは次のように前述のフィールド値を処理します:<br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> <br /><code class="language-text">&#124;" Love StarRocks "&#124;</code> <br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> |
  | enclose       | CSV形式のデータファイルの場合、[RFC4180](https://www.rfc-editor.org/rfc/rfc4180)に従ってフィールド値を囲むために使用される文字を指定します。タイプ: 1バイト文字。デフォルト値: `NONE`。最も一般的な文字はシングルクォーテーションマーク（`'`）とダブルクォーテーションマーク（`"`）です。<br />`enclose`で指定された文字で囲まれたすべての特殊文字（行セパレータや列セパレータを含む）は通常のシンボルと見なされます。StarRocksはRFC4180以上のことができますので、任意の1バイト文字を`enclose`で指定された文字として指定することができます。<br />フィールド値に`enclose`で指定された文字が含まれる場合は、同じ文字を使用してその`enclose`で指定された文字をエスケープすることができます。たとえば、`enclose`を`"`に設定し、フィールド値が`a "quoted" c`である場合、フィールド値を`"a ""quoted"" c"`としてデータファイルに入力することができます。 |
  | escape        | 行セパレータ、列セパレータ、エスケープ文字、および`enclose`で指定された文字など、さまざまな特殊文字をエスケープするために使用される文字を指定します。タイプ: 1バイト文字。デフォルト値: `NONE`。最も一般的な文字はスラッシュ（`\`）ですが、SQL文ではスラッシュを2重に書く必要があります（`\\`）。<br />**注意**<br />`escape`で指定された文字は、`enclose`で指定された文字のペアの内側と外側の両方に適用されます。<br />2つの例を以下に示します:<ul><li>`enclose`を`"`に設定し、`escape`を`\`に設定した場合、StarRocksは`"say \"Hello world\""`を`say "Hello world"`に解析します。</li><li>列セパレータがカンマ（`,`）の場合を想定します。`escape`を`\`に設定した場合、StarRocksは`a, b\, c`を2つの別々のフィールド値、`a`と`b, c`に解析します。</li></ul> |

- `column_list`

  データファイルとStarRocksテーブルの間の列マッピングを指定します。構文: `(<column_name>[, <column_name> ...])`。`column_list`で宣言された列は、名前でStarRocksテーブルの列にマッピングされます。

  > **注意**
  >
  > データファイルの列がStarRocksテーブルの列に順番にマッピングされる場合、`column_list`を指定する必要はありません。

  特定の列をスキップする場合は、データファイルのその列を一時的にStarRocksテーブルの列とは異なる名前で指定するだけです。詳細については、[ロード時のデータ変換](../../../loading/Etl_in_loading.md)を参照してください。

- `COLUMNS FROM PATH AS`

  指定したファイルパスから1つ以上のパーティションフィールドの情報を抽出します。このパラメータは、ファイルパスにパーティションフィールドが含まれる場合にのみ有効です。

  たとえば、データファイルがパス`/path/col_name=col_value/file1`に保存されており、`col_name`がパーティションフィールドであり、StarRocksテーブルの列にマッピングできる場合、このパラメータを`col_name`として指定することができます。これにより、StarRocksはパスから`col_value`の値を抽出し、`col_name`にマッピングされたStarRocksテーブルの列にロードします。

  > **注意**
  >
  > このパラメータは、HDFSからデータをロードする場合にのみ使用できます。

- `SET`

  データファイルの列を変換するために使用する関数を1つ以上指定します。例:

  - StarRocksテーブルは、`col1`、`col2`、および`col3`の3つの列で構成されています。データファイルは4つの列で構成されており、最初の2つの列がStarRocksテーブルの`col1`と`col2`に順番にマッピングされ、最後の2つの列の合計がStarRocksテーブルの`col3`にマッピングされます。この場合、`column_list`を`(col1,col2,tmp_col3,tmp_col4)`と指定し、SET句で`(col3=tmp_col3+tmp_col4)`を指定してデータ変換を実装する必要があります。
  - StarRocksテーブルは、`year`、`month`、および`day`の3つの列で構成されています。データファイルは、`yyyy-mm-dd hh:mm:ss`形式の日付と時刻の値を含む1つの列のみで構成されています。この場合、`column_list`を`(tmp_time)`と指定し、SET句で`(year = year(tmp_time), month=month(tmp_time), day=day(tmp_time))`を指定してデータ変換を実装する必要があります。

- `WHERE`

  ソースデータをフィルタリングするための条件を指定します。StarRocksは、WHERE句で指定されたフィルタ条件を満たすソースデータのみをロードします。

### WITH BROKER

v2.3以前では、`WITH BROKER "<broker_name>"`を入力して使用するブローカーを指定します。v2.5以降では、ブローカーを指定する必要はありませんが、`WITH BROKER`キーワードを保持する必要があります。

> **注意**
>
> v2.4以前では、StarRocksはブローカーに依存して、Broker Loadジョブを実行する際にStarRocksクラスタと外部ストレージシステムとの間の接続を設定します。これを「ブローカーベースのロード」と呼びます。ブローカーは、ファイルシステムインターフェースと統合された独立したステートレスサービスです。ブローカーを使用することで、StarRocksは外部ストレージシステムに保存されているデータファイルにアクセスして読み取ることができ、これらのデータファイルのデータを事前処理してロードするために独自の計算リソースを使用することができます。
>
> v2.5以降、StarRocksはブローカーへの依存を削除し、「ブローカーフリーロード」を実現します。
>
> [SHOW BROKER](../Administration/SHOW_BROKER.md)ステートメントを使用して、StarRocksクラスタにデプロイされているブローカーを確認できます。ブローカーがデプロイされていない場合は、[ブローカーのデプロイ](../../../deployment/deploy_broker.md)の手順に従ってブローカーをデプロイすることができます。

### StorageCredentialParams

StarRocksがストレージシステムにアクセスするために使用する認証情報です。

#### HDFS

オープンソースのHDFSは、シンプル認証とKerberos認証の2つの認証方法をサポートしています。Broker Loadはデフォルトでシンプル認証を使用します。オープンソースのHDFSはまた、NameNodeのHAメカニズムを構成することもサポートしています。HDFSをストレージシステムとして選択した場合、次のように認証構成とHA構成を指定できます。

- 認証構成

  - シンプル認証を使用する場合、`StorageCredentialParams`を次のように構成します:

    ```Plain
    "hadoop.security.authentication" = "simple",
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
    ```

    次の表に、`StorageCredentialParams`のパラメータの説明を示します。

    | パラメータ                       | 説明                                                         |
    | ------------------------------- | ------------------------------------------------------------ |
    | hadoop.security.authentication  | 認証方法。有効な値: `simple`および`kerberos`。デフォルト値: `simple`。`simple`はシンプル認証を表し、認証なしを意味し、`kerberos`はKerberos認証を表します。 |
    | username                        | HDFSクラスタのNameNodeにアクセスするために使用するアカウントのユーザー名です。 |
    | password                        | HDFSクラスタのNameNodeにアクセスするために使用するアカウントのパスワードです。 |

  - Kerberos認証を使用する場合、`StorageCredentialParams`を次のように構成します:

    ```Plain
    "hadoop.security.authentication" = "kerberos",
    "kerberos_principal" = "nn/zelda1@ZELDA.COM",
    "kerberos_keytab" = "/keytab/hive.keytab",
    "kerberos_keytab_content" = "YWFhYWFh"
    ```

    次の表に、`StorageCredentialParams`のパラメータの説明を示します。

    | パラメータ                       | 説明                                                         |
    | ------------------------------- | ------------------------------------------------------------ |
    | hadoop.security.authentication  | 認証方法。有効な値: `simple`および`kerberos`。デフォルト値: `simple`。`simple`はシンプル認証を表し、認証なしを意味し、`kerberos`はKerberos認証を表します。 |
    | kerberos_principal              | 認証されるKerberosプリンシパルです。各プリンシパルは、HDFSクラスタ全体で一意であることを保証するために、次の3つのパートで構成されます:<ul><li>`username`または`servicename`：プリンシパルの名前。</li><li>`instance`：HDFSクラスタのノードで認証されるサーバーの名前。サーバー名は、複数の独立した認証が行われる複数のDataNodeでHDFSクラスタが構成されている場合に、プリンシパルが一意であることを保証するために使用されます。</li><li>`realm`：レルムの名前。レルム名は大文字である必要があります。例: `nn/[zelda1@ZELDA.COM](mailto:zelda1@ZELDA.COM)`。</li></ul> |
    | kerberos_keytab                 | Kerberosキータブファイルの保存パスです。 |
    | kerberos_keytab_content         | KerberosキータブファイルのBase64エンコードされた内容です。`kerberos_keytab`または`kerberos_keytab_content`のいずれかを指定できます。 |

    複数のKerberosユーザーを構成している場合は、少なくとも1つの独立した[ブローカーグループ](../../../deployment/deploy_broker.md)がデプロイされていることを確認し、ロードステートメントで`WITH BROKER "<broker_name>"`を入力して使用するブローカーグループを指定する必要があります。さらに、ブローカーの起動スクリプトファイル**start_broker.sh**を開き、ファイルの42行目を変更して、ブローカーが**krb5.conf**ファイルを読み取るようにします。例:

    ```Plain
    export JAVA_OPTS="-Dlog4j2.formatMsgNoLookups=true -Xmx1024m -Dfile.encoding=UTF-8 -Djava.security.krb5.conf=/etc/krb5.conf"
    ```

    > **注意**
    >
    > - 前述の例では、`/etc/krb5.conf`は実際の**krb5.conf**ファイルの保存パスに置き換えることができます。ブローカーがそのファイルに読み取り権限を持っていることを確認してください。ブローカーグループが複数のブローカーで構成されている場合は、各ブローカーノードの**start_broker.sh**ファイルを変更し、ブローカーノードを再起動して変更を有効にする必要があります。
    > - [SHOW BROKER](../Administration/SHOW_BROKER.md)ステートメントを使用して、StarRocksクラスタにデプロイされているブローカーを確認できます。

- HA構成

  HDFSクラスタのNameNodeにHAメカニズムを構成することができます。これにより、NameNodeが別のノードに切り替わった場合、StarRocksは自動的に新しいNameNodeとして機能するノードを特定できます。これには次のシナリオが含まれます:

  - 1つのKerberosユーザーが構成された単一のHDFSクラスタからデータをロードする場合、ロードベースのロードとロードフリーロードの両方がサポートされます。

    - ロードベースのロードを実行するには、少なくとも1つの独立した[ブローカーグループ](../../../deployment/deploy_broker.md)がデプロイされていることを確認し、HDFSクラスタを提供するブローカーノードの`{deploy}/conf`パスに`hdfs-site.xml`ファイルを配置します。StarRocksは、ブローカーの起動時に環境変数`CLASSPATH`に`{deploy}/conf`パスを追加し、HDFSクラスタノードに関する情報をブローカーが読み取れるようにします。

    - ロードフリーロードを実行するには、各FEノードと各BEノードの`{deploy}/conf`パスに`hdfs-site.xml`ファイルを配置します。

  - 複数のKerberosユーザーが構成された単一のHDFSクラスタからデータをロードする場合、ブローカーベースのロードのみがサポートされます。少なくとも1つの独立した[ブローカーグループ](../../../deployment/deploy_broker.md)がデプロイされていることを確認し、HDFSクラスタを提供するブローカーノードの`{deploy}/conf`パスに`hdfs-site.xml`ファイルを配置します。StarRocksは、ブローカーの起動時に環境変数`CLASSPATH`に`{deploy}/conf`パスを追加し、HDFSクラスタノードに関する情報をブローカーが読み取れるようにします。

  - 複数のHDFSクラスタからデータをロードする場合（1つまたは複数のKerberosユーザーが構成されているかどうかに関係なく）、ブローカーベースのロードのみがサポートされます。これらのHDFSクラスタのそれぞれに少なくとも1つの独立した[ブローカーグループ](../../../deployment/deploy_broker.md)がデプロイされていることを確認し、ブローカーがHDFSクラスタのノードに関する情報を読み取れるようにするために、次のいずれかの操作を実行します:

    - 各HDFSクラスタを提供するブローカーノードの`{deploy}/conf`パスに`hdfs-site.xml`ファイルを配置します。StarRocksは、ブローカーの起動時に環境変数`CLASSPATH`に`{deploy}/conf`パスを追加し、そのHDFSクラスタのノードに関する情報をブローカーが読み取れるようにします。

    - 次のHA構成をジョブ作成時に追加します:

      ```Plain
      "dfs.nameservices" = "ha_cluster",
      "dfs.ha.namenodes.ha_cluster" = "ha_n1,ha_n2",
      "dfs.namenode.rpc-address.ha_cluster.ha_n1" = "<hdfs_host>:<hdfs_port>",
      "dfs.namenode.rpc-address.ha_cluster.ha_n2" = "<hdfs_host>:<hdfs_port>",
      "timeout" = "3600"
      "columns" = "col1, col2, col3"
`example1.csv`のすべてのデータを最大3600秒で`table1`にロードするには、次のコマンドを実行します。

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

#### 最大エラートレランスを設定する

StarRocksデータベース`test_db`には、`table2`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

データファイル`example2.csv`も、`col1`、`col2`、`col3`の順にマッピングされた3つの列で構成されています。

`example2.csv`のすべてのデータを`table2`に最大エラートレランス`0.1`でロードするには、次のコマンドを実行します。

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

StarRocksデータベース`test_db`には、`table3`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

HDFSクラスタの`/user/starrocks/data/input/`パスに格納されているすべてのデータファイルも、それぞれ`col1`、`col2`、`col3`の順にマッピングされた3つの列で構成されています。これらのデータファイルで使用される列区切り記号は`\x01`です。

`hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/`パスに格納されているこれらのデータファイルのすべてのデータを`table3`にロードするには、次のコマンドを実行します。

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

#### NameNode HAメカニズムを設定する

StarRocksデータベース`test_db`には、`table4`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

データファイル`example4.csv`も、`col1`、`col2`、`col3`の順にマッピングされた3つの列で構成されています。

`example4.csv`のすべてのデータをNameNodeに対してHAメカニズムを構成して`table4`にロードするには、次のコマンドを実行します。

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
    "dfs.ha.namenodes.my_ha" = "my_namenode1, my_namenode2","dfs.namenode.rpc-address.my_ha.my_namenode1" = "nn1_host:rpc_port",
    "dfs.namenode.rpc-address.my_ha.my_namenode2" = "nn2_host:rpc_port",
    "dfs.client.failover.proxy.provider" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
);
```

#### Kerberos認証を設定する

StarRocksデータベース`test_db`には、`table5`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

データファイル`example5.csv`も、`col1`、`col2`、`col3`の順にマッピングされた3つの列で構成されています。

`example5.csv`のすべてのデータをKerberos認証が構成され、キータブファイルパスが指定された状態で`table5`にロードするには、次のコマンドを実行します。

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

#### データのロードを取り消す

StarRocksデータベース`test_db`には、`table6`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

データファイル`example6.csv`も、`col1`、`col2`、`col3`の順にマッピングされた3つの列で構成されています。

Broker Loadジョブを実行して`example6.csv`のすべてのデータを`table6`にロードした場合、ロードしたデータを取り消すには、次のコマンドを実行します。

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

#### 宛先パーティションを指定する

StarRocksデータベース`test_db`には、`table7`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

データファイル`example7.csv`も、`col1`、`col2`、`col3`の順にマッピングされた3つの列で構成されています。

`example7.csv`のすべてのデータを`table7`の`p1`と`p2`の2つのパーティションにロードするには、次のコマンドを実行します。

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

#### カラムマッピングを設定する

StarRocksデータベース`test_db`には、`table8`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

データファイル`example8.csv`も、`col2`、`col1`、`col3`の順にマッピングされた3つの列で構成されています。

`example8.csv`のすべてのデータを`table8`にロードするには、次のコマンドを実行します。

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
> 上記の例では、`example8.csv`の列は、`table8`の列の並び順と同じ順序でマッピングすることはできません。そのため、`column_list`を使用して`example8.csv`と`table8`の間の列マッピングを設定する必要があります。

#### フィルタ条件を設定する

StarRocksデータベース`test_db`には、`table9`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

データファイル`example9.csv`も、`col1`、`col2`、`col3`の順にマッピングされた3つの列で構成されています。

`example9.csv`のうち、最初の列の値が`20180601`よりも大きいデータレコードのみを`table9`にロードするには、次のコマンドを実行します。

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
> 上記の例では、`example9.csv`の列は、`table9`の列の並び順と同じ順序でマッピングすることができますが、列ベースのフィルタ条件を指定するためにWHERE句を使用する必要があります。そのため、`column_list`を使用して`example9.csv`と`table9`の間の列マッピングを設定する必要があります。

#### HLL型の列を含むテーブルにデータをロードする

StarRocksデータベース`test_db`には、`table10`という名前のテーブルが含まれています。このテーブルは、`id`、`col1`、`col2`、`col3`の4つの列で構成されています。`col1`と`col2`はHLL型の列として定義されています。

データファイル`example10.csv`は、3つの列で構成されており、最初の列は`table10`の`id`に、2番目と3番目の列は`table10`の`col1`と`col2`にマッピングされます。`example10.csv`の2番目と3番目の列の値は、ロードされる前に関数を使用してHLL型のデータに変換することができます。

`example10.csv`のすべてのデータを`table10`にロードするには、次のコマンドを実行します。

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
> 上記の例では、`example10.csv`の3つの列は、`column_list`を使用して`id`、`temp1`、`temp2`という名前にマッピングされます。その後、関数を使用してデータを変換します。
>
> - `hll_hash`関数は、`example10.csv`の`temp1`と`temp2`の値をHLL型のデータに変換し、`example10.csv`の`temp1`と`temp2`を`table10`の`col1`と`col2`にマッピングします。
>
> - `empty_hll`関数は、指定されたデフォルト値を`table10`の`col3`に埋め込みます。

関数`hll_hash`および`empty_hll`の使用方法については、[hll_hash](../../sql-functions/aggregate-functions/hll_hash.md)および[hll_empty](../../sql-functions/aggregate-functions/hll_empty.md)を参照してください。

#### ファイルパスからパーティションフィールドの値を抽出する

Broker Loadは、StarRocksテーブルの列定義に基づいて、ファイルパスに含まれる特定のパーティションフィールドの値を解析することができます。これは、Apache Spark™のPartition Discovery機能に似たStarRocksの機能です。

StarRocksデータベース`test_db`には、`table11`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`、`city`、`utc_date`の5つの列で構成されています。

HDFSクラスタのファイルパス`/user/starrocks/data/input/dir/city=beijing`には、次のデータファイルが含まれています。

- `/user/starrocks/data/input/dir/city=beijing/utc_date=2019-06-26/0000.csv`

- `/user/starrocks/data/input/dir/city=beijing/utc_date=2019-06-26/0001.csv`

これらのデータファイルは、それぞれ3つの列で構成されており、`col1`、`col2`、`col3`の順に`table11`にマッピングされます。

`/user/starrocks/data/input/dir/city=beijing/utc_date=*/*`のファイルパスに含まれるすべてのデータファイルのデータを`table11`にロードし、同時に、ファイルパスに含まれるパーティションフィールド`city`と`utc_date`の値を抽出し、`table11`の`city`と`utc_date`にロードする場合、次のコマンドを実行します。

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

#### `%3A`を含むファイルパスからパーティションフィールドの値を抽出する

HDFSでは、ファイルパスにコロン（:）を含めることはできません。すべてのコロン（:）は`%3A`に変換されます。

StarRocksデータベース`test_db`には、`table12`という名前のテーブルが含まれています。このテーブルは、`data_time`、`col1`、`col2`の3つの列で構成されています。テーブルのスキーマは次のとおりです。

```SQL
data_time DATETIME,
col1        INT,
col2        INT
```

HDFSクラスタのファイルパス`/user/starrocks/data`には、次のデータファイルが含まれています。

- `/user/starrocks/data/data_time=2020-02-17 00%3A00%3A00/example12.csv`

- `/user/starrocks/data/data_time=2020-02-18 00%3A00%3A00/example12.csv`

`example12.csv`のすべてのデータを`table12`にロードし、同時に、ファイルパスに含まれるパーティションフィールド`data_time`の値を抽出し、`table12`の`data_time`にロードする場合、次のコマンドを実行します。

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
> 上記の例では、パーティションフィールド`data_time`から抽出された値は、`2020-02-17 00%3A00%3A00`のように`%3A`を含む文字列です。そのため、`str_to_date`関数を使用して文字列をDATETIME型のデータに変換する必要があります。

### Parquetデータのロード

このセクションでは、Parquetデータをロードする際に注意する必要があるいくつかのパラメータ設定について説明します。

StarRocksデータベース`test_db`には、`table13`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

データファイル`example13.parquet`も、`col1`、`col2`、`col3`の順にマッピングされた3つの列で構成されています。

`example13.parquet`のすべてのデータを`table13`にロードするには、次のコマンドを実行します。

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
> Parquetデータをロードする場合、デフォルトではStarRocksはファイル名に拡張子**.parquet**が含まれているかどうかに基づいてデータファイルの形式を判断します。ファイル名に拡張子**.parquet**が含まれていない場合は、`FORMAT AS`を使用してデータファイルの形式を`Parquet`として指定する必要があります。

### ORCデータのロード

このセクションでは、ORCデータをロードする際に注意する必要があるいくつかのパラメータ設定について説明します。

StarRocksデータベース`test_db`には、`table14`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

データファイル`example14.orc`も、`col1`、`col2`、`col3`の順にマッピングされた3つの列で構成されています。

`example14.orc`のすべてのデータを`table14`にロードするには、次のコマンドを実行します。

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
> - ORCデータをロードする場合、デフォルトではStarRocksはファイル名に拡張子**.orc**が含まれているかどうかに基づいてデータファイルの形式を判断します。ファイル名に拡張子**.orc**が含まれていない場合は、`FORMAT AS`を使用してデータファイルの形式を`ORC`として指定する必要があります。
>
> - StarRocks v2.3以前では、データファイルにARRAY型の列が含まれている場合、ORCデータファイルの列はStarRocksテーブルのマッピング列と同じ名前であることを確認する必要があり、列はSET句で指定することはできません。
