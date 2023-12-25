---
displayed_sidebar: Chinese
---

# FILES

## 機能

リモートストレージ内のデータファイルを定義します。

バージョン v3.1.0 から、StarRocks はテーブル関数 FILES() を使用してリモートストレージ内で読み取り専用のファイルを定義することをサポートしています。この関数は指定されたデータパスなどのパラメータをもとにデータを読み取り、データファイルの形式や列情報などから Table Schema を自動的に推測し、最終的にデータファイル内のデータを行として返します。[SELECT](../../sql-statements/data-manipulation/SELECT.md) を使用して直接データをクエリしたり、[INSERT](../../sql-statements/data-manipulation/INSERT.md) でデータをインポートしたり、[CREATE TABLE AS SELECT](../../sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md) でテーブルを作成してデータをインポートすることができます。

バージョン v3.2.0 からは、FILES() を使用してリモートストレージにデータを書き込むことができます。[INSERT INTO FILES() を使用して StarRocks からリモートストレージにデータをエクスポートする](../../../unloading/unload_using_insert_into_files.md) 方法があります。

現在、FILES() 関数は以下のデータソースとファイル形式をサポートしています：

- **データソース：**
  - HDFS
  - AWS S3
  - Google Cloud Storage
  - Microsoft Azure Blob Storage
- **ファイル形式：**
  - Parquet
  - ORC（データエクスポートは現在サポートしていません）

## 文法

- **インポート**:

  ```SQL
  FILES( data_location , data_format [, StorageCredentialParams ] [, columns_from_path ] )
  ```

- **エクスポート**:

  ```SQL
  FILES( data_location , data_format [, StorageCredentialParams ] , unload_data_param )
  ```


## パラメータ説明

すべてのパラメータは `"key" = "value"` 形式のペアです。

### data_location

ファイルにアクセスするための URI。パスまたはファイル名を指定できます。

- HDFS にアクセスするには、以下のように指定します：

  ```SQL
  "path" = "hdfs://<hdfs_host>:<hdfs_port>/<hdfs_path>"
  -- 例： "path" = "hdfs://127.0.0.1:9000/path/file.parquet"
  ```

- AWS S3 にアクセスするには：

  - S3 プロトコルを使用する場合、以下のように指定します：

    ```SQL
    "path" = "s3://<s3_path>"
    -- 例： "path" = "s3://mybucket/file.parquet"
    ```

  - S3A プロトコルを使用する場合、以下のように指定します：

    ```SQL
    "path" = "s3a://<s3_path>"
    -- 例： "path" = "s3a://mybucket/file.parquet"
    ```

- Google Cloud Storage にアクセスするには、以下のように指定します：

  ```SQL
  "path" = "gs://<gcs_path>"
  -- 例： "path" = "gs://mybucket/file.parquet"
  ```

- Azure Blob Storage にアクセスするには：

  - ストレージアカウントが HTTP 経由でのアクセスを許可している場合、以下のように指定します：

    ```SQL
    "path" = "wasb://<container>@<storage_account>.blob.core.windows.net/<blob_path>"
    -- 例： "path" = "wasb://testcontainer@testaccount.blob.core.windows.net/path/file.parquet"
    ```

  - ストレージアカウントが HTTPS 経由でのアクセスを許可している場合、以下のように指定します：

    ```SQL
    "path" = "wasbs://<container>@<storage_account>.blob.core.windows.net/<blob_path>"
    -- 例： "path" = "wasbs://testcontainer@testaccount.blob.core.windows.net/path/file.parquet"
    ```

### data_format

データファイルの形式。有効な値は `parquet` と `orc` です。

### StorageCredentialParams

StarRocks がストレージシステムにアクセスするための認証設定です。

StarRocks は現在、HDFS クラスターへのシンプル認証、AWS S3 および Google Cloud Storage への IAM User 認証、Azure Blob Storage への Shared Key 認証のみをサポートしています。

