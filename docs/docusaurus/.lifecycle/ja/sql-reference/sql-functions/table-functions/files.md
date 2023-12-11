---
displayed_sidebar: "Japanese"
---

# FILES（ファイル）

## 説明

リモートストレージ内のデータファイルを定義します。

v3.1.0以降、StarRocksはテーブル関数FILES()を使用してリモートストレージ内の読み取り専用ファイルを定義することができます。これにより、ファイルのパス関連のプロパティを使用してリモートストレージにアクセスし、ファイル内のデータのテーブルスキーマを推測し、データ行を返すことができます。[SELECT](../../sql-statements/data-manipulation/SELECT.md)を使用してデータ行を直接クエリしたり、[INSERT](../../sql-statements/data-manipulation/INSERT.md)を使用してデータ行を既存のテーブルにロードしたり、[CREATE TABLE AS SELECT](../../sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)を使用して新しいテーブルを作成し、データ行をロードすることができます。

v3.2.0以降、FILES()はリモートストレージ内のファイルにデータを書き込むことをサポートしています。[INSERT INTO FILES()を使用してStarRocksからデータをアンロード](../../../unloading/unload_using_insert_into_files.md)することができます。

現在、FILES()関数は以下のデータソースとファイルフォーマットをサポートしています。

- **データソース:**
  - HDFS
  - AWS S3
  - Google Cloud Storage
  - その他のS3互換ストレージシステム
  - Microsoft Azure Blob Storage
- **ファイルフォーマット:**
  - Parquet
  - ORC（データのアンロードには現在非対応）

## 構文

- **データのロード**:

  ```SQL
  FILES( data_location , data_format [, StorageCredentialParams ] [, columns_from_path ] )
  ```

- **データのアンロード**:

  ```SQL
  FILES( data_location , data_format [, StorageCredentialParams ] , unload_data_param )
  ```

## パラメータ

すべてのパラメータは`"key" = "value"`のペアで表されます。

### data_location

ファイルにアクセスするために使用されるURI。パスまたはファイルを指定できます。

- HDFSにアクセスする場合、パラメータを次のように指定する必要があります：

  ```SQL
  "path" = "hdfs://<hdfs_host>:<hdfs_port>/<hdfs_path>"
  -- 例: "path" = "hdfs://127.0.0.1:9000/path/file.parquet"
  ```

- AWS S3にアクセスする場合：

  - S3プロトコルを使用する場合、パラメータを次のように指定する必要があります：

    ```SQL
    "path" = "s3://<s3_path>"
    -- 例: "path" = "s3://path/file.parquet"
    ```

  - S3Aプロトコルを使用する場合、パラメータを次のように指定する必要があります：

    ```SQL
    "path" = "s3a://<s3_path>"
    -- 例: "path" = "s3a://path/file.parquet"
    ```

- Google Cloud Storageにアクセスする場合、パラメータを次のように指定する必要があります：

  ```SQL
  "path" = "s3a://<gcs_path>"
  -- 例: "path" = "s3a://path/file.parquet"
  ```

- Azure Blob Storageにアクセスする場合：

  - ストレージアカウントがHTTPアクセスを許可する場合、パラメータを次のように指定する必要があります：

    ```SQL
    "path" = "wasb://<container>@<storage_account>.blob.core.windows.net/<blob_path>"
    -- 例: "path" = "wasb://testcontainer@testaccount.blob.core.windows.net/path/file.parquet"
    ```
  
  - ストレージアカウントがHTTPSアクセスを許可する場合、パラメータを次のように指定する必要があります：

    ```SQL
    "path" = "wasbs://<container>@<storage_account>.blob.core.windows.net/<blob_path>"
    -- 例: "path" = "wasbs://testcontainer@testaccount.blob.core.windows.net/path/file.parquet"
    ```

### data_format

データファイルのフォーマット。有効な値: `parquet`および`orc`。

### StorageCredentialParams

StarRocksがストレージシステムにアクセスするために使用する認証情報です。

現在、StarRocksは単純な認証を使用してHDFSにアクセスし、IAMユーザーベースの認証を使用してAWS S3およびGCSにアクセスし、Shared Keyを使用してAzure Blob Storageにアクセスすることをサポートしています。

