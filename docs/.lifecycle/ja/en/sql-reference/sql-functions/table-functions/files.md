---
displayed_sidebar: English
---

# FILES

## Description

リモートストレージ内のデータファイルを定義します。

v3.1.0以降、StarRocksはテーブル関数FILES()を使用してリモートストレージ内の読み取り専用ファイルを定義することをサポートしています。ファイルのパス関連プロパティを使用してリモートストレージにアクセスし、ファイル内のデータのテーブルスキーマを推測してデータ行を返します。[SELECT](../../sql-statements/data-manipulation/SELECT.md)を使用してデータ行を直接クエリするか、[INSERT](../../sql-statements/data-manipulation/INSERT.md)を使用してデータ行を既存のテーブルにロードするか、[CREATE TABLE AS SELECT](../../sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)を使用して新しいテーブルを作成しデータ行をロードすることができます。

v3.2.0以降、FILES()はリモートストレージ内のファイルへのデータ書き込みをサポートしています。[INSERT INTO FILES()を使用してStarRocksからリモートストレージへデータをアンロードすることができます](../../../unloading/unload_using_insert_into_files.md)。

現在、FILES()関数は以下のデータソースとファイルフォーマットをサポートしています：

- **データソース:**
  - HDFS
  - AWS S3
  - Google Cloud Storage
  - その他のS3互換ストレージシステム
  - Microsoft Azure Blob Storage
- **ファイルフォーマット:**
  - Parquet
  - ORC (現在データアンロードには対応していません)

## Syntax

- **データローディング**:

  ```SQL
  FILES( data_location , data_format [, StorageCredentialParams ] [, columns_from_path ] )
  ```

- **データアンローディング**:

  ```SQL
  FILES( data_location , data_format [, StorageCredentialParams ] , unload_data_param )
  ```

## Parameters

すべてのパラメータは`"key" = "value"`のペアで指定します。

### data_location

ファイルにアクセスするために使用されるURIです。パスまたはファイルを指定できます。

- HDFSにアクセスするには、このパラメータを以下のように指定します：

  ```SQL
  "path" = "hdfs://<hdfs_host>:<hdfs_port>/<hdfs_path>"
  -- 例: "path" = "hdfs://127.0.0.1:9000/path/file.parquet"
  ```

- AWS S3にアクセスするには：

  - S3プロトコルを使用する場合、このパラメータを以下のように指定します：

    ```SQL
    "path" = "s3://<s3_path>"
    -- 例: "path" = "s3://path/file.parquet"
    ```

  - S3Aプロトコルを使用する場合、このパラメータを以下のように指定します：

    ```SQL
    "path" = "s3a://<s3_path>"
    -- 例: "path" = "s3a://path/file.parquet"
    ```

- Google Cloud Storageにアクセスするには、このパラメータを以下のように指定します：

  ```SQL
  "path" = "s3a://<gcs_path>"
  -- 例: "path" = "s3a://path/file.parquet"
  ```

- Azure Blob Storageにアクセスするには：

  - ストレージアカウントがHTTP経由のアクセスを許可している場合、このパラメータを以下のように指定します：

    ```SQL
    "path" = "wasb://<container>@<storage_account>.blob.core.windows.net/<blob_path>"
    -- 例: "path" = "wasb://testcontainer@testaccount.blob.core.windows.net/path/file.parquet"
    ```
  
  - ストレージアカウントがHTTPS経由のアクセスを許可している場合、このパラメータを以下のように指定します：

    ```SQL
    "path" = "wasbs://<container>@<storage_account>.blob.core.windows.net/<blob_path>"
    -- 例: "path" = "wasbs://testcontainer@testaccount.blob.core.windows.net/path/file.parquet"
    ```

### data_format

データファイルのフォーマットです。有効な値は`parquet`と`orc`です。

### StorageCredentialParams

StarRocksがストレージシステムにアクセスするために使用する認証情報です。

StarRocksは現在、シンプル認証を使用したHDFSへのアクセス、IAMユーザーベースの認証を使用したAWS S3およびGCSへのアクセス、Shared Keyを使用したAzure Blob Storageへのアクセスをサポートしています。

