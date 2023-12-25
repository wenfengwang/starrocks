---
displayed_sidebar: English
---

# 外部カタログの作成

## 説明

外部カタログを作成します。外部カタログを使用すると、StarRocks にデータをロードしたり、外部テーブルを作成したりすることなく、外部データソースのデータをクエリできます。現在、次のタイプの外部カタログを作成できます。

- [Hive カタログ](../../../data_source/catalog/hive_catalog.md): Apache Hive™ からのデータのクエリに使用されます。
- [Iceberg カタログ](../../../data_source/catalog/iceberg_catalog.md): Apache Iceberg からのデータのクエリに使用されます。
- [Hudi カタログ](../../../data_source/catalog/hudi_catalog.md): Apache Hudi からのデータをクエリするために使用されます。
- [Delta Lake カタログ](../../../data_source/catalog/deltalake_catalog.md): Delta Lake からのデータのクエリに使用されます。
- [JDBC カタログ](../../../data_source/catalog/jdbc_catalog.md): JDBC 互換データソースからのデータをクエリするために使用されます。

> **注記**
>
> - v3.0 以降、このステートメントには SYSTEM レベルの CREATE EXTERNAL CATALOG 権限が必要です。
> - 外部カタログを作成する前に、外部データソースのデータストレージシステム（Amazon S3 など）、メタデータサービス（Hive メタストアなど）、および認証サービス（Kerberos など）の要件を満たすように StarRocks クラスターを設定します。詳細については、各外部カタログトピックの「開始する前に」セクションを参照してください[](../../../data_source/catalog/catalog_overview.md)。

## 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

## パラメーター

| **パラメーター** | **必須** | **説明**                                              |
| ------------- | ------------ | ------------------------------------------------------------ |
| catalog_name  | はい          | 外部カタログの名前。命名規則は次のとおりです：<ul><li>名前には、文字、数字 (0 から 9)、およびアンダースコア (_) を含めることができます。文字で始まる必要があります。</li><li>名前は大文字と小文字が区別され、長さは 1023 文字を超えることはできません。</li></ul> |
| コメント       | いいえ           | 外部カタログの説明。 |
| PROPERTIES    | はい          | 外部カタログのプロパティ。外部カタログの種類に基づいてプロパティを構成します。詳細については、[Hive カタログ](../../../data_source/catalog/hive_catalog.md)、[Iceberg カタログ](../../../data_source/catalog/iceberg_catalog.md)、[Hudi カタログ](../../../data_source/catalog/hudi_catalog.md)、[Delta Lake カタログ](../../../data_source/catalog/deltalake_catalog.md)、および [JDBC カタログ](../../../data_source/catalog/jdbc_catalog.md) を参照してください。 |

## 例

例 1: `hive_metastore_catalog` という名前の Hive カタログを作成します。対応する Hive クラスターは、メタデータサービスとして Hive メタストアを使用します。

```SQL
CREATE EXTERNAL CATALOG hive_metastore_catalog
PROPERTIES(
   "type"="hive", 
   "hive.metastore.uris"="thrift://xx.xx.xx.xx:9083"
);
```

例 2: `hive_glue_catalog` という名前の Hive カタログを作成します。対応する Hive クラスターは、メタデータサービスとして AWS Glue を使用します。

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

例 3: `iceberg_metastore_catalog` という名前の Iceberg カタログを作成します。対応する Iceberg クラスターは、メタデータサービスとして Hive メタストアを使用します。

```SQL
CREATE EXTERNAL CATALOG iceberg_metastore_catalog
PROPERTIES(
    "type"="iceberg",
    "iceberg.catalog.type"="hive",
    "iceberg.catalog.hive.metastore.uris"="thrift://xx.xx.xx.xx:9083"
);
```

例 4: `iceberg_glue_catalog` という名前の Iceberg カタログを作成します。対応する Iceberg クラスターは、メタデータサービスとして AWS Glue を使用します。

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

例 5: `hudi_metastore_catalog` という名前の Hudi カタログを作成します。対応する Hudi クラスターは、メタデータサービスとして Hive メタストアを使用します。

```SQL
CREATE EXTERNAL CATALOG hudi_metastore_catalog
PROPERTIES(
    "type"="hudi",
    "hive.metastore.uris"="thrift://xx.xx.xx.xx:9083"
);
```

例 6: `hudi_glue_catalog` という名前の Hudi カタログを作成します。対応する Hudi クラスターは、メタデータサービスとして AWS Glue を使用します。

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

例 7: `delta_metastore_catalog` という名前の Delta Lake カタログを作成します。対応する Delta Lake サービスは、メタデータサービスとして Hive メタストアを使用します。

```SQL
CREATE EXTERNAL CATALOG delta_metastore_catalog
PROPERTIES(
    "type"="deltalake",
    "hive.metastore.uris"="thrift://xx.xx.xx.xx:9083"
);
```

例 8: `delta_glue_catalog` という名前の Delta Lake カタログを作成します。対応する Delta Lake サービスは、メタデータサービスとして AWS Glue を使用します。

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

- StarRocks クラスター内のすべてのカタログを表示するには、[SHOW CATALOGS](../data-manipulation/SHOW_CATALOGS.md) を参照してください。
- 外部カタログの作成ステートメントを表示するには、[SHOW CREATE CATALOG](../data-manipulation/SHOW_CREATE_CATALOG.md) を参照してください。
- StarRocks クラスターから外部カタログを削除するには、[DROP CATALOG](../data-definition/DROP_CATALOG.md) を参照してください。
