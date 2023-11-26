---
displayed_sidebar: "Japanese"
---

# FILES

## 説明

リモートストレージ内のデータファイルを定義します。

v3.1.0以降、StarRocksはテーブル関数FILES()を使用して、リモートストレージ内の読み取り専用ファイルを定義することができます。FILES()は、ファイルのパス関連のプロパティを使用してリモートストレージにアクセスし、ファイル内のデータのテーブルスキーマを推定し、データ行を返します。[SELECT](../../sql-statements/data-manipulation/SELECT.md)を使用してデータ行を直接クエリできますし、[INSERT](../../sql-statements/data-manipulation/INSERT.md)を使用して既存のテーブルにデータ行をロードすることもできますし、[CREATE TABLE AS SELECT](../../sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)を使用して新しいテーブルを作成し、データ行をロードすることもできます。

v3.2.0以降、FILES()はリモートストレージ内の書き込み可能なデータファイルを定義することもサポートしています。[INSERT INTO FILES()を使用してStarRocksからリモートストレージにデータをアンロード](../../../unloading/unload_using_insert_into_files.md)することができます。

現在、FILES()関数は次のデータソースとファイル形式をサポートしています。

- **データソース:**
  - HDFS
  - AWS S3
  - Google Cloud Storage
  - その他のS3互換ストレージシステム
  - Microsoft Azure Blob Storage
- **ファイル形式:**
  - Parquet
  - ORC（データのアンロードには現在サポートされていません）

## 構文

```SQL
FILES( data_location , data_format [, StorageCredentialParams ] 
    [, columns_from_path ] [, unload_data ] )

data_location ::=
    "path" = { "hdfs://<hdfs_host>:<hdfs_port>/<hdfs_path>"
             | "s3://<s3_path>" 
             | "s3a://<gcs_path>" 
             | "wasb://<container>@<storage_account>.blob.core.windows.net/<blob_path>"
             | "wasbs://<container>@<storage_account>.blob.core.windows.net/<blob_path>"
             }

data_format ::=
    "format" = { "parquet" | "orc" }

-- v3.2以降でサポートされています。
columns_from_path ::=
    "columns_from_path" = "<column_name> [, ...]"

-- v3.2以降でサポートされています。
unload_data::=
    "compression" = "<compression_method>"
    [, "max_file_size" = "<file_size>" ]
    [, "partition_by" = "<column_name> [, ...]" ]
    [, "single" = { "true" | "false" } ]
```

## パラメータ

すべてのパラメータは`"key" = "value"`のペアで指定します。

### data_location

ファイルにアクセスするために使用するURIです。パスまたはファイルを指定できます。

- HDFSにアクセスする場合は、次のようにこのパラメータを指定する必要があります:

  ```SQL
  "path" = "hdfs://<hdfs_host>:<hdfs_port>/<hdfs_path>"
  -- 例: "path" = "hdfs://127.0.0.1:9000/path/file.parquet"
  ```

- AWS S3にアクセスする場合:

  - S3プロトコルを使用する場合は、次のようにこのパラメータを指定する必要があります:

    ```SQL
    "path" = "s3://<s3_path>"
    -- 例: "path" = "s3://path/file.parquet"
    ```

  - S3Aプロトコルを使用する場合は、次のようにこのパラメータを指定する必要があります:

    ```SQL
    "path" = "s3a://<s3_path>"
    -- 例: "path" = "s3a://path/file.parquet"
    ```

- Google Cloud Storageにアクセスする場合は、次のようにこのパラメータを指定する必要があります:

  ```SQL
  "path" = "s3q://<gcs_path>"
  -- 例: "path" = "s3a://path/file.parquet"
  ```