- HDFS クラスターへのシンプル認証を使用する場合：

  ```SQL
  "hadoop.security.authentication" = "simple",
  "username" = "xxxxxxxxxx",
  "password" = "yyyyyyyyyy"
  ```

  | **パラメータ**                       | **必須** | **説明**                                                     |
  | ------------------------------ | -------- | ------------------------------------------------------------ |
  | hadoop.security.authentication | いいえ       | アクセスする HDFS クラスターの認証方式を指定します。有効な値：`simple`（デフォルト）。`simple` はシンプル認証、つまり認証なしを意味します。 |
  | username                       | はい       | HDFS クラスター内の NameNode ノードにアクセスするためのユーザー名です。                 |
  | password                       | はい       | HDFS クラスター内の NameNode ノードにアクセスするためのパスワードです。                   |

- AWS S3 への IAM User 認証を使用する場合：

  ```SQL
  "aws.s3.access_key" = "xxxxxxxxxx",
  "aws.s3.secret_key" = "yyyyyyyyyy",
  "aws.s3.region" = "<s3_region>"
  ```

  | **パラメータ**          | **必須** | **説明**                                                 |
  | ----------------- | -------- | -------------------------------------------------------- |
  | aws.s3.access_key | はい       | AWS S3 ストレージスペースにアクセスするための Access Key を指定します。              |
  | aws.s3.secret_key | はい       | AWS S3 ストレージスペースにアクセスするための Secret Key を指定します。              |
  | aws.s3.region     | はい       | アクセスする AWS S3 ストレージスペースのリージョンを指定します。例：`us-west-2`。 |

- GCS への IAM User 認証を使用する場合：

  ```SQL
  "fs.gs.access.key" = "xxxxxxxxxx",
  "fs.gs.secret.key" = "yyyyyyyyyy",
  "fs.gs.endpoint" = "<gcs_endpoint>"
  ```

  | **パラメータ**          | **必須** | **説明**                                                 |
  | ----------------- | -------- | -------------------------------------------------------- |
  | fs.gs.access.key | はい       | GCS ストレージスペースにアクセスするための Access Key を指定します。              |
  | fs.gs.secret.key | はい       | GCS ストレージスペースにアクセスするための Secret Key を指定します。              |
  | fs.gs.endpoint   | はい       | アクセスする GCS ストレージスペースのエンドポイントを指定します。例：`storage.googleapis.com`。 |

- Azure Blob Storage への Shared Key 認証を使用する場合：

  ```SQL
  "azure.blob.storage_account" = "<storage_account>",
  "azure.blob.shared_key" = "<shared_key>"
  ```

  | **パラメータ**                   | **必須** | **説明**                                                 |
  | -------------------------- | -------- | ------------------------------------------------------ |
  | azure.blob.storage_account | はい       | Azure Blob Storage アカウント名を指定します。                  |
  | azure.blob.shared_key      | はい       | Azure Blob Storage スペースにアクセスするための Shared Key を指定します。     |

### columns_from_path

バージョン v3.2 以降、StarRocks はファイルパスから Key/Value ペアの Value を列の値として抽出することをサポートしています。

```SQL
"columns_from_path" = "<column_name> [, ...]"
```

例えば、データファイル **file1** がパス `/geo/country=US/city=LA/` に保存されている場合、`columns_from_path` パラメータを `"columns_from_path" = "country, city"` と指定することで、ファイルパスから地理情報を列の値として抽出することができます。詳細な使用方法は以下の例四を参照してください。

<!--

### schema_detect


自 v3.2 版本起、`FILES()` はバッチデータファイルに対して自動的なスキーマ検出とユニオン操作をサポートしています。StarRocks はまず、同一バッチ内のランダムなデータファイルをスキャンしてサンプリングし、データのスキーマを検出します。その後、StarRocks は同一バッチ内の全てのデータファイルの列に対してユニオン操作を行います。

以下のパラメータを使用してサンプリングルールを設定できます：

- `schema_auto_detect_sample_rows`：各サンプリングデータファイルの行数をスキャンします。範囲：[-1, 500]。このパラメータを `-1` に設定すると、全ての行をスキャンします。
- `schema_auto_detect_sample_files`：各バッチでサンプリングされるランダムなデータファイルの数。有効な値：`1`（デフォルト）と `-1`。このパラメータを `-1` に設定すると、全てのデータファイルをスキャンします。