- シンプル認証を使用してHDFSにアクセスするには：

  ```SQL
  "hadoop.security.authentication" = "simple",
  "username" = "xxxxxxxxxx",
  "password" = "yyyyyyyyyy"
  ```

  | **キー**                        | **必須** | **説明**                                              |
  | ------------------------------ | ------------ | ------------------------------------------------------------ |
  | hadoop.security.authentication | いいえ           | 認証方法です。有効な値は`simple`（デフォルト）。`simple`はシンプル認証を表し、認証が不要であることを意味します。 |
  | username                       | はい          | HDFSクラスタのNameNodeにアクセスするために使用するアカウントのユーザー名です。 |
  | password                       | はい          | HDFSクラスタのNameNodeにアクセスするために使用するアカウントのパスワードです。 |

- IAMユーザーベースの認証を使用してAWS S3にアクセスするには：

  ```SQL
  "aws.s3.access_key" = "xxxxxxxxxx",
  "aws.s3.secret_key" = "yyyyyyyyyy",
  "aws.s3.region" = "<s3_region>"
  ```

  | **キー**           | **必須** | **説明**                                              |
  | ----------------- | ------------ | ------------------------------------------------------------ |
  | aws.s3.access_key | はい          | Amazon S3バケットにアクセスするために使用できるアクセスキーIDです。 |
  | aws.s3.secret_key | はい          | Amazon S3バケットにアクセスするために使用できるシークレットアクセスキーです。 |
  | aws.s3.region     | はい          | AWS S3バケットが存在するリージョンです。例：`us-west-2`。 |

- IAMユーザーベースの認証を使用してGCSにアクセスするには：

  ```SQL
  "fs.s3a.access.key" = "xxxxxxxxxx",
  "fs.s3a.secret.key" = "yyyyyyyyyy",
  "fs.s3a.endpoint" = "<gcs_endpoint>"
  ```

  | **キー**           | **必須** | **説明**                                              |
  | ----------------- | ------------ | ------------------------------------------------------------ |
  | fs.s3a.access.key | はい          | GCSバケットにアクセスするために使用できるアクセスキーIDです。 |
  | fs.s3a.secret.key | はい          | GCSバケットにアクセスするために使用できるシークレットアクセスキーです。 |
  | fs.s3a.endpoint   | はい          | GCSバケットにアクセスするために使用できるエンドポイントです。例：`storage.googleapis.com`。 |

- Shared Keyを使用してAzure Blob Storageにアクセスするには：

  ```SQL
  "azure.blob.storage_account" = "<storage_account>",
  "azure.blob.shared_key" = "<shared_key>"
  ```

  | **キー**                    | **必須** | **説明**                                              |
  | -------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.blob.storage_account | はい          | Azure Blob Storageアカウントの名前です。                  |
  | azure.blob.shared_key      | はい          | Azure Blob Storageアカウントにアクセスするために使用できる共有キーです。 |

### columns_from_path

v3.2以降、StarRocksはファイルパスからキー/値ペアの値を列の値として抽出することができます。

```SQL
"columns_from_path" = "<column_name> [, ...]"
```

例えば、データファイル **file1** が `/geo/country=US/city=LA/` の形式のパスに格納されている場合、`columns_from_path` パラメータを `"columns_from_path" = "country, city"` と指定することで、ファイルパス内の地理情報を返される列の値として抽出することができます。詳細な手順については、例4を参照してください。

<!--

### schema_detect


v3.2以降、FILES()は自動スキーマ検出と同一バッチのデータファイルの統合をサポートしています。StarRocksはまず、バッチ内のランダムなデータファイルの特定のデータ行をサンプリングしてデータのスキーマを検出します。その後、バッチ内の全てのデータファイルから列を統合します。

サンプリングルールは以下のパラメータを使用して設定できます：

- `schema_auto_detect_sample_rows`：サンプリングされた各データファイルでスキャンするデータ行の数。範囲：[-1, 500]。このパラメータが`-1`に設定されている場合、全てのデータ行がスキャンされます。
- `schema_auto_detect_sample_files`：各バッチでサンプリングするランダムデータファイルの数。有効な値：`1`（デフォルト）と`-1`。このパラメータが`-1`に設定されている場合、全てのデータファイルがスキャンされます。