- Azure Blob Storageにアクセスする場合:

  - ストレージアカウントがHTTP経由でアクセスを許可している場合は、次のようにこのパラメータを指定する必要があります:

    ```SQL
    "path" = "wasb://<container>@<storage_account>.blob.core.windows.net/<blob_path>"
    -- 例: "path" = "wasb://testcontainer@testaccount.blob.core.windows.net/path/file.parquet"
    ```
  
  - ストレージアカウントがHTTPS経由でアクセスを許可している場合は、次のようにこのパラメータを指定する必要があります:

    ```SQL
    "path" = "wasbs://<container>@<storage_account>.blob.core.windows.net/<blob_path>"
    -- 例: "path" = "wasbs://testcontainer@testaccount.blob.core.windows.net/path/file.parquet"
    ```

### data_format

データファイルの形式です。有効な値は`parquet`および`orc`です。

### StorageCredentialParams

StarRocksがストレージシステムにアクセスするために使用する認証情報です。

StarRocksは現在、シンプル認証を使用してHDFSにアクセスし、IAMユーザーベースの認証を使用してAWS S3およびGCSにアクセスし、Shared Keyを使用してAzure Blob Storageにアクセスすることをサポートしています。

- HDFSにアクセスするためのシンプル認証を使用する場合:

  ```SQL
  "hadoop.security.authentication" = "simple",
  "username" = "xxxxxxxxxx",
  "password" = "yyyyyyyyyy"
  ```

  | **キー**                        | **必須** | **説明**                                                     |
  | ------------------------------ | -------- | ------------------------------------------------------------ |
  | hadoop.security.authentication | いいえ   | 認証方法。有効な値: `simple`（デフォルト）。`simple`はシンプル認証を表し、認証なしを意味します。 |
  | username                       | はい     | HDFSクラスタのNameNodeにアクセスするために使用するアカウントのユーザー名。 |
  | password                       | はい     | HDFSクラスタのNameNodeにアクセスするために使用するアカウントのパスワード。 |

- IAMユーザーベースの認証を使用してAWS S3にアクセスする場合:

  ```SQL
  "aws.s3.access_key" = "xxxxxxxxxx",
  "aws.s3.secret_key" = "yyyyyyyyyy",
  "aws.s3.region" = "<s3_region>"
  ```

  | **キー**           | **必須** | **説明**                                                     |
  | ----------------- | -------- | ------------------------------------------------------------ |
  | aws.s3.access_key | はい     | Amazon S3バケットにアクセスするために使用できるアクセスキーID。 |
  | aws.s3.secret_key | はい     | Amazon S3バケットにアクセスするために使用できるシークレットアクセスキー。 |
  | aws.s3.region     | はい     | AWS S3バケットが存在するリージョン。例: `us-west-2`。 |

- IAMユーザーベースの認証を使用してGCSにアクセスする場合:

  ```SQL
  "fs.s3a.access.key" = "xxxxxxxxxx",
  "fs.s3a.secret.key" = "yyyyyyyyyy",
  "fs.s3a.endpoint" = "<gcs_endpoint>"
  ```

  | **キー**           | **必須** | **説明**                                                     |
  | ----------------- | -------- | ------------------------------------------------------------ |
  | fs.s3a.access.key | はい     | GCSバケットにアクセスするために使用できるアクセスキーID。 |
  | fs.s3a.secret.key | はい     | GCSバケットにアクセスするために使用できるシークレットアクセスキー。|
  | fs.s3a.endpoint   | はい     | GCSバケットにアクセスするために使用できるエンドポイント。例: `storage.googleapis.com`。 |

- Shared Keyを使用してAzure Blob Storageにアクセスする場合:

  ```SQL
  "azure.blob.storage_account" = "<storage_account>",
  "azure.blob.shared_key" = "<shared_key>"
  ```

  | **キー**                    | **必須** | **説明**                                                     |
  | -------------------------- | -------- | ------------------------------------------------------------ |
  | azure.blob.storage_account | はい     | Azure Blob Storageアカウントの名前。                          |
  | azure.blob.shared_key      | はい     | Azure Blob Storageアカウントにアクセスするために使用できる共有キー。 |

