---
displayed_sidebar: English
---

# EXPORT

## 説明

テーブルのデータを指定された場所にエクスポートします。

これは非同期操作です。エクスポートタスクを送信した後に結果が返されます。[SHOW EXPORT](../../../sql-reference/sql-statements/data-manipulation/SHOW_EXPORT.md) を使用して、エクスポートタスクの進行状況を確認できます。

> **注意**
>
> StarRocks テーブルからデータをエクスポートするには、その StarRocks テーブルに対する EXPORT 権限が必要です。EXPORT 権限がない場合は、[GRANT](../account-management/GRANT.md) の指示に従って、StarRocks クラスタに接続するユーザーに EXPORT 権限を付与してください。

## 構文

```SQL
EXPORT TABLE <table_name>
[PARTITION (<partition_name>[, ...])]
[(<column_name>[, ...])]
TO <export_path>
[opt_properties]
WITH BROKER
[broker_properties]
```

## パラメーター

- `table_name`

  テーブル名。StarRocks は `engine` が `olap` または `mysql` のテーブルデータのエクスポートをサポートしています。

- `partition_name`

  エクスポートしたいパーティション。デフォルトでは、このパラメータを指定しない場合、StarRocks はテーブルの全パーティションからデータをエクスポートします。

- `column_name`

  エクスポートしたいカラム。このパラメータを使用して指定するカラムの順序は、テーブルのスキーマと異なる場合があります。デフォルトでは、このパラメータを指定しない場合、StarRocks はテーブルの全カラムからデータをエクスポートします。

- `export_path`

  テーブルデータをエクスポートしたい場所。場所にパスが含まれている場合、パスがスラッシュ (/) で終わることを確認してください。そうでない場合、パスの最後のスラッシュ (/) 以降の部分がエクスポートされたファイルの名前のプレフィックスとして使用されます。デフォルトでは、ファイル名のプレフィックスが指定されていない場合は `data_` が使用されます。

- `opt_properties`

  エクスポートタスクに設定できるオプションのプロパティ。

  構文：

  ```SQL
  [PROPERTIES ("<key>"="<value>", ...)]
  ```

  | **プロパティ**     | **説明**                                              |
  | ---------------- | ------------------------------------------------------------ |
  | column_separator | エクスポートファイルで使用するカラムセパレータ。デフォルト値: `\t`。 |
  | line_delimiter   | エクスポートファイルで使用する行区切り文字。デフォルト値: `\n`。 |
  | load_mem_limit   | 各 BE で許可されるエクスポートタスクの最大メモリ。単位: バイト。デフォルトの最大メモリは 2 GB。 |
  | timeout          | エクスポートタスクがタイムアウトするまでの時間。単位: 秒。デフォルト値: `86400`（1日）。 |
  | include_query_id | エクスポートされるファイル名に `query_id` を含めるかどうかを指定します。有効な値: `true` または `false`。`true` はファイル名に `query_id` を含めることを指定し、`false` は含めないことを指定します。 |

- `WITH BROKER`

  v2.4 以前では、`WITH BROKER "<broker_name>"` を入力して使用するブローカーを指定します。v2.5 以降では、ブローカーを指定する必要はありませんが、`WITH BROKER` キーワードは必要です。詳細は [EXPORT を使用したデータのエクスポート > 背景情報](../../../unloading/Export.md#background-information) を参照してください。

- `broker_properties`

  ソースデータの認証に使用される情報。認証情報はデータソースによって異なります。詳細は [BROKER LOAD](../../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。

## 例

### テーブルの全データを HDFS にエクスポート

次の例では、`testTbl` テーブルの全データを HDFS クラスターの `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/` パスにエクスポートします。

```SQL
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
WITH BROKER
(
        "username"="xxx",
        "password"="yyy"
);
```

### 指定したパーティションのテーブルデータを HDFS にエクスポート

次の例では、`testTbl` テーブルの `p1` と `p2` の 2 つのパーティションのデータを HDFS クラスターの `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/` パスにエクスポートします。

```SQL
EXPORT TABLE testTbl
PARTITION (p1,p2) 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
WITH BROKER
(
        "username"="xxx",
        "password"="yyy"
);
```

### カラムセパレータを指定してテーブルの全データを HDFS にエクスポート

次の例では、`testTbl` テーブルの全データを HDFS クラスターの `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/` パスにエクスポートし、カラムセパレータとしてコンマ (`,`) を使用するように指定します。

```SQL
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
PROPERTIES
(
    "column_separator"=","
) 
WITH BROKER
(
    "username"="xxx",
    "password"="yyy"
);
```

次の例では、`testTbl` テーブルの全データを HDFS クラスターの `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/` パスにエクスポートし、カラムセパレータとして `\x01` (Hive でサポートされているデフォルトのカラムセパレータ) を使用するように指定します。

```SQL
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
PROPERTIES
(
    "column_separator"="\\x01"
) 
WITH BROKER;
```

### ファイル名のプレフィックスを指定してテーブルの全データを HDFS にエクスポート

次の例では、`testTbl` テーブルの全データを HDFS クラスターの `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/testTbl_` パスにエクスポートし、エクスポートされたファイルの名前のプレフィックスとして `testTbl_` を使用することを指定します。

```SQL
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/testTbl_" 
WITH BROKER;
```

### AWS S3 にデータをエクスポート

次の例では、`testTbl` テーブルの全データを AWS S3 バケットの `s3a://s3-package/export/` パスにエクスポートします。

```SQL
EXPORT TABLE testTbl 
TO "s3a://s3-package/export/"
WITH BROKER
(
    "aws.s3.access_key" = "xxx",
    "aws.s3.secret_key" = "yyy",
    "aws.s3.region" = "zzz"
);
```
