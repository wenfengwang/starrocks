---
displayed_sidebar: "Japanese"
---

# エクスポート

## 説明

指定された場所にテーブルのデータをエクスポートします。

これは非同期操作です。エクスポートタスクを送信した後でエクスポート結果が返されます。[SHOW EXPORT](../../../sql-reference/sql-statements/data-manipulation/SHOW_EXPORT.md) を使用してエクスポートタスクの進行状況を表示できます。

> **注意**
>
> StarRocks テーブルからデータをエクスポートするには、StarRocks テーブルに対するエクスポート権限を持つユーザーとしてのみデータをエクスポートできます。エクスポート権限がない場合は、[GRANT](../account-management/GRANT.md) で接続先の StarRocks クラスタに接続するユーザーにエクスポート権限を付与するように指示されます。

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

  テーブル名。StarRocks は `engine` が `olap` または `mysql` であるテーブルのデータをエクスポートできます。

- `partition_name`

  データをエクスポートしたいパーティション。このパラメータを指定しない場合は、デフォルトでテーブルのすべてのパーティションからデータがエクスポートされます。

- `column_name`

  データをエクスポートしたい列。このパラメータを使用して指定する列のシーケンスは、テーブルのスキーマと異なることがあります。このパラメータを指定しない場合は、デフォルトでテーブルのすべての列からデータがエクスポートされます。

- `export_path`

  テーブルのデータをエクスポートしたい場所。場所にパスが含まれる場合は、パスがスラッシュ (/) で終わることを確認してください。さもないと、パスの最後のスラッシュ (/) の後の部分がエクスポートされるファイルの名前の接頭辞として使用されます。ファイル名の接頭辞が指定されていない場合は、デフォルトで `data_` がファイル名の接頭辞として使用されます。

- `opt_properties`

  エクスポートタスクの構成を行うことができるオプションのプロパティ。

  構文:

  ```SQL
  [PROPERTIES ("<key>"="<value>", ...)]
  ```

  | **プロパティ**    | **説明**                                                     |
  | ---------------- | ------------------------------------------------------------ |
  | column_separator | エクスポートされたファイルで使用する列の区切り記号。デフォルト値: `\t`。 |
  | line_delimiter   | エクスポートされたファイルで使用する行の区切り記号。デフォルト値: `\n`。 |
  | load_mem_limit   | 各個々の BE でのエクスポートタスクに許可される最大メモリ。単位: バイト。デフォルトの最大メモリは 2 GB です。 |
  | timeout          | エクスポートタスクがタイムアウトするまでの時間。単位: 秒。デフォルト値: `86400`、つまり 1 日です。 |
  | include_query_id | エクスポートされたファイルの名前に `query_id` を含めるかを指定します。有効な値: `true` および `false`。値 `true` はファイル名に `query_id` が含まれることを指定し、値 `false` はファイル名に `query_id` が含まれないことを指定します。 |

- `WITH BROKER`

  v2.4 以前では、`WITH BROKER "<broker_name>"` を入力して使用するブローカーを指定します。v2.5 以降では、ブローカーを指定する必要はなくなりましたが、`WITH BROKER` キーワードは引き続き保持する必要があります。詳細については、[Export data using EXPORT > Background information](../../../unloading/Export.md#background-information) を参照してください。

- `broker_properties`

  ソースデータを認証するために使用される情報。データソースに応じて認証情報は異なります。詳細については、[BROKER LOAD](../../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。

## 例

### テーブルのすべてのデータを HDFS にエクスポートする

次の例では、`testTbl` テーブルのすべてのデータを HDFS クラスタの `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/` パスにエクスポートします:

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

次の例では、`testTbl` テーブルの `p1` と `p2` の 2 つのパーティションのデータを HDFS クラスタの `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/` パスにエクスポートします:

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

### 列区切り記号を指定してテーブルのすべてのデータを HDFS にエクスポートする

次の例では、`testTbl` テーブルのすべてのデータを HDFS クラスタの `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/` パスにエクスポートしながら、列区切り記号としてカンマ (`,`) を使用することを指定します:

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

次の例では、`testTbl` テーブルのすべてのデータを HDFS クラスタの `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/` パスにエクスポートしながら、Hive でサポートされているデフォルトの列区切り記号である `\x01` を列区切り記号として使用することを指定します:

```SQL
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
PROPERTIES
(
    "column_separator"="\\x01"
) 
WITH BROKER;
```

### ファイル名接頭辞を指定してテーブルのすべてのデータを HDFS にエクスポートする

次の例では、`testTbl` テーブルのすべてのデータを HDFS クラスタの `hdfs://<hdfs_host>:<hdfs_port>/a/b/c/testTbl_` パスにエクスポートしながら、`testTbl_` をエクスポートされるファイルの名前の接頭辞として使用することを指定します:

```SQL
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/testTbl_" 
WITH BROKER;
```

### AWS S3 にデータをエクスポートする

次の例では、`testTbl` テーブルのすべてのデータを AWS S3 バケットの `s3-package/export/` パスにエクスポートします:

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