---
displayed_sidebar: "Japanese"
---

# ブローカーロード

import InsertPrivNote from '../../../assets/commonMarkdown/insertPrivNote.md'

## 説明

StarRocksは、MySQLベースのローディング方法であるブローカーロードを提供します。ロードジョブを送信すると、StarRocksは非同期でジョブを実行します。`SELECT * FROM information_schema.loads`を使用してジョブの結果をクエリできます。この機能はv3.1以降でサポートされています。背景情報、原則、サポートされているデータファイル形式、単一テーブルのロード、複数テーブルのロードの実行方法、ジョブ結果の表示方法の詳細については、[HDFSからデータをロード](../../../loading/hdfs_load.md)および[クラウドストレージからデータをロード](../../../loading/cloud_storage_load.md)を参照してください。

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

StarRocksでは、いくつかのリテラルがSQL言語の予約キーワードとして使用されています。これらのキーワードを直接SQLステートメントで使用しないでください。SQLステートメントでこのようなキーワードを使用する場合は、バッククォート（`）で囲んでください。[キーワード](../../../sql-reference/sql-statements/keywords.md)を参照してください。

## パラメータ

### database_nameおよびlabel_name

`label_name`はロードジョブのラベルを指定します。

`database_name`は、宛先テーブルが所属するデータベースの名前をオプションで指定します。

各ロードジョブには、データベース全体で一意であるラベルがあります。ロードジョブのラベルを使用して、ロードジョブの実行状況を表示したり、同じデータを繰り返しロードすることを防いだりできます。ロードジョブが**FINISHED**状態に入ると、そのラベルは再利用できません。**CANCELLED**状態に入ったロードジョブのラベルのみ再利用できます。ほとんどの場合、ロードジョブのラベルは、そのロードジョブを再試行して同じデータをロードし、つまりExactly-Onceセマンティクスを実現するために再利用されます。

ラベルの命名規則については、「System limits」を参照してください。

### data_desc

ロードするデータのバッチの説明です。各`data_desc`ディスクリプタは、データソース、ETL（抽出、変換、ロード）関数、宛先StarRocksテーブル、および宛先パーティションなどの情報を宣言します。

ブローカーロードは複数のデータファイルを同時にロードできます。1つのロードジョブで、複数の`data_desc`記述子を使用して、ロードする複数のデータファイルを宣言したり、1つの`data_desc`記述子を使用して、その中のすべてのデータファイルをロードするファイルパスを宣言したりできます。ブローカーロードはまた、複数のデータファイルをロードする実行される各ロードジョブのトランザクション的な原子性を保証できます。原子性とは、1つのロードジョブで複数のデータファイルをロードする場合、それらのすべてが成功するか失敗するかのどちらかであることを意味します。1つのデータファイルがロードされる場合はあり得ません。

`data_desc`は以下の構文をサポートしています。

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

`data_desc`には以下のパラメータが必須です。

- `file_path`

  ロードする1つまたは複数のデータファイルの保存パスを指定します。

  このパラメータは1つのデータファイルの保存パスを指定することができます。たとえば、このパラメータを`"hdfs://<hdfs_host>:<hdfs_port>/user/data/tablename/20210411"`として指定すると、HDFSサーバーのパス`/user/data/tablename`から`20210411`という名前のデータファイルをロードすることができます。

  また、ワイルドカード`?`、`*`、`[]`、`{}`、`^`を使用して複数のデータファイルの保存パスを指定することもできます。[ワイルドカードリファレンス](https://hadoop.apache.org/docs/stable/api/org/apache/hadoop/fs/FileSystem.html#globStatus-org.apache.hadoop.fs.Path-)を参照してください。たとえば、`"hdfs://<hdfs_host>:<hdfs_port>/user/data/tablename/*/*"`または`"hdfs://<hdfs_host>:<hdfs_port>/user/data/tablename/dt=202104*/*"`として指定すると、HDFSサーバーのパス`/user/data/tablename`からすべてのパーティションまたは`202104`のパーティションのデータファイルをロードすることができます。

  > **注意**
  >
  > ワイルドカードは中間パスを指定する際にも使用できます。

  上記の例で、`hdfs_host`と`hdfs_port`パラメータは次のように記述されています。

  - `hdfs_host`: HDFSクラスタのNameNodeホストのIPアドレス。

  - `hdfs_host`: HDFSクラスタのNameNodeホストのファイルシステムポート。デフォルトのポート番号は`9000`です。

  > **注意**
  >
  > - ブローカーロードはS3またはS3Aプロトコルに従ってAWS S3へのアクセスをサポートしています。したがって、AWS S3からデータをロードする場合は、ファイルパスとして渡すS3 URIに`s3://`または`s3a://`を含めることができます。
  > - ブローカーロードはgsプロトコルにのみ従ってGoogle GCSへのアクセスをサポートします。したがって、Google GCSからデータをロードする場合は、ファイルパスとして渡すGCS URIに`gs://`を含める必要があります。
  > - Blob Storageからデータをロードする場合は、wasbまたはwasbsプロトコルを使用してデータにアクセスする必要があります。次のようにファイルパスを記述してください:
  >   - ストレージアカウントがHTTP経由でアクセスを許可している場合は、wasbプロトコルを使用してファイルパスを`wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`と記述してください。
  >   - ストレージアカウントがHTTPS経由でアクセスを許可している場合は、wasbsプロトコルを使用してファイルパスを`wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`と記述してください。
  > - Data Lake Storage Gen1からデータをロードする場合は、データにアクセスするためにadlプロトコルを使用し、ファイルパスを`adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`と記述してください。
  > - Data Lake Storage Gen2からデータをロードする場合は、abfsまたはabfssプロトコルを使用してデータにアクセスする必要があります。次のようにファイルパスを記述してください:
  >   - ストレージアカウントがHTTP経由でアクセスを許可している場合は、abfsプロトコルを使用してファイルパスを`abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>`と記述してください。
  >   - ストレージアカウントがHTTPS経由でアクセスを許可している場合は、abfssプロトコルを使用してファイルパスを`abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>`と記述してください。

