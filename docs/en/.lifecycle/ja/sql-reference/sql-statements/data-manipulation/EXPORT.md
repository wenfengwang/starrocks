---
displayed_sidebar: "Japanese"
---

# EXPORT（エクスポート）

## 説明

指定された場所にテーブルのデータをエクスポートします。

これは非同期操作です。エクスポートタスクを送信した後、エクスポート結果が返されます。エクスポートタスクの進行状況を表示するには、[SHOW EXPORT](../../../sql-reference/sql-statements/data-manipulation/SHOW_EXPORT.md) を使用できます。

> **注意**
>
> StarRocks テーブルからデータをエクスポートするには、StarRocks テーブルに対して EXPORT 権限を持つユーザーとしてのみエクスポートできます。EXPORT 権限を持っていない場合は、StarRocks クラスタに接続するために使用するユーザーに EXPORT 権限を付与するための[GRANT](../account-management/GRANT.md) の手順に従ってください。

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

## パラメータ

- `table_name`

  テーブルの名前です。StarRocks は `engine` が `olap` または `mysql` のテーブルのデータをエクスポートできます。

- `partition_name`

  データをエクスポートしたいパーティションです。このパラメータを指定しない場合、デフォルトでテーブルのすべてのパーティションからデータがエクスポートされます。

- `column_name`

  データをエクスポートしたい列です。このパラメータを使用して指定する列の順序は、テーブルのスキーマと異なる場合があります。このパラメータを指定しない場合、デフォルトでテーブルのすべての列からデータがエクスポートされます。

- `export_path`

  テーブルのデータをエクスポートする場所です。場所にパスが含まれている場合は、パスの末尾がスラッシュ (/) で終わることを確認してください。そうでない場合、パスの最後のスラッシュ (/) の後に続く部分がエクスポートされたファイルの名前の接頭辞として使用されます。デフォルトでは、ファイル名の接頭辞として `data_` が使用されます。

- `opt_properties`

  エクスポートタスクのために設定できるオプションのプロパティです。

  構文:

  ```SQL
  [PROPERTIES ("<key>"="<value>", ...)]
  ```

  | **プロパティ**     | **説明**                                                     |
  | ---------------- | ------------------------------------------------------------ |
  | column_separator | エクスポートされたファイルで使用する列セパレータです。デフォルト値: `\t`。 |
  | line_delimiter   | エクスポートされたファイルで使用する行セパレータです。デフォルト値: `\n`。 |
  | load_mem_limit   | 各個別の BE でエクスポートタスクに許可される最大メモリです。単位: バイト。デフォルトの最大メモリは 2 GB です。 |
  | timeout          | エクスポートタスクがタイムアウトするまでの時間です。単位: 秒。デフォルト値: `86400`（1 日）。 |
  | include_query_id | エクスポートされたファイルの名前に `query_id` が含まれるかどうかを指定します。有効な値: `true`、`false`。`true` の場合、ファイル名に `query_id` が含まれ、`false` の場合、ファイル名に `query_id` が含まれません。 |

- `WITH BROKER`

  v2.4 以前では、`WITH BROKER "<broker_name>"` を入力して使用するブローカーを指定します。v2.5 以降では、ブローカーを指定する必要はなくなりましたが、`WITH BROKER` キーワードを保持する必要があります。詳細については、[EXPORT を使用してデータをエクスポートする > 背景情報](../../../unloading/Export.md#background-information) を参照してください。

- `broker_properties`

  ソースデータの認証に使用される情報です。認証情報はデータソースによって異なります。詳細については、[BROKER LOAD](../../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。

## 例

### テーブルのすべてのデータを HDFS にエクスポートする

次の例では、`testTbl` テーブルのすべてのデータを HDFS クラスタの `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/` パスにエクスポートします。

```SQL
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
WITH BROKER
(
        "username"="xxx",
        "password"="yyy"
);
```

### テーブルの指定されたパーティションのデータを HDFS にエクスポートする

次の例では、`testTbl` テーブルの `p1` と `p2` の 2 つのパーティションのデータを HDFS クラスタの `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/` パスにエクスポートします。

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

### テーブルのすべてのデータを HDFS にエクスポートし、列セパレータを指定する

次の例では、`testTbl` テーブルのすべてのデータを HDFS クラスタの `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/` パスにエクスポートし、カンマ (`,`) を列セパレータとして使用することを指定します。

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

次の例では、`testTbl` テーブルのすべてのデータを HDFS クラスタの `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/` パスにエクスポートし、Hive でサポートされているデフォルトの列セパレータである `\x01` を列セパレータとして使用することを指定します。

```SQL
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
PROPERTIES
(
    "column_separator"="\\x01"
) 
WITH BROKER;
```

### テーブルのすべてのデータを HDFS にエクスポートし、ファイル名の接頭辞を指定する

次の例では、`testTbl` テーブルのすべてのデータを HDFS クラスタの `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/` パスにエクスポートし、エクスポートされたファイルの名前の接頭辞として `testTbl_` を指定します。

```SQL
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/testTbl_" 
WITH BROKER;
```

### AWS S3 にデータをエクスポートする

次の例では、`testTbl` テーブルのすべてのデータを AWS S3 バケットの `s3-package/export/` パスにエクスポートします。

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