### columns_from_path

v3.2以降、StarRocksはファイルパスからキー/値ペアの値をカラムの値として抽出することができます。

```SQL
"columns_from_path" = "<column_name> [, ...]"
```

データファイル**file1**が`/geo/country=US/city=LA/`の形式のパスに保存されているとします。ファイルパス内の地理情報を返されるカラムの値として抽出するために、`columns_from_path`パラメータを`"columns_from_path" = "country, city"`と指定することができます。詳細な手順については、Example 4を参照してください。

<!--

### schema_detect

v3.2以降、FILES()はデータファイルのスキーマを自動的に検出し、同じバッチのデータファイルを結合します。StarRocksはまず、バッチ内のランダムなデータファイルの一部のデータ行をサンプリングしてデータのスキーマを検出します。次に、StarRocksはバッチ内のすべてのデータファイルから列を結合します。

次のパラメータを使用してサンプリングルールを設定できます。

- `schema_auto_detect_sample_rows`: サンプリングされる各サンプルデータファイル内のデータ行数。範囲: [-1, 500]。このパラメータを`-1`に設定すると、すべてのデータ行がスキャンされます。
- `schema_auto_detect_sample_files`: 各バッチでサンプリングされるランダムなデータファイルの数。有効な値: `1`（デフォルト）および`-1`。このパラメータを`-1`に設定すると、すべてのデータファイルがスキャンされます。

サンプリング後、StarRocksは次のルールに従ってすべてのデータファイルから列を結合します。

- 列名またはインデックスが異なる列については、各列が個別の列として識別され、最終的にすべての個別の列の結合が返されます。
- 列名は同じですがデータ型が異なる場合、同じ列として識別されますが、より一般的なデータ型が使用されます。たとえば、ファイルAの列`col1`がINTであるが、ファイルBではDECIMALである場合、返される列にはDOUBLEが使用されます。STRING型はすべてのデータ型を結合するために使用できます。

すべての列を結合できない場合、StarRocksはエラーレポートを生成し、エラー情報とすべてのファイルスキーマを含めます。

> **注意**
>
> 単一のバッチ内のすべてのデータファイルは同じファイル形式である必要があります。

-->

### unload_data

v3.2以降、FILES()はリモートストレージにデータをアンロードするための書き込み可能なファイルを定義することもサポートしています。詳細な手順については、[INSERT INTO FILESを使用してデータをアンロード](../../../unloading/unload_using_insert_into_files.md)を参照してください。

- `compression`（必須）: データをアンロードする際に使用する圧縮方法。有効な値:
  - `uncompressed`: 圧縮アルゴリズムを使用しません。
  - `gzip`: gzip圧縮アルゴリズムを使用します。
  - `brotli`: Brotli圧縮アルゴリズムを使用します。
  - `zstd`: Zstd圧縮アルゴリズムを使用します。
  - `lz4`: LZ4圧縮アルゴリズムを使用します。
- `max_file_size`: データを複数のファイルにアンロードする場合の各データファイルの最大サイズ。デフォルト値: `1GB`。単位: B、KB、MB、GB、TB、PB。
- `partition_by`: データファイルを異なるストレージパスにパーティション分割するために使用する列のリスト。FILES()は指定された列のキー/値情報を抽出し、抽出されたキー/値ペアが特徴付けられたストレージパスの下にデータファイルを格納します。詳細な手順については、Example 5を参照してください。
- `single`: データを単一のファイルにアンロードするかどうか。有効な値:
  - `true`: データは単一のデータファイルに格納されます。
  - `false`: `max_file_size`に到達した場合、データは複数のファイルに格納されます。

> **注意**
>
> `max_file_size`と`single`の両方を指定することはできません。

## 使用上の注意