- `INTO TABLE`

  宛先のStarRocksテーブルの名前を指定します。

`data_desc`には、以下のパラメータをオプションで含めることもできます。

- `NEGATIVE`

  特定のデータバッチのロードを取り消します。このためには、`NEGATIVE`キーワードを指定して同じデータバッチをロードする必要があります。

  > **注意**
  >
  > このパラメータは、StarRocksテーブルが集計テーブルであり、そのすべての値列が`sum`関数によって計算されている場合のみ有効です。

- `PARTITION`

  ロードするデータを指定するパーティションを指定します。このパラメータを指定しない場合は、デフォルトでソースデータがStarRocksテーブルのすべてのパーティションにロードされます。

- `TEMPORARY PARTITION`

  ロードするデータを指定する[一時パーティション](../../../table_design/Temporary_partition.md)の名前を指定します。複数の一時パーティションを指定することができますが、その場合はコンマ(,)で区切られていなければなりません。

- `COLUMNS TERMINATED BY`

  データファイルで使用されている列セパレータを指定します。このパラメータを指定しない場合、デフォルトで`\t`（タブ）が使用されます。このパラメータで指定した列セパレータは、実際にデータファイルで使用されている列セパレータと同じでなければなりません。そうでない場合、データ品質が不十分であるため、ロードジョブは失敗し、その`State`は`CANCELLED`となります。

  ブローカーロードジョブは、MySQLプロトコルに従って提出されます。StarRocksとMySQLは両方ともロードリクエストでエスケープ文字を使用します。したがって、列セパレータがタブなど不可視文字である場合は、列セパレータの前にバックスラッシュ（\）を追加する必要があります。たとえば、列セパレータが`\t`の場合は`\\t`と入力する必要があり、列セパレータが`\n`の場合は`\\n`と入力する必要があります。Apache Hive™ファイルは列セパレータとして`\x01`を使用するため、Hiveからのデータファイルであれば`\\x01`と入力する必要があります。

  > **注意**
  >
  > - CSVデータの場合、50バイトを超えないようなUTF-8文字列（コンマ（,）、タブ、パイプ（|）など）をテキストデリミタとして使用できます。

- Null values are denoted by using `\N`. For example, a data file consists of three columns, and a record from that data file holds data in the first and third columns but no data in the second column. In this situation, you need to use `\N` in the second column to denote a null value. This means the record must be compiled as `a,\N,b` instead of `a,,b`. `a,,b` denotes that the second column of the record holds an empty string.
- `ROWS TERMINATED BY`  
  Specifies the row separator used in the data file. By default, if you do not specify this parameter, this parameter defaults to `\n`, indicating line break. The row separator you specify using this parameter must be the same as the row separator that is actually used in the data file. Otherwise, the load job will fail due to inadequate data quality, and its `State` will be `CANCELLED`. This parameter is supported from v2.5.4 onwards.  
  For the usage notes about the row separator, see the usage notes for the preceding `COLUMNS TERMINATED BY` parameter.
- `FORMAT AS`  
  Specifies the format of the data file. Valid values: `CSV`, `Parquet`, and `ORC`. By default, if you do not specify this parameter, StarRocks determines the data file format based on the filename extension **.csv**, **.parquet**, or **.orc** specified in the `file_path` parameter.