サンプリング後、StarRocksは以下のルールに従って全てのデータファイルから列を統合します：

- 列名やインデックスが異なる場合、各列は個別の列として識別され、最終的に全ての個別列の統合が返されます。
- 同じ列名だがデータ型が異なる場合、それらは同じ列として識別されますが、より一般的なデータ型が使用されます。例えば、ファイルAの`col1`列がINTで、ファイルBではDECIMALの場合、返される列にはDOUBLEが使用されます。STRING型は全てのデータ型を統合するために使用することができます。

StarRocksが全ての列を統合することに失敗した場合、エラー情報と全てのファイルスキーマを含むスキーマエラーレポートを生成します。

> **注意**
>
> 単一バッチ内の全てのデータファイルは同じファイル形式でなければなりません。

-->

### unload_data_param

v3.2以降、FILES()はデータアンロード用にリモートストレージ内の書き込み可能なファイルを定義することをサポートしています。詳細な手順については、[INSERT INTO FILESを使用したデータのアンロード](../../../unloading/unload_using_insert_into_files.md)を参照してください。

```sql
-- v3.2からサポートされています。
unload_data_param::=
    "compression" = "<compression_method>",
    "max_file_size" = "<file_size>",
    "partition_by" = "<column_name> [, ...]" 
    "single" = { "true" | "false" } 
```


| **キー**          | **必須** | **説明**                                              |
| ---------------- | ------------ | ------------------------------------------------------------ |
| compression      | はい          | データアンロード時に使用する圧縮方法。有効な値：<ul><li>`uncompressed`：圧縮アルゴリズムは使用されません。</li><li>`gzip`：gzip圧縮アルゴリズムを使用します。</li><li>`brotli`：Brotli圧縮アルゴリズムを使用します。</li><li>`zstd`：Zstd圧縮アルゴリズムを使用します。</li><li>`lz4`：LZ4圧縮アルゴリズムを使用します。</li></ul>                  |
| max_file_size    | いいえ           | 複数のファイルにデータをアンロードする際の各データファイルの最大サイズ。デフォルト値：`1GB`。単位：B、KB、MB、GB、TB、PB。 |
| partition_by     | いいえ           | データファイルを異なるストレージパスに分割するために使用される列のリスト。複数の列はコンマ（,）で区切られます。FILES()は指定された列のキー/値情報を抽出し、抽出されたキー/値ペアを特徴とするストレージパスにデータファイルを格納します。詳細な手順については、例5を参照してください。 |
| single           | いいえ           | データを単一のファイルにアンロードするかどうか。有効な値：<ul><li>`true`：データは単一のデータファイルに格納されます。</li><li>`false`（デフォルト）：`max_file_size`に達した場合、データは複数のファイルに格納されます。</li></ul>                  |

> **注意**
>
> `max_file_size`と`single`を同時に指定することはできません。

## 使用上の注意

v3.2以降、FILES()は基本的なデータ型に加えて、ARRAY、JSON、MAP、STRUCTなどの複雑なデータ型もサポートしています。

## 例

例1：AWS S3バケット`inserttest`内のParquetファイル**parquet/par-dup.parquet**からデータをクエリします。

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

例2：AWS S3バケット`inserttest`内のParquetファイル**parquet/insert_wiki_edit_append.parquet**からデータ行をテーブル`insert_wiki_edit`に挿入します。

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

例3：`ctas_wiki_edit`という名前のテーブルを作成し、AWS S3バケット`inserttest`内のParquetファイル**parquet/insert_wiki_edit_append.parquet**からデータ行をテーブルに挿入します。

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

例4：Parquetファイル**/geo/country=US/city=LA/file1.parquet**（`id`と`user`の2つの列のみ含む）からデータをクエリし、そのパス内のキー/値情報を列として抽出します。

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


例 5: `sales_records` のすべてのデータ行を、HDFS クラスターのパス **/unload/partitioned/** 下に複数の Parquet ファイルとしてアンロードします。これらのファイルは、`sales_time` 列の値によって区別される異なるサブパスに格納されます。

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

