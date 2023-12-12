```yaml
      + ファイル
      + ファイルとは？
    + 定義
  + リモートストレージ内のデータファイルを定義します。

v3.1.0から、StarRocksはFILES()テーブル関数を使用して、リモートストレージにある読み取り専用ファイルを定義できるようになりました。これにより、ファイルのパス関連のプロパティからリモートストレージにアクセスし、ファイル内のデータのテーブルスキーマを推論し、データ行を返すことができます。 [SELECT](../../sql-statements/data-manipulation/SELECT.md) を使用してデータ行を直接クエリできます。 [INSERT](../../sql-statements/data-manipulation/INSERT.md) を使用して既存のテーブルにデータ行をロードしたり、 [CREATE TABLE AS SELECT](../../sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md) を使用して新しいテーブルを作成し、データ行をロードすることもできます。

v3.2.0から、FILES()はリモートストレージにデータを書き込むことをサポートしています。[INSERT INTO FILES() を使用して、StarRocksからリモートストレージにデータをアンロード](../../../unloading/unload_using_insert_into_files.md) することができます。

現在、FILES() 関数は以下のデータソースとファイルフォーマットをサポートしています。

- **データソース:**
  - HDFS
  - AWS S3
  - Google Cloud Storage
  - その他のS3互換ストレージシステム
  - Microsoft Azure Blob Storage
- **ファイルフォーマット:**
  - Parquet
  - ORC（現在、データのアンロードにはサポートされていません）

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

すべてのパラメータは、`"key" = "value"` ペアで指定されています。

### data_location

ファイルにアクセスするために使用するURI。パスまたはファイルを指定できます。

- HDFSにアクセスする場合、このパラメータは次のように指定する必要があります：

  ```SQL
  "path" = "hdfs://<hdfs_host>:<hdfs_port>/<hdfs_path>"
  -- 例: "path" = "hdfs://127.0.0.1:9000/path/file.parquet"
  ```

- AWS S3にアクセスする場合：

  - S3プロトコルを使用する場合、このパラメータを次のように指定する必要があります：

    ```SQL
    "path" = "s3://<s3_path>"
    -- 例: "path" = "s3://path/file.parquet"
    ```

  - S3Aプロトコルを使用する場合、このパラメータを次のように指定する必要があります：

    ```SQL
    "path" = "s3a://<s3_path>"
    -- 例: "path" = "s3a://path/file.parquet"
    ```

- Google Cloud Storageにアクセスする場合、このパラメータを次のように指定する必要があります：

  ```SQL
  "path" = "s3a://<gcs_path>"
  -- 例: "path" = "s3a://path/file.parquet"
  ```

- Azure Blob Storageにアクセスする場合：

  - ストレージアカウントがHTTP経由でのアクセスを許可している場合、このパラメータを次のように指定する必要があります：

    ```SQL
    "path" = "wasb://<container>@<storage_account>.blob.core.windows.net/<blob_path>"
    -- 例: "path" = "wasb://testcontainer@testaccount.blob.core.windows.net/path/file.parquet"
    ```

  - ストレージアカウントがHTTPS経由でのアクセスを許可している場合、このパラメータを次のように指定する必要があります：

    ```SQL
    "path" = "wasbs://<container>@<storage_account>.blob.core.windows.net/<blob_path>"
    -- 例: "path" = "wasbs://testcontainer@testaccount.blob.core.windows.net/path/file.parquet"
    ```

### data_format

データファイルのフォーマット。有効な値: `parquet`、`orc`。

### StorageCredentialParams

StarRocksがストレージシステムにアクセスする際に使用する認証情報。

現在、StarRocksは、HDFSへのアクセスにシンプルな認証、AWS S3およびGCSへのIAMユーザーベースの認証、Azure Blob Storageへの共有キーをサポートしています。

- HDFSにアクセスするためにシンプルな認証を使用する場合：

  ```SQL
  "hadoop.security.authentication" = "simple",
  "username" = "xxxxxxxxxx",
  "password" = "yyyyyyyyyy"
  ```

  | **Key**                        | **Required** | **Description**                                              |
  | ------------------------------ | ------------ | ------------------------------------------------------------ |
  | hadoop.security.authentication | No           | 認証メソッド。有効な値: `simple` (デフォルト)。 `simple` は、シンプルな認証を表し、認証を行いません。 |
  | username                       | Yes          | HDFSクラスターのNameNodeにアクセスするために使用するアカウントのユーザー名。 |
  | password                       | Yes          | HDFSクラスターのNameNodeにアクセスするために使用するアカウントのパスワード。 |

- AWS S3にアクセスするためにIAMユーザーベースの認証を使用する場合：

  ```SQL
  "aws.s3.access_key" = "xxxxxxxxxx",
  "aws.s3.secret_key" = "yyyyyyyyyy",
  "aws.s3.region" = "<s3_region>"
  ```

  | **Key**           | **Required** | **Description**                                              |
  | ----------------- | ------------ | ------------------------------------------------------------ |
  | aws.s3.access_key | Yes          | Amazon S3バケットにアクセスするために使用できるアクセスキーID。 |
  | aws.s3.secret_key | Yes          | Amazon S3バケットにアクセスするために使用できるシークレットアクセスキー。 |
  | aws.s3.region     | Yes          | AWS S3バケットが存在するリージョン。例: `us-west-2`。 |

- GCSにアクセスするためにIAMユーザーベースの認証を使用する場合：

  ```SQL
  "fs.s3a.access.key" = "xxxxxxxxxx",
  "fs.s3a.secret.key" = "yyyyyyyyyy",
  "fs.s3a.endpoint" = "<gcs_endpoint>"
  ```

  | **Key**           | **Required** | **Description**                                              |
  | ----------------- | ------------ | ------------------------------------------------------------ |
  | fs.s3a.access.key | Yes          | GCSバケットにアクセスするために使用できるアクセスキーID。 |
  | fs.s3a.secret.key | Yes          | GCSバケットにアクセスするために使用できるシークレットアクセスキー。 |
  | fs.s3a.endpoint   | Yes          | GCSバケットにアクセスするために使用できるエンドポイント。例: `storage.googleapis.com`。 |

- Azure Blob Storageにアクセスするために共有キーを使用する場合：

  ```SQL
  "azure.blob.storage_account" = "<storage_account>",
  "azure.blob.shared_key" = "<shared_key>"
  ```

  | **Key**                    | **Required** | **Description**                                              |
  | -------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.blob.storage_account | Yes          | Azure Blob Storageアカウントの名前。                          |
  | azure.blob.shared_key      | Yes          | Azure Blob Storageアカウントにアクセスするために使用できる共有キー。 |

### columns_from_path

StarRocks v3.2以降、StarRocksはファイルパスからキー/値ペアの値を列の値として抽出できます。

```SQL
"columns_from_path" = "<column_name> [, ...]"
```

データファイル **file1** が、`/geo/country=US/city=LA/` の形式のパスに保存されているとします。ファイルパスの地理的情報を返される列の値として抽出するには、`"columns_from_path" = "country, city"` というように指定することができます。詳細な手順については、Example 4 を参照してください。

<!--

### schema_detect

StarRocks v3.2以降、FILES()はデータファイルのスキーマの自動検出と同じバッチ内のデータファイルの統合をサポートしています。StarRocksはまず、バッチ内のランダムなデータファイルの一部のデータ行をサンプリングしてデータのスキーマを検出します。その後、StarRocksはバッチ内のすべてのデータファイルからの列を統合します。

次のパラメータを使用してサンプリングルールを構成できます：

- `schema_auto_detect_sample_rows`:  サンプリングされたデータファイルごとにスキャンするデータ行の数。範囲: [-1, 500]。このパラメータが `-1` に設定されている場合、すべてのデータ行がスキャンされます。
- `schema_auto_detect_sample_files`:  バッチごとにサンプリングされたランダムなデータファイルの数。有効な値: `1` (デフォルト)、`-1`。このパラメータが `-1` に設定されている場合、すべてのデータファイルがスキャンされます。

サンプリング後、StarRocksは次のルールに従ってバッチ内のすべてのデータファイルからの列を統合します：

- 異なる列名またはインデックスを持つ列の場合、各列は個々の列として識別され、最終的にすべての個々の列が結合されます。
- データ型が同じ列名でも異なる場合、同じ列として識別されますが、より一般的なデータ型が使用されます。例えば、ファイルAの列 `col1` がINTである場合、ファイルBではDECIMALですが、返される列にはDOUBLEが使用されます。STRING型はすべてのデータ型を統計するために使用できます。

StarRocksがすべての列を統合できない場合、エラーレポートが生成され、エラー情報とすべてのファイルスキーマが含まれます。

> **注意**
>
> 1つのバッチ内のすべてのデータファイルは同じファイルフォーマットである必要があります。

-->
### unload_data_param
```
v3.2以降、FILES()はデータのアンロードのためにリモートストレージ内で書き込み可能なファイルを定義することをサポートしています。詳しい手順については、[FILESを使用したデータのアンロード](../../../unloading/unload_using_insert_into_files.md)を参照してください。

