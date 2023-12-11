---
displayed_sidebar: "英語"
---

# 外部カタログを作成する

## 説明

外部カタログを作成します。外部データソースのデータをStarRocksにロードせず、外部テーブルを作成せずにクエリするために外部カタログを使用できます。現在、次のタイプの外部カタログを作成できます。

- [Hiveカタログ](../../../data_source/catalog/hive_catalog.md): Apache Hive™ からデータをクエリするために使用されます。
- [Icebergカタログ](../../../data_source/catalog/iceberg_catalog.md): Apache Iceberg からデータをクエリするために使用されます。
- [Hudiカタログ](../../../data_source/catalog/hudi_catalog.md): Apache Hudi からデータをクエリするために使用されます。
- [Delta Lakeカタログ](../../../data_source/catalog/deltalake_catalog.md): Delta Lake からデータをクエリするために使用されます。
- [JDBCカタログ](../../../data_source/catalog/jdbc_catalog.md): JDBC互換のデータソースからデータをクエリするために使用されます。

> **注**
>
> - v3.0以降、このステートメントの実行にはSYSTEMレベルのCREATE EXTERNAL CATALOG権限が必要です。
> - 外部カタログを作成する前に、StarRocksクラスタを外部データソースのデータストレージシステム（たとえばAmazon S3）、メタデータサービス（たとえばHiveメタストア）、認証サービス（たとえばKerberos）の要件を満たすように構成してください。詳細については、各[外部カタログのトピック](../../../data_source/catalog/catalog_overview.md)の"はじめる前に"セクションを参照してください。

## 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

## パラメータ

| **パラメータ** | **必須** | **説明**                                              |
| ------------- | ------------ | ------------------------------------------------------------ |
| catalog_name  | はい          | 外部カタログの名前。命名規則は次のとおりです:<ul><li>名前は文字、数字（0-9）、アンダースコア(_)を含めることができます。文字で始める必要があります。</li><li>名前は大文字と小文字を区別し、長さが1023文字を超えることはできません。</li></ul> |
| comment       | いいえ           | 外部カタログの説明 |
| PROPERTIES    | はい          | 外部カタログのプロパティ。外部カタログのタイプに基づいてプロパティを構成してください。詳細については、[Hiveカタログ](../../../data_source/catalog/hive_catalog.md)、[Icebergカタログ](../../../data_source/catalog/iceberg_catalog.md)、[Hudiカタログ](../../../data_source/catalog/hudi_catalog.md)、[Delta Lakeカタログ](../../../data_source/catalog/deltalake_catalog.md)、[JDBCカタログ](../../../data_source/catalog/jdbc_catalog.md)を参照してください。 |

## 例

Example 1: `hive_metastore_catalog` というHiveカタログを作成します。対応するHiveクラスタはHiveメタストアをそのメタデータサービスとして使用しています。

```SQL
CREATE EXTERNAL CATALOG hive_metastore_catalog
PROPERTIES(
   "type"="hive", 
   "hive.metastore.uris"="thrift://x.x.x.x:9083"
);
```

Example 2: `hive_glue_catalog` というHiveカタログを作成します。対応するHiveクラスタはAWS Glueをそのメタデータサービスとして使用しています。

```SQL
CREATE EXTERNAL CATALOG hive_glue_catalog
PROPERTIES(
    "type"="hive", 
    "hive.metastore.type"="glue",
    "aws.hive.metastore.glue.aws-access-key"="xxxxxx",
    "aws.hive.metastore.glue.aws-secret-key"="xxxxxxxxxxxx",
    "aws.hive.metastore.glue.endpoint"="https://glue.x-x-x.amazonaws.com"
);
```

Example 3: `iceberg_metastore_catalog` というIcebergカタログを作成します。対応するIcebergクラスタはHiveメタストアをそのメタデータサービスとして使用しています。

```SQL
CREATE EXTERNAL CATALOG iceberg_metastore_catalog
PROPERTIES(
    "type"="iceberg",
    "iceberg.catalog.type"="hive",
    "iceberg.catalog.hive.metastore.uris"="thrift://x.x.x.x:9083"
);
```

Example 4: `iceberg_glue_catalog` というIcebergカタログを作成します。対応するIcebergクラスタはAWS Glueをそのメタデータサービスとして使用しています。

```SQL
CREATE EXTERNAL CATALOG iceberg_glue_catalog
PROPERTIES(
    "type"="iceberg", 
    "iceberg.catalog.type"="glue",
    "aws.hive.metastore.glue.aws-access-key"="xxxxx",
    "aws.hive.metastore.glue.aws-secret-key"="xxxxxxxxxxxx",
    "aws.hive.metastore.glue.endpoint"="https://glue.x-x-x.amazonaws.com"
);
```

Example 5: `hudi_metastore_catalog` というHudiカタログを作成します。対応するHudiクラスタはHiveメタストアをそのメタデータサービスとして使用しています。

```SQL
CREATE EXTERNAL CATALOG hudi_metastore_catalog
PROPERTIES(
    "type"="hudi",
    "hive.metastore.uris"="thrift://x.x.x.x:9083"
);
```

Example 6: `hudi_glue_catalog` というHudiカタログを作成します。対応するHudiクラスタはAWS Glueをそのメタデータサービスとして使用しています。

```SQL
CREATE EXTERNAL CATALOG hudi_glue_catalog
PROPERTIES(
    "type"="hudi", 
    "hive.metastore.type"="glue",
    "aws.hive.metastore.glue.aws-access-key"="xxxxxx",
    "aws.hive.metastore.glue.aws-secret-key"="xxxxxxxxxxxx",
    "aws.hive.metastore.glue.endpoint"="https://glue.x-x-x.amazonaws.com"
);
```

Example 7: `delta_metastore_catalog` というDelta Lakeカタログを作成します。対応するDelta LakeサービスはHiveメタストアをそのメタデータサービスとして使用しています。

```SQL
CREATE EXTERNAL CATALOG delta_metastore_catalog
PROPERTIES(
    "type"="deltalake",
    "hive.metastore.uris"="thrift://x.x.x.x:9083"
);
```

Example 8: `delta_glue_catalog` というDelta Lakeカタログを作成します。対応するDelta LakeサービスはAWS Glueをそのメタデータサービスとして使用しています。

```SQL
CREATE EXTERNAL CATALOG delta_glue_catalog
PROPERTIES(
    "type"="deltalake", 
    "hive.metastore.type"="glue",
    "aws.hive.metastore.glue.aws-access-key"="xxxxxx",
    "aws.hive.metastore.glue.aws-secret-key"="xxxxxxxxxxxx",
    "aws.hive.metastore.glue.endpoint"="https://glue.x-x-x.amazonaws.com"
);
```

## 参照

- StarRocksクラスタ内のすべてのカタログを表示するには、[SHOW CATALOGS](../data-manipulation/SHOW_CATALOGS.md)を参照してください。
- 外部カタログの作成ステートメントを表示するには、[SHOW CREATE CATALOG](../data-manipulation/SHOW_CREATE_CATALOG.md)を参照してください。
- StarRocksクラスタから外部カタログを削除するには、[DROP CATALOG](../data-definition/DROP_CATALOG.md)を参照してください。