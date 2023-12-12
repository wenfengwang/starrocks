---
displayed_sidebar: "Japanese"
---

# EXPORT（エクスポート）

## 説明

特定の場所にあるテーブルのデータをエクスポートします。

これは非同期操作です。エクスポートタスクを提出した後でエクスポート結果が返されます。[SHOW EXPORT](../../../sql-reference/sql-statements/data-manipulation/SHOW_EXPORT.md)を使用してエクスポートタスクの進行状況を表示できます。

> **注意**
>
> StarRocksテーブルからデータをエクスポートする権限を持つユーザーだけがデータをエクスポートできます。EXPORT権限を持っていない場合は、[GRANT](../account-management/GRANT.md)でStarRocksクラスタに接続するユーザーにEXPORT権限を付与する手順に従ってください。

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

  テーブルの名前。StarRocksは`engine`が`olap`または`mysql`であるテーブルのデータをエクスポートできます。

- `partition_name`

  データをエクスポートしたいパーティション。デフォルトでは、このパラメータを指定しない場合、StarRocksはテーブルのすべてのパーティションからデータをエクスポートします。

- `column_name`

  データをエクスポートしたい列。このパラメータを使用して指定する列のシーケンスは、テーブルのスキーマと異なる場合があります。デフォルトでは、このパラメータを指定しない場合、StarRocksはテーブルのすべての列からデータをエクスポートします。

- `export_path`

  テーブルのデータをエクスポートする場所。場所にパスが含まれる場合は、パスがスラッシュ(/)で終わることを確認してください。さもないと、パスの最後のスラッシュ(/)の後に続く部分がエクスポートされたファイルの名前のプレフィックスとして使用されます。デフォルトでは、ファイル名のプレフィックスには`data_`が使用されます。

- `opt_properties`

  エクスポートタスクの構成を設定できるオプションのプロパティ。

  構文:

  ```SQL
  [PROPERTIES ("<key>"="<value>", ...)]
  ```

  | **プロパティ**  | **説明**                              |
  | ---------------- | -------------------------------------- |
  | column_separator | エクスポートされたファイルで使用する列のセパレータ。デフォルト値: `\t`。 |
  | line_delimiter   | エクスポートされたファイルで使用する改行のセパレータ。デフォルト値: `\n`。 |
  | load_mem_limit   | 個々のBE上のエクスポートタスクに許可される最大メモリ。単位: バイト。デフォルトの最大メモリは2 GBです。 |
  | timeout          | エクスポートタスクがタイムアウトする時間。単位: 秒。デフォルト値: `86400`、つまり1日です。 |
  | include_query_id | エクスポートされたファイルの名前に`query_id`が含まれるかどうかを指定します。有効な値: `true` および `false`。`true`の場合、ファイルの名前に`query_id`が含まれ、`false`の場合、ファイルの名前に`query_id`が含まれません。 |

- `WITH BROKER`

  v2.4およびそれ以前では、`WITH BROKER "<broker_name>"`を入力して使用するブローカーを指定します。v2.5以降では、ブローカーを指定する必要はなくなりましたが、依然として`WITH BROKER`キーワードを保持する必要があります。詳細については、[Export data using EXPORT > Background information](../../../unloading/Export.md#background-information)を参照してください。

- `broker_properties`

  ソースデータの認証に使用される情報。認証情報はデータソースによって異なります。詳細については、[BROKER LOAD](../../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

## 例

### テーブルのすべてのデータをHDFSにエクスポートする

次の例では、`testTbl`テーブルのすべてのデータをHDFSクラスタの`hdfs://<hdfs_host>:<hdfs_port>/a/b/c/`パスにエクスポートします。

```SQL
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
WITH BROKER
(
        "username"="xxx",
        "password"="yyy"
);
```

### テーブルの指定されたパーティションのデータをHDFSにエクスポートする

次の例では、`testTbl`テーブルの2つのパーティション`p1`および`p2`のデータをHDFSクラスタの`hdfs://<hdfs_host>:<hdfs_port>/a/b/c/`パスにエクスポートします。

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

### テーブルのすべてのデータをHDFSにエクスポートし、指定した列セパレータを使用する

次の例では、`testTbl`テーブルのすべてのデータをHDFSクラスタの`hdfs://<hdfs_host>:<hdfs_port>/a/b/c/`パスにエクスポートし、コンマ(,)を列のセパレータとして使用することを指定します。

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

次の例では、`testTbl`テーブルのすべてのデータをHDFSクラスタの`hdfs://<hdfs_host>:<hdfs_port>/a/b/c/`パスにエクスポートし、Hiveがサポートするデフォルトの列セパレータである`\x01`を列のセパレータとして使用することを指定します。

```SQL
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
PROPERTIES
(
    "column_separator"="\\x01"
) 
WITH BROKER;
```

### テーブルのすべてのデータをHDFSにエクスポートし、エクスポートされたファイルの名前プレフィックスを指定する

次の例では、`testTbl`テーブルのすべてのデータをHDFSクラスタの`hdfs://<hdfs_host>:<hdfs_port>/a/b/c/testTbl_`パスにエクスポートし、`testTbl_`をエクスポートされたファイルの名前のプレフィックスとして指定します。

```SQL
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/testTbl_" 
WITH BROKER;
```

### データをAWS S3にエクスポートする

次の例は、`testTbl`テーブルのすべてのデータをAWS S3バケットの`s3-package/export/`パスにエクスポートします。

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