v3.2以降、FILES()は基本データ型に加えてARRAY、JSON、MAP、STRUCTのような複雑なデータ型もサポートしています。

## 例

Example 1: AWS S3バケット`inserttest`内のParquetファイル**parquet/par-dup.parquet**からデータをクエリします:

```Plain
MySQL > SELECT * FROM FILES(
     "path" = "s3://inserttest/parquet/par-dup.parquet",
     "format" = "parquet",
     "aws.s3.access_key" = "XXXXXXXXXX",
     "aws.s3.secret_key" = "YYYYYYYYYY",
     "aws.s3.region" = "us-west-2"
);
+------+---------------------------------------------------------+
| c1   | c2                                                      |
+------+---------------------------------------------------------+
|    1 | {"1": "key", "1": "1", "111": "1111", "111": "aaaa"}    |
|    2 | {"2": "key", "2": "NULL", "222": "2222", "222": "bbbb"} |
+------+---------------------------------------------------------+
2 rows in set (22.335 sec)
```

Example 2: AWS S3バケット`inserttest`内のParquetファイル**parquet/insert_wiki_edit_append.parquet**からデータ行をテーブル`insert_wiki_edit`に挿入します:

```Plain
MySQL > INSERT INTO insert_wiki_edit
    SELECT * FROM FILES(
        "path" = "s3://inserttest/parquet/insert_wiki_edit_append.parquet",
        "format" = "parquet",
        "aws.s3.access_key" = "XXXXXXXXXX",
        "aws.s3.secret_key" = "YYYYYYYYYY",
        "aws.s3.region" = "us-west-2"
);
Query OK, 2 rows affected (23.03 sec)
{'label':'insert_d8d4b2ee-ac5c-11ed-a2cf-4e1110a8f63b', 'status':'VISIBLE', 'txnId':'2440'}
```

Example 3: テーブル`ctas_wiki_edit`を作成し、AWS S3バケット`inserttest`内のParquetファイル**parquet/insert_wiki_edit_append.parquet**からデータ行をテーブルに挿入します:

```Plain
MySQL > CREATE TABLE ctas_wiki_edit AS
    SELECT * FROM FILES(
        "path" = "s3://inserttest/parquet/insert_wiki_edit_append.parquet",
        "format" = "parquet",
        "aws.s3.access_key" = "XXXXXXXXXX",
        "aws.s3.secret_key" = "YYYYYYYYYY",
        "aws.s3.region" = "us-west-2"
);
Query OK, 2 rows affected (22.09 sec)
{'label':'insert_1a217d70-2f52-11ee-9e4a-7a563fb695da', 'status':'VISIBLE', 'txnId':'3248'}
```

Example 4: Parquetファイル**/geo/country=US/city=LA/file1.parquet**（`id`と`user`の2つの列のみを含む）からデータをクエリし、パス内のキー/値情報を返されるカラムの値として抽出します。

```Plain
SELECT * FROM FILES(
    "path" = "hdfs://xxx.xx.xxx.xx:9000/geo/country=US/city=LA/file1.parquet",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "xxxxx",
    "password" = "xxxxx",
    "columns_from_path" = "country, city"
);
+------+---------+---------+------+
| id   | user    | country | city |
+------+---------+---------+------+
|    1 | richard | US      | LA   |
|    2 | amber   | US      | LA   |
+------+---------+---------+------+
2 rows in set (3.84 sec)
```

Example 5: `sales_records`のすべてのデータ行をHDFSクラスタのパス**/unload/partitioned/**に複数のParquetファイルとしてアンロードします。これらのファイルは、`sales_time`列の値で異なるサブパスに格納されます。

```SQL
INSERT INTO 
FILES(
    "path" = "hdfs://xxx.xx.xxx.xx:9000/unload/partitioned/",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "xxxxx",
    "password" = "xxxxx",
    "compression" = "lz4",
    "partition_by" = "sales_time"
)
SELECT * FROM sales_records;
```