```sql
-- v3.2からサポートされています。
unload_data_param::=
    "compression" = "<compression_method>",
    "max_file_size" = "<file_size>",
    "partition_by" = "<column_name> [, ...]" 
    "single" = { "true" | "false" } 
```


| **Key**          | **Required** | **Description**                                              |
| ---------------- | ------------ | ------------------------------------------------------------ |
| compression      | Yes          | データのアンロード時に使用する圧縮メソッド。有効な値:<ul><li>`uncompressed`: 圧縮アルゴリズムは使用されません。</li><li>`gzip`: gzip圧縮アルゴリズムを使用します。</li><li>`brotli`: Brotli圧縮アルゴリズムを使用します。</li><li>`zstd`: Zstd圧縮アルゴリズムを使用します。</li><li>`lz4`: LZ4圧縮アルゴリズムを使用します。</li></ul>                  |
| max_file_size    | No           | データが複数のファイルにアンロードされる場合の各データファイルの最大サイズ。デフォルト値: `1GB`。単位: B、KB、MB、GB、TB、PB。 |
| partition_by     | No           | データファイルを異なるストレージパスにパーティション化するために使用される列のリスト。複数の列はコンマ(,)で区切られます。FILES()は指定された列のキー/値情報を抽出し、抽出されたキー/値ペアで特徴付けられたストレージパスの下にデータファイルを格納します。詳しい手順については、Example 5を参照してください。 |
| single           | No           | データを単一のファイルにアンロードするかどうか。有効な値:<ul><li>`true`: データは単一のデータファイルに格納されます。</li><li>`false` (デフォルト): `max_file_size`が到達した場合、データは複数のファイルに格納されます。</li></ul>                  |

> **CAUTION**
>
> `max_file_size`と`single`の両方を指定することはできません。

## 使用上の注意

v3.2以降、FILES()は基本的なデータ型に加えて、ARRAY、JSON、MAP、STRUCTを含む複雑なデータ型をさらにサポートしています。

## 例

Example 1: AWS S3バケット`inserttest`内のParquetファイル **parquet/par-dup.parquet** からデータをクエリします。

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

Example 2: AWS S3バケット`inserttest`内のParquetファイル **parquet/insert_wiki_edit_append.parquet** からデータ行をテーブル`insert_wiki_edit`に挿入します。

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

Example 3: AWS S3バケット`inserttest`内のParquetファイル **parquet/insert_wiki_edit_append.parquet** からデータ行をテーブル`ctas_wiki_edit`に挿入し、テーブル`ctas_wiki_edit`を作成します。

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

Example 4: パス内にある`geo/country=US/city=LA/file1.parquet`からデータをクエリし、そのパスからキー/値情報を抽出して返される列として使用します。

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

Example 5: `sales_records`のすべてのデータ行をHDFSクラスタ内のパス **/unload/partitioned/** に複数のParquetファイルとしてアンロードします。これらのファイルは、列`sales_time`の値によって異なるサブパスに保存されます。

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