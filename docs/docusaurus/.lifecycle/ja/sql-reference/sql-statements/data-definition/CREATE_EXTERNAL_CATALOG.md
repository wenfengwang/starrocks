---
displayed_sidebar: "Japanese"
---

# 外部カタログの作成

## 説明

外部カタログを作成します。外部カタログを使用すると、StarRocksにデータをロードせず、外部データソースのデータをクエリできます。現在、次のタイプの外部カタログを作成できます。

- [Hiveカタログ](../../../data_source/catalog/hive_catalog.md): Apache Hive™からデータをクエリするためのものです。
- [Icebergカタログ](../../../data_source/catalog/iceberg_catalog.md): Apache Icebergからデータをクエリするためのものです。
- [Hudiカタログ](../../../data_source/catalog/hudi_catalog.md): Apache Hudiからデータをクエリするためのものです。
- [Delta Lakeカタログ](../../../data_source/catalog/deltalake_catalog.md): Delta Lakeからデータをクエリするためのものです。
- [JDBCカタログ](../../../data_source/catalog/jdbc_catalog.md): JDBC互換のデータソースからデータをクエリするためのものです。

> **注記**
>
> - v3.0以降では、このステートメントにはSYSTEMレベルのCREATE EXTERNAL CATALOG権限が必要です。
> - 外部カタログを作成する前に、StarRocksクラスタを外部データソースのデータストレージシステム（Amazon S3など）、メタデータサービス（Hiveメタストアなど）、および認証サービス（Kerberosなど）の要件を満たすように構成してください。詳細については、各[外部カタログのトピック](../../../data_source/catalog/catalog_overview.md)の「開始前に」セクションを参照してください。

## 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

## パラメータ

| **パラメータ** | **必須** | **説明**                                              |
| ------------- | ------------ | ------------------------------------------------------------ |
| catalog_name  | はい          | 外部カタログの名前。命名規則は次のとおりです:<ul><li>名前には、文字、数字（0-9）、アンダースコア（_）を含めることができます。文字で始める必要があります。</li><li>名前は大文字と小文字を区別し、1023文字を超えることはできません。</li></ul> |
| comment       | いいえ           | 外部カタログの説明。 |
| PROPERTIES    | はい          | 外部カタログのプロパティ。外部カタログのタイプに応じてプロパティを構成します。詳細については、[Hiveカタログ](../../../data_source/catalog/hive_catalog.md)、[Icebergカタログ](../../../data_source/catalog/iceberg_catalog.md)、[Hudiカタログ](../../../data_source/catalog/hudi_catalog.md)、[Delta Lakeカタログ](../../../data_source/catalog/deltalake_catalog.md)、および[JDBCカタログ](../../../data_source/catalog/jdbc_catalog.md)を参照してください。|

## 例

例1: `hive_metastore_catalog`というHiveカタログを作成します。対応するHiveクラスタは、メタデータサービスとしてHiveメタストアを使用しています。

```SQL
CREATE EXTERNAL CATALOG hive_metastore_catalog
PROPERTIES(
   "type"="hive", 
   "hive.metastore.uris"="thrift://x.x.x.x:9083"
);
```

例2: `hive_glue_catalog`というHiveカタログを作成します。対応するHiveクラスタは、メタデータサービスとしてAWS Glueを使用しています。

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

例3: `iceberg_metastore_catalog`というIcebergカタログを作成します。対応するIcebergクラスタは、メタデータサービスとしてHiveメタストアを使用しています。

```SQL
CREATE EXTERNAL CATALOG iceberg_metastore_catalog
PROPERTIES(
    "type"="iceberg",
    "iceberg.catalog.type"="hive",
    "iceberg.catalog.hive.metastore.uris"="thrift://x.x.x.x:9083"
);
```

例4: `iceberg_glue_catalog`というIcebergカタログを作成します。対応するIcebergクラスタは、メタデータサービスとしてAWS Glueを使用しています。

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

例5: `hudi_metastore_catalog`というHudiカタログを作成します。対応するHudiクラスタは、メタデータサービスとしてHiveメタストアを使用しています。

```SQL
CREATE EXTERNAL CATALOG hudi_metastore_catalog
PROPERTIES(
    "type"="hudi",
    "hive.metastore.uris"="thrift://x.x.x.x:9083"
);
```

例6: `hudi_glue_catalog`というHudiカタログを作成します。対応するHudiクラスタは、メタデータサービスとしてAWS Glueを使用しています。

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

例7: `delta_metastore_catalog`というDelta Lakeカタログを作成します。対応するDelta Lakeサービスは、メタデータサービスとしてHiveメタストアを使用しています。

```SQL
CREATE EXTERNAL CATALOG delta_metastore_catalog
PROPERTIES(
    "type"="deltalake",
    "hive.metastore.uris"="thrift://x.x.x.x:9083"
);
```

例8: `delta_glue_catalog`というDelta Lakeカタログを作成します。対応するDelta Lakeサービスは、メタデータサービスとしてAWS Glueを使用しています。

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