- HDFSにアクセスする際の単純な認証の使用例：

  ```SQL
  "hadoop.security.authentication" = "simple",
  "username" = "xxxxxxxxxx",
  "password" = "yyyyyyyyyy"
  ```

  | **Key**                        | **Required** | **Description**                                              |
  | ------------------------------ | ------------ | ------------------------------------------------------------ |
  | hadoop.security.authentication | いいえ       | 認証メソッド。有効な値: `simple`（デフォルト）。`simple`は単純な認証を表し、認証を行わないことを意味します。 |
  | username                       | はい         | HDFSクラスターのNameNodeにアクセスするために使用するアカウントのユーザー名。 |
  | password                       | はい         | HDFSクラスターのNameNodeにアクセスするために使用するアカウントのパスワード。 |

- AWS S3にアクセスする際のIAMユーザーベースの認証の使用例：

  ```SQL
  "aws.s3.access_key" = "xxxxxxxxxx",
  "aws.s3.secret_key" = "yyyyyyyyyy",
  "aws.s3.region" = "<s3_region>"
  ```

  | **Key**           | **Required** | **Description**                                              |
  | ----------------- | ------------ | ------------------------------------------------------------ |
  | aws.s3.access_key | はい         | Amazon S3バケットにアクセスするために使用できるアクセスキーID。 |
  | aws.s3.secret_key | はい         | Amazon S3バケットにアクセスするために使用できるシークレットアクセスキー。 |
  | aws.s3.region     | はい         | AWS S3バケットが存在するリージョン。例: `us-west-2`。 |

- GCSにアクセスする際のIAMユーザーベースの認証の使用例：

  ```SQL
  "fs.s3a.access.key" = "xxxxxxxxxx",
  "fs.s3a.secret.key" = "yyyyyyyyyy",
  "fs.s3a.endpoint" = "<gcs_endpoint>"
  ```

  | **Key**           | **Required** | **Description**                                              |
  | ----------------- | ------------ | ------------------------------------------------------------ |
  | fs.s3a.access.key | はい         | GCSバケットにアクセスするために使用できるアクセスキーID。 |
  | fs.s3a.secret.key | はい         | GCSバケットにアクセスするために使用できるシークレットアクセスキー。|
  | fs.s3a.endpoint   | はい         | GCSバケットにアクセスするために使用できるエンドポイント。例: `storage.googleapis.com`。 |

- Azure Blob StorageにアクセスするためのShared Keyの使用例：

  ```SQL
  "azure.blob.storage_account" = "<storage_account>",
  "azure.blob.shared_key" = "<shared_key>"
  ```

  | **Key**                    | **Required** | **Description**                                              |
  | -------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.blob.storage_account | はい         | Azure Blob Storageアカウントの名前。                          |
  | azure.blob.shared_key      | はい         | Azure Blob Storageアカウントにアクセスするために使用できる共有キー。 |

### columns_from_path

v3.2以降、StarRocksはファイルパスからキー/値のペアの値を列の値として抽出することができます。

```SQL
"columns_from_path" = "<column_name> [, ...]"
```

データファイル**file1**が`/geo/country=US/city=LA/`という形式のパスに格納されている場合、「"columns_from_path" = "country, city"」のように`columns_from_path`パラメータを指定して、ファイルパス内の地理情報を返される列の値として抽出することができます。詳細な手順については、Example 4を参照してください。

<!--

### schema_detect

v3.2以降、FILES()はデータファイルのスキーマの自動検出と同じバッチのデータファイルの合併をサポートしています。StarRocksはまず、バッチ内のランダムなデータファイルの一定のデータ行をサンプリングしてデータのスキーマを検出します。その後、StarRocksはバッチ内のすべてのデータファイルからの列を合併します。

以下のパラメータを使用してサンプリングルールを構成することができます。