- `format_type_options`  
  Specifies CSV format options when `FORMAT AS` is set to `CSV`. Syntax:
  ```JSON
  (
      key = value
      key = value
      ...
  )
  ```  
  > **NOTE**  
  > `format_type_options` is supported in v3.0 and later.  
  The following table describes the options.
  | **Parameter** | **Description** |
  | ------------- | ------------------------------------------------------------ |
  | skip_header   | Specifies whether to skip the first rows of the data file when the data file is in CSV format. Type: INTEGER. Default value: `0`.<br />In some CSV-formatted data files, the first rows at the beginning are used to define metadata such as column names and column data types. By setting the `skip_header` parameter, you can enable StarRocks to skip the first rows of the data file during data loading. For example, if you set this parameter to `1`, StarRocks skips the first row of the data file during data loading.<br />The first rows at the beginning in the data file must be separated by using the row separator that you specify in the load statement. |
  | trim_space    | Specifies whether to remove spaces preceding and following column separators from the data file when the data file is in CSV format. Type: BOOLEAN. Default value: `false`.<br />For some databases, spaces are added to column separators when you export data as a CSV-formatted data file. Such spaces are called leading spaces or trailing spaces depending on their locations. By setting the `trim_space` parameter, you can enable StarRocks to remove such unnecessary spaces during data loading.<br />Note that StarRocks does not remove the spaces (including leading spaces and trailing spaces) within a field wrapped in a pair of `enclose`-specified characters. For example, the following field values use pipe (<code class="language-text">&#124;</code>) as the column separator and double quotation marks (`"`) as the `enclose`-specified character:<br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> <br /><code class="language-text">&#124;" Love StarRocks "&#124;</code> <br /><code class="language-text">&#124; "Love StarRocks" &#124;</code> <br />If you set `trim_space` to `true`, StarRocks processes the preceding field values as follows:<br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> <br /><code class="language-text">&#124;" Love StarRocks "&#124;</code> <br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> |
  | enclose       | Specifies the character that is used to wrap the field values in the data file according to [RFC4180](https://www.rfc-editor.org/rfc/rfc4180) when the data file is in CSV format. Type: single-byte character. Default value: `NONE`. The most prevalent characters are single quotation mark (`'`) and double quotation mark (`"`).<br />All special characters (including row separators and column separators) wrapped by using the `enclose`-specified character are considered normal symbols. StarRocks can do more than RFC4180 as it allows you to specify any single-byte character as the `enclose`-specified character.<br />If a field value contains an `enclose`-specified character, you can use the same character to escape that `enclose`-specified character. For example, you set `enclose` to `"`, and a field value is `a "quoted" c`. In this case, you can enter the field value as `"a ""quoted"" c"` into the data file. |
  | escape        | Specifies the character that is used to escape various special characters, such as row separators, column separators, escape characters, and `enclose`-specified characters, which are then considered by StarRocks to be common characters and are parsed as part of the field values in which they reside. Type: single-byte character. Default value: `NONE`. The most prevalent character is slash (`\`), which must be written as double slashes (`\\`) in SQL statements.<br />**NOTE**<br />The character specified by `escape` is applied to both inside and outside of each pair of `enclose`-specified characters.<br />Two examples are as follows:<ul><li>When you set `enclose` to `"` and `escape` to `\`, StarRocks parses `"say \"Hello world\""` into `say "Hello world"`.</li><li>Assume that the column separator is comma (`,`). When you set `escape` to `\`, StarRocks parses `a, b\, c` into two separate field values: `a` and `b, c`.</li></ul> |
- `column_list`  
  Specifies the column mapping between the data file and the StarRocks table. Syntax: `(<column_name>[, <column_name> ...])`. The columns declared in `column_list` are mapped by name onto the StarRocks table columns.  
  > **NOTE**  
  > If the columns of the data file are mapped in sequence onto the columns of the StarRocks table, you do not need to specify `column_list`.  
  If you want to skip a specific column of the data file, you only need to temporarily name that column as different from any of the StarRocks table columns. For more information, see [Transform data at loading](../../../loading/Etl_in_loading.md).
- `COLUMNS FROM PATH AS`  
  Extracts the information about one or more partition fields from the file path you specify. This parameter is valid only when the file path contains partition fields.  
  For example, if the data file is stored in the path `/path/col_name=col_value/file1` in which `col_name` is a partition field and can be mapped onto a column of the StarRocks table, you can specify this parameter as `col_name`. As such, StarRocks extracts `col_value` values from the path and loads them into the StarRocks table column onto which `col_name` is mapped.  
  > **NOTE**  
  > This parameter is available only when you load data from HDFS.
- `SET`  
  Specifies one or more functions that you want to use to convert a column of the data file. Examples:  
  - The StarRocks table consists of three columns, which are `col1`, `col2`, and `col3` in sequence. The data file consists of four columns, among which the first two columns are mapped in sequence onto `col1` and `col2` of the StarRocks table and the sum of the last two columns is mapped onto `col3` of the StarRocks table. In this case, you need to specify `column_list` as `(col1,col2,tmp_col3,tmp_col4)` and specify `(col3=tmp_col3+tmp_col4)` in the SET clause to implement data conversion.  
  - The StarRocks table consists of three columns, which are `year`, `month`, and `day` in sequence. The data file consists of only one column that accommodates date and time values in `yyyy-mm-dd hh:mm:ss` format. In this case, you need to specify `column_list` as `(tmp_time)` and specify `(year = year(tmp_time), month=month(tmp_time), day=day(tmp_time))` in the SET clause to implement data conversion.
- `WHERE`  
  Specifies the conditions based on which you want to filter the source data. StarRocks loads only the source data that meets the filter conditions specified in the WHERE clause.  
### WITH BROKER  
In v2.3 and earlier, input `WITH BROKER "<broker_name>"` to specify the broker you want to use. From v2.5 onwards, you no longer need to specify a broker, but you still need to retain the `WITH BROKER` keyword.  
> **NOTE**
+ {R}
  + {R}
    + {R}
      + {R}
        + {R}
        + {R}

- インスタンスプロファイルベースの認証方法を選択するには、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 仮定されるロールベースの認証方法を選択するには、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAMユーザーベースの認証方法を選択するには、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

以下の表には、`StorageCredentialParams` で構成する必要があるパラメータが記載されています。

| パラメータ                       | 必須 | 説明                                                         |
| ------------------------------- | ---- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile    | Yes  | 認証方法としてのインスタンスプロファイルと仮定されるロールの有効化を指定します。有効な値: `true` および `false`。デフォルト値: `false`。 |
| aws.s3.iam_role_arn             | No   | AWS S3 バケットに特権を持つ IAM ロールのARN。AWS S3 へのアクセスに仮定されるロールを認証方法として選択した場合、このパラメータを指定する必要があります。 |
| aws.s3.region                   | Yes  | AWS S3 バケットが存在するリージョン。例: `us-west-1`。      |
| aws.s3.access_key               | No   | IAM ユーザーのアクセスキー。AWS S3 へのアクセスに IAM ユーザーを認証方法として選択した場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key               | No   | IAM ユーザーのシークレットキー。AWS S3 へのアクセスに IAM ユーザーを認証方法として選択した場合、このパラメータを指定する必要があります。 |

AWS S3 へのアクセスの認証方法を選択する手順や AWS IAM コンソールでアクセス制御ポリシーを構成する手順については、[AWS S3へのアクセスの認証パラメータ](../../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3) を参照してください。

#### Google GCS

ストレージシステムとして Google GCS を選択した場合は、次のいずれかのアクションを実行してください：

- VMベースの認証方法を選択するには、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  以下の表には、`StorageCredentialParams` で構成する必要があるパラメータが記載されています。

  | **Parameter**                              | **Default value** | **Value** **example** | **Description**                                              |
  | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | false             | true                  | 直接 Compute Engine にバインドされているサービスアカウントを使用するかどうかを指定します。 |

- サービスアカウントベースの認証方法を選択するには、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  以下の表には、`StorageCredentialParams` で構成する必要があるパラメータが記載されています。

  | **Parameter**                          | **Default value** | **Value** **example**                                        | **Description**                                              |
  | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウントの作成時に生成されたJSONファイル内の電子メールアドレス。 |
  | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | サービスアカウントの作成時に生成されたJSONファイル内のプライベートキーID。 |
  | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | サービスアカウントの作成時に生成されたJSONファイル内の秘密キー。 |

- インパーソネーションベースの認証方法を選択するには、`StorageCredentialParams` を次のように構成します。

  - VMインスタンスにサービスアカウントの権限を委任する場合：

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    以下の表には、`StorageCredentialParams` で構成する必要があるパラメータが記載されています。

    | **Parameter**                              | **Default value** | **Value** **example** | **Description**                                              |
    | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false             | true                  | 直接 Compute Engine にバインドされているサービスアカウントを使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""                | "hello"               | 委任したいサービスアカウント。                               |

  - サービスアカウント（メタサービスアカウントと呼ばれます）が別のサービスアカウント（データサービスアカウントと呼ばれます）を偽装する場合：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    以下の表には、`StorageCredentialParams` で構成する必要があるパラメータが記載されています。

    | **Parameter**                          | **Default value** | **Value** **example**                                        | **Description**                                              |
    | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | メタサービスアカウントの作成時に生成されたJSONファイル内の電子メールアドレス。 |
    | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | メタサービスアカウントの作成時に生成されたJSONファイル内のプライベートキーID。 |
    | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | メタサービスアカウントの作成時に生成されたJSONファイル内の秘密キー。 |
    | gcp.gcs.impersonation_service_account  | ""                | "hello"                                                       | 偽装したいデータサービスアカウント。                        |

#### その他のS3互換のストレージシステム

MinIOなどのその他のS3互換のストレージシステムを選択する場合、`StorageCredentialParams` を次のように構成します。

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下の表には、`StorageCredentialParams` で構成する必要があるパラメータが記載されています。

| パラメータ                       | 必須 | 説明                                                         |
| ------------------------------- | ---- | ------------------------------------------------------------ |
| aws.s3.enable_ssl               | Yes  | SSL接続を有効にするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `true`。 |
| aws.s3.enable_path_style_access | Yes  | パス形式のURLアクセスを有効にするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `false`。MinIOの場合は、値を`true`に設定する必要があります。 |
| aws.s3.endpoint                 | Yes  | AWS S3の代わりにS3互換のストレージシステムに接続するために使用されるエンドポイント。 |
| aws.s3.access_key               | Yes  | IAMユーザーのアクセスキー。                                  |
| aws.s3.secret_key               | Yes  | IAMユーザーのシークレットキー。                              |

#### Microsoft Azure Storage

##### Azure Blob Storage

Blob Storageをストレージシステムとして選択した場合、次のいずれかのアクションを実行してください：

- 共有キー認証方法を選択するには、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  以下の表には、`StorageCredentialParams` で構成する必要があるパラメータが記載されています。

  | **Parameter**              | **Required** | **Description**                              |
  | -------------------------- | ------------ | -------------------------------------------- |
  | azure.blob.storage_account | Yes          | Blob Storageアカウントのユーザー名。              |
  | azure.blob.shared_key      | Yes          | Blob Storageアカウントの共有キー。              |

- SAS トークン認証方法を選択するには、`StorageCredentialParams` を次のように構成します：

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  ```
```json
"azure.blob.container_name" = "<blob_container_name>",
"azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
```

`StorageCredentialParams`に設定する必要のあるパラメータについて、以下の表に記載します。

| **パラメータ**           | **必須** | **説明**                                                      |
| ----------------------- | -------- | -------------------------------------------------------------- |
| azure.blob.account_name | Yes      | Blob Storageアカウントのユーザー名。                            |
| azure.blob.container_name | Yes    | データを格納するBlobコンテナの名前。                           |
| azure.blob.sas_token      | Yes    | Blob Storageアカウントにアクセスするために使用されるSASトークン。|

##### Azure Data Lake Storage Gen1

ストレージシステムとしてData Lake Storage Gen1を選択する場合、次のいずれかのアクションを実行します。

- Managed Service Identity認証メソッドを選択する場合、`StorageCredentialParams`を次のように構成します。

```SQL
"azure.adls1.use_managed_service_identity" = "true"
```

`StorageCredentialParams`に設定する必要のあるパラメータについて、以下の表に記載します。

| **パラメータ**                            | **必須** | **説明**                                                      |
| ---------------------------------------- | -------- | -------------------------------------------------------------- |
| azure.adls1.use_managed_service_identity | Yes      | Managed Service Identity認証メソッドを有効にするかどうかを指定します。値を`true`に設定します。|

- Service Principal認証メソッドを選択する場合、`StorageCredentialParams`を次のように構成します。

```SQL
"azure.adls1.oauth2_client_id" = "<application_client_id>",
"azure.adls1.oauth2_credential" = "<application_client_credential>",
"azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
```

`StorageCredentialParams`に設定する必要のあるパラメータについて、以下の表に記載します。

| **パラメータ**                 | **必須** | **説明**                                                      |
| ----------------------------- | -------- | -------------------------------------------------------------- |
| azure.adls1.oauth2_client_id  | Yes      | .のクライアント（アプリケーション）ID。                        |
| azure.adls1.oauth2_credential | Yes      | 作成された新しいクライアント（アプリケーション）シークレットの値。 |
| azure.adls1.oauth2_endpoint   | Yes      | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）。|

##### Azure Data Lake Storage Gen2

ストレージシステムとしてData Lake Storage Gen2を選択する場合、次のいずれかのアクションを実行します。

- Managed Identity認証メソッドを選択する場合、`StorageCredentialParams`を次のように構成します。

```SQL
"azure.adls2.oauth2_use_managed_identity" = "true",
"azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
"azure.adls2.oauth2_client_id" = "<service_client_id>"
```

`StorageCredentialParams`に設定する必要のあるパラメータについて、以下の表に記載します。

| **パラメータ**                           | **必須** | **説明**                                                      |
| --------------------------------------- | -------- | -------------------------------------------------------------- |
| azure.adls2.oauth2_use_managed_identity | Yes      | Managed Identity認証メソッドを有効にするかどうかを指定します。値を`true`に設定します。 |
| azure.adls2.oauth2_tenant_id            | Yes      | アクセスするデータのテナントのID。                             |
| azure.adls2.oauth2_client_id            | Yes      | マネージドアイデンティティのクライアント（アプリケーション）ID。|

- Shared Key認証メソッドを選択する場合、`StorageCredentialParams`を次のように構成します。

```SQL
"azure.adls2.storage_account" = "<storage_account_name>",
"azure.adls2.shared_key" = "<shared_key>"
```

`StorageCredentialParams`に設定する必要のあるパラメータについて、以下の表に記載します。

| **パラメータ**               | **必須** | **説明**                                                      |
| --------------------------- | -------- | -------------------------------------------------------------- |
| azure.adls2.storage_account | Yes      | Data Lake Storage Gen2ストレージアカウントのユーザー名。        |
| azure.adls2.shared_key      | Yes      | Data Lake Storage Gen2ストレージアカウントの共有キー。           |

- Service Principal認証メソッドを選択する場合、`StorageCredentialParams`を次のように構成します。

```SQL
"azure.adls2.oauth2_client_id" = "<service_client_id>",
"azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
"azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
```

`StorageCredentialParams`に設定する必要のあるパラメータについて、以下の表に記載します。

| **パラメータ**                      | **必須** | **説明**                                                      |
| ---------------------------------- | -------- | -------------------------------------------------------------- |
| azure.adls2.oauth2_client_id       | Yes      | Service Principalのクライアント（アプリケーション）ID。        |
| azure.adls2.oauth2_client_secret   | Yes      | 作成された新しいクライアント（アプリケーション）シークレットの値。 |
| azure.adls2.oauth2_client_endpoint | Yes      | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）。|

### opt_properties

ロードジョブ全体に適用されるいくつかのオプションパラメータを指定します。構文:

```Plain
PROPERTIES ("<key1>" = "<value1>"[, "<key2>" = "<value2>" ...])
```

以下のパラメータがサポートされています:

- `timeout`

  ロードジョブのタイムアウト期間を指定します。単位: 秒。デフォルトのタイムアウト期間は4時間です。6時間より短いタイムアウト期間を指定することをお勧めします。ロードジョブがタイムアウト期間内に完了しない場合、StarRocksはロードジョブをキャンセルし、ロードジョブのステータスは**CANCELLED**になります。

  > **注記**
  >
  > ほとんどの場合、タイムアウト期間を設定する必要はありません。デフォルトのタイムアウト期間内にロードジョブが完了しない場合のみ、タイムアウト期間を設定することをお勧めします。

  タイムアウト期間を推定するために、次の式を使用します:

  **タイムアウト期間 > (ロードするデータファイルの合計サイズ x ロードするデータファイルの合計数と、データファイルに作成されたマテリアライズドビューの合計数)/平均ロード速度**

  > **注記**
  >
  > "平均ロード速度"は、StarRocksクラスタ全体の平均ロード速度です。平均ロード速度は、クラスタごとにサーバー構成とクラスタで許可される最大同時問合せタスク数に応じて異なります。過去のロードジョブのロード速度に基づいて、平均ロード速度を推定できます。

  例えば、平均ロード速度が10 MB/sであるStarRocksクラスタに、2つのマテリアライズドビューが作成された1GBのデータファイルをロードしたい場合、データのロードにかかる時間の目安は約102秒です。

  (1 x 1024 x 3)/10 = 307.2 (秒)

  この例では、タイムアウト期間を308秒より大きい値に設定することをお勧めします。

- `max_filter_ratio`

  ロードジョブの最大エラートレランスを指定します。最大エラートレランスは、データ品質が不十分であるために除外される行の最大パーセンテージです。有効な値: `0`〜`1`。デフォルト値: `0`。

  - このパラメータを`0`に設定すると、StarRocksはロード中に資格のない行を無視しません。そのため、ソースデータに資格のない行が含まれている場合、ロードジョブは失敗します。これにより、StarRocksにロードされるデータの正確性が確保されます。

  - このパラメータを`0`より大きい値に設定すると、StarRocksはロード中に資格のない行を無視できます。そのため、ソースデータに資格のない行が含まれていても、ロードジョブは成功することがあります。

    > **注記**
    >
    > データ品質が不十分なために除外された行には、WHERE句によって除外された行は含まれません。

  ロードジョブが最大エラートレランスを`0`に設定したために失敗した場合は、[SHOW LOAD](../../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)を使用してジョブの結果を表示し、資格のない行を除外できるかどうかを判断します。資格のない行を除外できる場合は、ジョブ結果で返された`dpp.abnorm.ALL`および`dpp.norm.ALL`の値に基づいて、最大エラートレランスを計算し直し、最大エラートレランスを調整してロードジョブを再送信します。最大エラートレランスを計算するための式は次のとおりです:

  `max_filter_ratio` = [`dpp.abnorm.ALL`/(`dpp.abnorm.ALL` + `dpp.norm.ALL`)]

  `dpp.abnorm.ALL`および`dpp.norm.ALL`の値の合計は、ロードする行の合計数です。

- `log_rejected_record_num`

  ログに記録できる最大の資格のないデータ行の数を指定します。このパラメータはv3.1からサポートされています。有効な値: `0`、`-1`、および正の整数。デフォルト値: `0`。
  
  - 値`0`は、除外されたデータ行をログに記録しません。
  - 値`-1`は、除外されたすべてのデータ行をログに記録します。
  - `n`のような正の非ゼロの整数は、BEごとに最大で`n`個の除外されたデータ行をログに記録できます。
```
      + Specifies the maximum amount of memory that can be provided to the load job. Unit: bytes: The default memory limit is 2 GB.
      + `strict_mode`
          + Specifies whether to enable the [strict mode](../../../loading/load_concept/strict_mode.md). Valid values: `true` and `false`. Default value: `false`. `true` specifies to enable the strict mode, and `false` specifies to disable the strict mode.
      + `timezone`
          + Specifies the time zone of the load job. Default value: `Asia/Shanghai`. The time zone setting affects the results returned by functions such as strftime, alignment_timestamp, and from_unixtime. For more information, see [Configure a time zone](../../../administration/timezone.md). The time zone specified in the `timezone` parameter is a session-level time zone.
      + `priority`
          + Specifies the priority of the load job. Valid values: `LOWEST`, `LOW`, `NORMAL`, `HIGH`, and `HIGHEST`. Default value: `NORMAL`. Broker Load provides the [FE parameter](../../../administration/Configuration.md#fe-configuration-items) `max_broker_load_job_concurrency`, determines the maximum number of Broker Load jobs that can be concurrently run within your StarRocks cluster. If the number of Broker Load jobs that are submitted within the specified time period exceeds the maximum number, excessive jobs will be waiting to be scheduled based on their priorities.
You can use the [ALTER LOAD](../../../sql-reference/sql-statements/data-manipulation/ALTER_LOAD.md) statement to change the priority of an existing load job that is in the `QUEUEING` or `LOADING` state.
StarRocks allows setting the `priority` parameter for a Broker Load job since v2.5.
## Column mapping
If the columns of the data file can be mapped one on one in sequence to the columns of the StarRocks table, you do not need to configure the column mapping between the data file and the StarRocks table.
If the columns of the data file cannot be mapped one on one in sequence to the columns of the StarRocks table, you need to use the `columns` parameter to configure the column mapping between the data file and the StarRocks table. This includes the following two use cases:
- **Same number of columns but different column sequence.** **Also, the data from the data file does not need to be computed by functions before it is loaded into the matching StarRocks table columns.**
  In the `columns` parameter, you need to specify the names of the StarRocks table columns in the same sequence as how the data file columns are arranged.
  For example, the StarRocks table consists of three columns, which are `col1`, `col2`, and `col3` in sequence, and the data file also consists of three columns, which can be mapped to the StarRocks table columns `col3`, `col2`, and `col1` in sequence. In this case, you need to specify `"columns: col3, col2, col1"`.
- **Different number of columns and different column sequence. Also, the data from the data file needs to be computed by functions before it is loaded into the matching StarRocks table columns.**
  In the `columns` parameter, you need to specify the names of the StarRocks table columns in the same sequence as how the data file columns are arranged and specify the functions you want to use to compute the data. Two examples are as follows:
  - The StarRocks table consists of three columns, which are `col1`, `col2`, and `col3` in sequence. The data file consists of four columns, among which the first three columns can be mapped in sequence to the StarRocks table columns `col1`, `col2`, and `col3` and the fourth column cannot be mapped to any of the StarRocks table columns. In this case, you need to temporarily specify a name for the fourth column of the data file, and the temporary name must be different from any of the StarRocks table column names. For example, you can specify `"columns: col1, col2, col3, temp"`, in which the fourth column of the data file is temporarily named `temp`.
  - The StarRocks table consists of three columns, which are `year`, `month`, and `day` in sequence. The data file consists of only one column that accommodates date and time values in `yyyy-mm-dd hh:mm:ss` format. In this case, you can specify `"columns: col, year = year(col), month=month(col), day=day(col)"`, in which `col` is the temporary name of the data file column and the functions `year = year(col)`, `month=month(col)`, and `day=day(col)` are used to extract data from the data file column `col` and loads the data into the mapping StarRocks table columns. For example, `year = year(col)` is used to extract the `yyyy` data from the data file column `col` and loads the data into the StarRocks table column `year`.
For detailed examples, see [Configure column mapping](#configure-column-mapping).
## Related configuration items
The [FE configuration item](../../../administration/Configuration.md#fe-configuration-items) `max_broker_load_job_concurrency` specifies the maximum number of Broker Load jobs that can be concurrently run within your StarRocks cluster.
In StarRocks v2.4 and earlier, if the total number of Broker Load jobs that are submitted within a specific period of time exceeds the maximum number, excessive jobs will be queued and scheduled based on their submission time.
Since StarRocks v2.5, if the total number of Broker Load jobs that are submitted within a specific period of time exceeds the maximum number, excessive jobs are queued and scheduled based on their priorities. You can specify a priority for a job by using the `priority` parameter described above. You can use [ALTER LOAD](../data-manipulation/ALTER_LOAD.md) to modify the priority of an existing job that is in the **QUEUEING** or **LOADING** state.
## Examples
This section uses HDFS as an example to describe various load configurations.
### Load CSV data
This section uses CSV as an example to explain the various parameter configurations that you can use to meet your diverse load requirements.
#### Set timeout period
Your StarRocks database `test_db` contains a table named `table1`. The table consists of three columns, which are `col1`, `col2`, and `col3` in sequence.
Your data file `example1.csv` also consists of three columns, which are mapped in sequence onto `col1`, `col2`, and `col3` of `table1`.
If you want to load all data from `example1.csv` into `table1` within up to 3600 seconds, run the following command:
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
#### Set maximum error tolerance
Your StarRocks database `test_db` contains a table named `table2`. The table consists of three columns, which are `col1`, `col2`, and `col3` in sequence.
Your data file `example2.csv` also consists of three columns, which are mapped in sequence onto `col1`, `col2`, and `col3` of `table2`.
If you want to load all data from `example2.csv` into `table2` with a maximum error tolerance of `0.1`, run the following command:
```SQL
LOAD LABEL test_db.label2
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example2.csv")
    INTO TABLE table2
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" ="<hdfs_password>"
)
PROPERTIES
(
    "max_filter_ratio" = "0.1"
);
```
#### Load all data files from a file path
Your StarRocks database `test_db` contains a table named `table3`. The table consists of three columns, which are `col1`, `col2`, and `col3` in sequence.
All data files stored in the `/user/starrocks/data/input/` path of your HDFS cluster also each consist of three columns, which are mapped in sequence onto `col1`, `col2`, and `col3` of `table3`. The column separator used in these data files is `\x01`.
If you want to load data from all these data files stored in the `hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/` path into `table3`, run the following command:
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
#### Set NameNode HA mechanism
```
あなたのStarRocksデータベース`test_db`には`table4`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

データファイル`example4.csv`もまた3つの列で構成されており、これらは`table4`の`col1`、`col2`、`col3`にそれぞれマッピングされています。

`NameNode`にHAメカニズムが構成された状態で`example4.csv`からすべてのデータを`table4`にロードしたい場合、次のコマンドを実行します。

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

#### Kerberos認証の設定

あなたのStarRocksデータベース`test_db`には`table5`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

データファイル`example5.csv`もまた3つの列で構成されており、これらは`table5`の`col1`、`col2`、`col3`にそれぞれマッピングされています。

Kerberos認証が構成され、キータブファイルのパスが指定された状態で`example5.csv`からすべてのデータを`table5`にロードしたい場合、次のコマンドを実行します。

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

あなたのStarRocksデータベース`test_db`には`table6`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

データファイル`example6.csv`もまた3つの列で構成されており、これらは`table6`の`col1`、`col2`、`col3`にそれぞれマッピングされています。

あなたはすでに`example6.csv`からすべてのデータを`table6`にロードした状態です。

ロードしたデータを取り消したい場合、次のコマンドを実行します。

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

あなたのStarRocksデータベース`test_db`には`table7`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

データファイル`example7.csv`もまた3つの列で構成されており、これらは`table7`の`col1`、`col2`、`col3`にそれぞれマッピングされています。

`example7.csv`からすべてのデータを`table7`の`p1`と`p2`の2つのパーティションにロードしたい場合、次のコマンドを実行します。

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

#### 列マッピングの構成

あなたのStarRocksデータベース`test_db`には`table8`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

データファイル`example8.csv`もまた3つの列で構成されており、これらは`table8`の`col2`、`col1`、`col3`にそれぞれマッピングされています。

`example8.csv`からすべてのデータを`table8`にロードしたい場合、次のコマンドを実行します。

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
> 上記の例では、`example8.csv`の列は、`table8`に配置されている列の順序と同じ順序でマッピングされていません。そのため、`column_list`を使用して`example8.csv`と`table8`の間の列マッピングを構成する必要があります。

#### フィルタ条件の設定

あなたのStarRocksデータベース`test_db`には`table9`という名前のテーブルが含まれています。このテーブルは、`col1`、`col2`、`col3`の3つの列で構成されています。

データファイル`example9.csv`もまた3つの列で構成されており、これらは`table9`の`col1`、`col2`、`col3`にそれぞれマッピングされています。

`example9.csv`から`col1`の値が`20180601`より大きいデータレコードのみを`table9`にロードしたい場合、次のコマンドを実行します。

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
> 上記の例では、`example9.csv`の列は`table9`の列と同じ順序でマッピングされていますが、列ベースのフィルタ条件を指定するためにWHERE句を使用する必要があります。そのため、`column_list`を使用して`example9.csv`と`table9`の間の列マッピングを構成する必要があります。

#### HLLタイプの列を含むテーブルへのデータロード

あなたのStarRocksデータベース`test_db`には`table10`という名前のテーブルが含まれています。このテーブルは、`id`、`col1`、`col2`、`col3`の4つの列で構成されています。`col1`と`col2`はHLLタイプの列として定義されています。

データファイル`example10.csv`は3つの列で構成されており、最初の列が`table10`の`id`にマッピングされ、2番目と3番目の列が`table10`の`col1`と`col2`に順番にマッピングされています。`example10.csv`の2番目と3番目の列の値は、`table10`の`col1`および`col2`にロードされる前に、関数を使用してHLLタイプのデータに変換することができます。

`example10.csv`からすべてのデータを`table10`にロードしたい場合、次のコマンドを実行します。

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
> 上記の例では、`example10.csv`の3つの列は`column_list`を使用してそれぞれ`id`、`temp1`、`temp2`として命名されました。その後、以下のように関数を使用してデータを変換しています：
>
> - `hll_hash`関数を使用して`example10.csv`の`temp1`および`temp2`の値をHLLタイプのデータに変換し、それらを`table10`の`col1`および`col2`にマッピングしています。
- `hll_empty` 関数は `table10` の `col3` に指定されたデフォルト値を埋め込むために使用されます。

関数 `hll_hash` および `hll_empty` の使用方法については、[hll_hash](../../sql-functions/aggregate-functions/hll_hash.md) および [hll_empty](../../sql-functions/aggregate-functions/hll_empty.md) を参照してください。

#### ファイルパスから抽出されたパーティションフィールドの値

ブローカーロードは、StarRocks テーブルの列定義に基づいてファイルパスに含まれる特定のパーティションフィールドの値を解析することをサポートしています。これは Apache Spark™ の Partition Discovery 機能に類似した StarRocks の機能です。

あなたの StarRocks データベース `test_db` には `table11` という名前のテーブルが含まれています。このテーブルは `col1`、`col2`、`col3`、`city`、`utc_date` の 5 つの列で構成されています。

あなたの HDFS クラスタのファイルパス `/user/starrocks/data/input/dir/city=beijing` には、以下のデータファイルが含まれています：

- `/user/starrocks/data/input/dir/city=beijing/utc_date=2019-06-26/0000.csv`
- `/user/starrocks/data/input/dir/city=beijing/utc_date=2019-06-26/0001.csv`

これらのデータファイルはそれぞれ、`table11` の `col1`、`col2`、`col3` に順にマップされる 3 つの列で構成されています。

もし、ファイルパス `/user/starrocks/data/input/dir/city=beijing/utc_date=*/*` からすべてのデータファイルを `table11` に読み込みたい場合、同時にファイルパスに含まれるパーティションフィールド `city` および `utc_date` の値を抽出して `table11` の `city` および `utc_date` に読み込みたい場合は、次のコマンドを実行してください：

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

#### `%3A` がファイルパスに含まれる場合のパーティションフィールドの値の抽出

HDFS では、ファイルパスにはコロン（:）を含めることができません。すべてのコロン（:）は `%3A` に変換されます。

あなたの StarRocks データベース `test_db` には `table12` という名前のテーブルが含まれています。このテーブルは `data_time`、`col1`、`col2` の 3 つの列で構成されています。テーブルのスキーマは以下の通りです：

```SQL
data_time DATETIME,
col1        INT,
col2        INT
```

あなたの HDFS クラスタのファイルパス `/user/starrocks/data` には、以下のデータファイルが含まれています：

- `/user/starrocks/data/data_time=2020-02-17 00%3A00%3A00/example12.csv`
- `/user/starrocks/data/data_time=2020-02-18 00%3A00%3A00/example12.csv`

もし、`example12.csv` のデータをすべて `table12` に読み込みたい場合、同時にファイルパスからのパーティションフィールド `data_time` の値を抽出して `table12` の `data_time` に読み込みたい場合は、次のコマンドを実行してください：

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

> **注記**
>
> 前述の例では、パーティションフィールド `data_time` から抽出された値は `%3A` を含む文字列です（たとえば `2020-02-17 00%3A00%3A00`）。そのため、これらの値を `table8` の `data_time` に読み込む前に、`str_to_date` 関数を使用して文字列を DATETIME 型のデータに変換する必要があります。

#### `format_type_options` の設定

あなたの StarRocks データベース `test_db` には `table13` という名前のテーブルが含まれています。このテーブルは `col1`、`col2`、`col3` の 3 つの列で構成されています。

あなたのデータファイル `example13.csv` もまた、同様に `table13` の `col2`、`col1`、`col3` に順にマップされる 3 つの列で構成されています。

もし、`example13.csv` のすべてのデータを `table13` に読み込むため、`example13.csv` の最初の 2 行をスキップし、列区切り記号の前後の空白を削除し、`enclose` を `\` に設定し、`escape` を `\` に設定したい場合は、次のコマンドを実行してください：

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

### Parquet データの読み込み

このセクションでは、Parquet データの読み込む際に注意する必要のあるいくつかのパラメータ設定について説明します。

あなたの StarRocks データベース `test_db` には `table13` という名前のテーブルが含まれています。このテーブルは `col1`、`col2`、`col3` の 3 つの列で構成されています。

あなたのデータファイル `example13.parquet` もまた、同様に `table13` の `col1`、`col2`、`col3` に順にマップされる 3 つの列で構成されています。

もし、`example13.parquet` のすべてのデータを `table13` に読み込みたい場合は、次のコマンドを実行してください：

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
> デフォルトでは、Parquet データを読み込む場合、StarRocks はファイル名に拡張子 **.parquet** が含まれているかどうかに基づいてデータファイルの形式を決定します。ファイル名に拡張子 **.parquet** が含まれていない場合、`FORMAT AS` を使用してデータファイル形式を `Parquet` として指定する必要があります。

### ORC データの読み込み

このセクションでは、ORC データの読み込む際に注意する必要のあるいくつかのパラメータ設定について説明します。

あなたの StarRocks データベース `test_db` には `table14` という名前のテーブルが含まれています。このテーブルは `col1`、`col2`、`col3` の 3 つの列で構成されています。

あなたのデータファイル `example14.orc` はまた、`table14` の `col1`、`col2`、`col3` に順にマップされる 3 つの列で構成されています。

もし、`example14.orc` のすべてのデータを `table14` に読み込みたい場合は、次のコマンドを実行してください：

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
> - デフォルトでは、ORC データを読み込む場合、StarRocks はファイル名に拡張子 **.orc** が含まれているかどうかに基づいてデータファイルの形式を決定します。ファイル名に拡張子 **.orc** が含まれていない場合、`FORMAT AS` を使用してデータファイル形式を `ORC` として指定する必要があります。
> - StarRocks v2.3 およびそれ以前のバージョンでは、データファイルに ARRAY 型の列が含まれている場合、ORC データファイルの列が StarRocks テーブルでのマッピング列と同じ名前を持っていること、および列が SET 句で指定されていないことを確認する必要があります。