サンプリング後、StarRocks は以下のルールに基づいて全てのデータファイルの列をユニオンします：

- 列名またはインデックスが異なる列については、StarRocks はそれぞれの列を個別の列として認識し、最終的に全ての個別列を返します。
- 列名は同じだがデータ型が異なる列については、StarRocks はこれらの列を同一の列として認識し、共通のデータ型を選択します。例えば、ファイル A の `col1` 列が INT 型で、ファイル B の `col1` 列が DECIMAL 型の場合、返される列では DOUBLE データ型を使用します。STRING 型は全てのデータ型を統一するために使用できます。

StarRocks が全ての列を統一できない場合、エラーメッセージと全てのファイルスキーマを含むエラーレポートが生成されます。

> **注意**
>
> 同一バッチ内の全てのデータファイルは同じファイルフォーマットでなければなりません。

-->

### unload_data_param

v3.2 版から、`FILES()` はリモートストレージに書き込み可能なファイルを定義してデータエクスポートをサポートしています。詳細は[INSERT INTO FILES を使用したデータエクスポート](../../../unloading/unload_using_insert_into_files.md)を参照してください。

```sql
-- v3.2 版からサポートされています。
unload_data_param::=
    "compression" = "<compression_method>",
    "max_file_size" = "<file_size>",
    "partition_by" = "<column_name> [, ...]" 
    "single" = { "true" | "false" } 
```

| **パラメータ**   | **必須** | **説明**                                                          |
| ---------------- | ------------ | ------------------------------------------------------------ |
| compression      | はい          | データエクスポート時に使用する圧縮方法。有効な値：<ul><li>`uncompressed`：圧縮を使用しません。</li><li>`gzip`：gzip 圧縮アルゴリズムを使用します。</li><li>`brotli`：Brotli 圧縮アルゴリズムを使用します。</li><li>`zstd`：Zstd 圧縮アルゴリズムを使用します。</li><li>`lz4`：LZ4 圧縮アルゴリズムを使用します。</li></ul>                  |
| max_file_size    | いいえ           | データが複数のファイルにエクスポートされる場合の各データファイルの最大サイズ。デフォルト値：`1GB`。単位：B、KB、MB、GB、TB、PB。 |
| partition_by     | いいえ           | データファイルを異なるストレージパスに分割するための列。複数の列を指定できます。`FILES()` は指定された列のキー/値情報を抽出し、対応するキー/値に基づいてサブパスにデータファイルを格納します。使用方法の詳細は以下の例五を参照してください。 |
| single           | いいえ           | データを単一のファイルにエクスポートするかどうか。有効な値：<ul><li>`true`：データを単一のファイルに格納します。</li><li>`false`（デフォルト）：`max_file_size` に達した場合、データを複数のファイルに格納します。</li></ul>                  |

> **注意**
>
> `max_file_size` と `single` を同時に指定することはできません。

## 注意事項

v3.2 版から、基本データ型に加えて、`FILES()` は ARRAY、JSON、MAP、STRUCT といった複雑なデータ型もサポートしています。

## 例

例一：AWS S3 バケット `inserttest` 内の Parquet ファイル **parquet/par-dup.parquet** のデータをクエリします

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

例二：AWS S3 バケット `inserttest` 内の Parquet ファイル **parquet/insert_wiki_edit_append.parquet** のデータを `insert_wiki_edit` テーブルに挿入します：

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

例三：AWS S3 バケット `inserttest` 内の Parquet ファイル **parquet/insert_wiki_edit_append.parquet** のデータを基に `ctas_wiki_edit` テーブルを作成します：

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

例四：HDFS クラスタ内の Parquet ファイル **/geo/country=US/city=LA/file1.parquet** のデータをクエリし（`id` と `user` の2列のみ含む）、そのパス内のキー/値情報を返される列として抽出します。

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

例五：`sales_records` の全てのデータ行を複数の Parquet ファイルとしてエクスポートし、HDFS クラスタのパス **/unload/partitioned/** に保存します。これらのファイルは、`sales_time` 列の値に基づいて異なるサブパスに格納されます。

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