- `schema_auto_detect_sample_rows`: サンプリングされる各データファイル内のデータ行の数。範囲: [-1, 500]。このパラメータが`-1`に設定されている場合、すべてのデータ行がスキャンされます。
- `schema_auto_detect_sample_files`: 各バッチでサンプリングされるランダムなデータファイルの数。有効な値: `1` (デフォルト) および`-1`。このパラメータが`-1`に設定されている場合、すべてのデータファイルがスキャンされます。

サンプリング後、StarRocksはこれらのルールに従ってバッチ内のすべてのデータファイルからの列を合併します:

- 列名またはインデックスが異なる列については、それぞれ個別の列として識別され、最終的にすべての個別の列の合併が返されます。
- 列名が同じでデータ型が異なる場合、同じ列として識別されますが、より一般的なデータ型が使用されます。例えば、ファイルAの`col1`がINTであるが、ファイルBではDECIMALである場合、返される列ではDOUBLEが使用されます。STRING型はすべてのデータ型を合併するために使用できます。

すべての列を合併できない場合、StarRocksはエラーレポートを生成し、エラー情報とすべてのファイルスキーマを含めます。

> **注意**
>
> 1つのバッチ内のすべてのデータファイルは同じファイルフォーマットである必要があります。

-->

### unload_data_param
v3.2から、FILES()ではデータのアンロードのためにリモートストレージで書き込み可能なファイルを定義することがサポートされています。詳細な手順については、[FILESを使用したデータのアンロード](../../../unloading/unload_using_insert_into_files.md)を参照してください。

```sql
-- v3.2以降でサポートされています。
unload_data_param::=
    "compression" = "<compression_method>",
    "max_file_size" = "<file_size>",
    "partition_by" = "<column_name> [, ...]" 
    "single" = { "true" | "false" } 
```


| **Key**          | **Required** | **Description**                                              |
| ---------------- | ------------ | ------------------------------------------------------------ |
| compression      | Yes          | データのアンロード時に使用する圧縮方法。有効な値:<ul><li>`uncompressed`: 圧縮アルゴリズムは使用されません。</li><li>`gzip`: gzip圧縮アルゴリズムを使用します。</li><li>`brotli`: Brotli圧縮アルゴリズムを使用します。</li><li>`zstd`: Zstd圧縮アルゴリズムを使用します。</li><li>`lz4`: LZ4圧縮アルゴリズムを使用します。</li></ul>                  |
| max_file_size    | No           | データを複数のファイルにアンロードする場合の各データファイルの最大サイズ。デフォルト値: `1GB`。単位: B、KB、MB、GB、TB、PB。 |
| partition_by     | No           | データファイルを異なるストレージパスにパーティション分けするために使用される列のリスト。複数の列はカンマ(,)で区切られます。FILES()は指定した列のキー/値情報を抽出し、抽出されたキー/値のペアを備えたストレージパスの下にデータファイルを保存します。詳細な手順については、Example 5を参照してください。 |
| single           | No           | データを単一のファイルにアンロードするかどうか。有効な値:<ul><li>`true`: データは単一のデータファイルに保存されます。</li><li>`false` (デフォルト): `max_file_size`が到達した場合、データは複数のファイルに保存されます。</li></ul>                  |

> **注意**
>
> `max_file_size`と`single`の両方を指定することはできません。

## 使用上の注意

v3.2以降、FILES()は基本データ型に加えて、ARRAY、JSON、MAP、STRUCTなどの複雑なデータ型もサポートしています。

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
2行の結果(22.335秒)
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
クエリは正常に実行され、2行が変更されました(23.03秒)
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
クエリは正常に実行され、2行が変更されました(22.09秒)
{'label':'insert_1a217d70-2f52-11ee-9e4a-7a563fb695da', 'status':'VISIBLE', 'txnId':'3248'}
```

Example 4: **/geo/country=US/city=LA/file1.parquet**のParquetファイルからデータをクエリし（ここには`id`と`user`の2つの列しか含まれていません）、そのパスからのキー/値情報を返される列として抽出します。

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
2行の結果(3.84秒)
```

Example 5: `sales_records`から全データ行をHDFSクラスタ内の**/unload/partitioned/**パスに複数のParquetファイルとしてアンロードします。これらのファイルは、`sales_time`の値で区別された異なるサブパスに格納されます。

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