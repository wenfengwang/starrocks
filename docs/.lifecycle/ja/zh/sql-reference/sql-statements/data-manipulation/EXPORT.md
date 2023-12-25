---
displayed_sidebar: Chinese
---

# EXPORT

## 機能

このステートメントは、指定されたテーブルのデータを指定された場所にエクスポートするために使用されます。

これは非同期操作で、タスクが正常に送信されると結果が返されます。実行後は [SHOW EXPORT](../../../sql-reference/sql-statements/data-manipulation/SHOW_EXPORT.md) コマンドを使用して進捗状況を確認できます。

> **注意**
>
> エクスポート操作には、対象テーブルのEXPORT権限が必要です。もしEXPORT権限がない場合は、[GRANT](../account-management/GRANT.md) を参照してユーザーに権限を付与してください。

## 文法

```sql
EXPORT TABLE <table_name>
[PARTITION (<partition_name>[, ...])]
[(<column_name>[, ...])]
TO <export_path>
[opt_properties]
WITH BROKER
[broker_properties]
```

## パラメータ説明

- `table_name`

  エクスポートするデータがあるテーブル。現在、`engine` が `olap` または `mysql` のテーブルのエクスポートがサポートされています。

- `partition_name`

  エクスポートするパーティション。指定しない場合は、デフォルトでテーブル内のすべてのパーティションのデータがエクスポートされます。

- `column_name`

  エクスポートする列。列のエクスポート順序は、元のテーブルのスキーマと異なることがあります。指定しない場合は、デフォルトでテーブル内のすべての列のデータがエクスポートされます。

- `export_path`

  エクスポート先のパス。ディレクトリの場合は、スラッシュ (/) で終わる必要があります。そうでない場合、最後のスラッシュの後の部分はエクスポートファイルのプレフィックスとして使用されます。ファイル名プレフィックスを指定しない場合、デフォルトのファイル名プレフィックスは `data_` です。

- `opt_properties`

  エクスポートに関連するプロパティ設定。

  文法：

  ```sql
  [PROPERTIES ("<key>"="<value>", ...)]
  ```

  設定項目：

  | **設定項目**         | **説明**                                                     |
  | ---------------- | ------------------------------------------------------------ |
  | column_separator | エクスポートファイルの列区切り文字を指定します。デフォルト値：`\t`。                       |
  | line_delimiter   | エクスポートファイルの行区切り文字を指定します。デフォルト値：`\n`。                       |
  | load_mem_limit   | エクスポートタスクが単一のBEノード上で使用するメモリの上限を指定します。単位：バイト。デフォルトのメモリ使用上限は2GBです。 |
  | timeout          | エクスポートタスクのタイムアウト時間を指定します。単位：秒。デフォルト値：`86400`（1日）。  |
  | include_query_id | エクスポートファイル名に `query_id` を含めるかどうかを指定します。値の範囲：`true` または `false`。デフォルト値：`true`。`true` は含めることを意味し、`false` は含めないことを意味します。 |

- `WITH BROKER`

  v2.4 以前のバージョンでは、`WITH BROKER "<broker_name>"` を使用して、どのBrokerを使用するかをエクスポートステートメントで指定する必要がありました。v2.5以降では `broker_name` を指定する必要はありませんが、`WITH BROKER` キーワードは引き続き保持されています。[EXPORTを使用してデータをエクスポートする > 背景情報](../../../unloading/Export.md#背景情報)を参照してください。

- `broker_properties`

  データソースへのアクセス認証情報を提供するために使用されます。データソースが異なるため、提供する必要がある認証情報も異なります。[BROKER LOAD](../../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

## 例

### HDFSにテーブル内のすべてのデータをエクスポートする

`testTbl` テーブル内のすべてのデータをHDFSクラスターの指定されたパスにエクスポートします。

```sql
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
WITH BROKER
(
    "username"="xxx",
    "password"="yyy"
);
```

### HDFSにテーブルの特定のパーティションのデータをエクスポートする

`testTbl` テーブルの `p1` と `p2` パーティションのデータをHDFSクラスターの指定されたパスにエクスポートします。

```sql
EXPORT TABLE testTbl
PARTITION (p1,p2) 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
WITH BROKER
(
    "username"="xxx",
    "password"="yyy"
);
```

### 区切り文字を指定する

`testTbl` テーブル内のすべてのデータをHDFSクラスターの指定されたパスにエクスポートし、`,` をエクスポートファイルの列区切り文字として使用します。

```sql
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

Hiveのデフォルト区切り文字 `\x01` をエクスポートファイルの列区切り文字として使用して、`testTbl` テーブル内のすべてのデータをHDFSクラスターの指定されたパスにエクスポートします。

```sql
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
PROPERTIES
(
    "column_separator"="\\x01"
) 
WITH BROKER;
```

### エクスポートファイル名のプレフィックスを指定する

`testTbl_` をエクスポートファイルのファイル名プレフィックスとして使用して、`testTbl` テーブル内のすべてのデータをHDFSクラスターの指定されたパスにエクスポートします。

```sql
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/testTbl_" 
WITH BROKER;
```

### Alibaba Cloud OSSにデータをエクスポートする

`testTbl` テーブル内のすべてのデータをAlibaba Cloud OSSの指定されたバケットパスにエクスポートします。

```sql
EXPORT TABLE testTbl 
TO "oss://oss-package/export/"
WITH BROKER
(
    "fs.oss.accessKeyId" = "xxx",
    "fs.oss.accessKeySecret" = "yyy",
    "fs.oss.endpoint" = "oss-cn-zhangjiakou-internal.aliyuncs.com"
);
```

### Tencent Cloud COSにデータをエクスポートする

`testTbl` テーブル内のすべてのデータをTencent Cloud COSの指定されたバケットパスにエクスポートします。

```sql
EXPORT TABLE testTbl 
TO "cosn://cos-package/export/"
WITH BROKER
(
    "fs.cosn.userinfo.secretId" = "xxx",
    "fs.cosn.userinfo.secretKey" = "yyy",
    "fs.cosn.bucket.endpoint_suffix" = "cos.ap-beijing.myqcloud.com"
);
```

### AWS S3にデータをエクスポートする

`testTbl` テーブル内のすべてのデータをAWS S3の指定されたバケットパスにエクスポートします。

```sql
EXPORT TABLE testTbl 
TO "s3a://s3-package/export/"
WITH BROKER
(
    "aws.s3.access_key" = "xxx",
    "aws.s3.secret_key" = "yyy",
    "aws.s3.region" = "zzz"
);
```

### Huawei Cloud OBSにデータをエクスポートする

`testTbl` テーブル内のすべてのデータをHuawei Cloud OBSの指定されたバケットパスにエクスポートします。

```sql
EXPORT TABLE testTbl 
TO "obs://obs-package/export/"
WITH BROKER
(
    "fs.obs.access.key" = "xxx",
    "fs.obs.secret.key" = "yyy",
    "fs.obs.endpoint" = "obs.cn-east-3.myhuaweicloud.com"
);
```

> **説明**
>
> Huawei Cloud OBSへデータをエクスポートする場合は、[依存ライブラリ](https://github.com/huaweicloud/obsa-hdfs/releases/download/v45/hadoop-huaweicloud-2.8.3-hw-45.jar)をダウンロードして **$BROKER_HOME/lib/** に追加し、Brokerを再起動する必要があります。
