---
displayed_sidebar: Chinese
---

# CREATE EXTERNAL CATALOG

## 機能

このステートメントは、External Catalogを作成するために使用されます。作成後、データのインポートや外部テーブルの作成を行わずに外部データをクエリすることができます。現在、以下のExternal Catalogの作成がサポートされています：

- [Hive catalog](../../../data_source/catalog/hive_catalog.md)：Apache Hive™ クラスター内のデータをクエリするために使用します。
- [Iceberg catalog](../../../data_source/catalog/iceberg_catalog.md)：Apache Iceberg クラスター内のデータをクエリするために使用します。
- [Hudi catalog](../../../data_source/catalog/hudi_catalog.md)：Apache Hudi クラスター内のデータをクエリするために使用します。
- [Delta Lake catalog](../../../data_source/catalog/deltalake_catalog.md)：Delta Lakeのデータをクエリするために使用します。
- [JDBC catalog](../../../data_source/catalog/jdbc_catalog.md)：JDBCデータソースのデータをクエリするために使用します。

> **注意**
>
> - バージョン3.0以降では、このステートメントを使用するにはSystemレベルのCREATE EXTERNAL CATALOG権限が必要です。
> - External Catalogを作成する前に、データソースのストレージシステム（例：AWS S3）、メタデータサービス（例：Hive metastore）、認証方式（例：Kerberos）に応じた設定をStarRocksで行う必要があります。詳細は、上記の各External Catalogのドキュメント内の「前提条件」セクションを参照してください。

## 文法

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

## パラメータ説明

| パラメータ     | 必須 | 説明                                                         |
| ------------ | ---- | ------------------------------------------------------------ |
| catalog_name | はい  | External Catalogの名前。命名規則は以下の通りです：<ul><li>英字（a-zまたはA-Z）、数字（0-9）、アンダースコア（_）のみを使用し、英字で始める必要があります。</li><li>全長は1023文字を超えることはできません。</li><li>Catalog名は大文字と小文字を区別します。</li></ul> |
| comment      | いいえ | External Catalogの説明。 |
| PROPERTIES   | はい  | External Catalogの属性。異なるExternal Catalogには異なる属性が必要です。詳細な設定情報については、[Hive catalog](../../../data_source/catalog/hive_catalog.md)、[Iceberg catalog](../../../data_source/catalog/iceberg_catalog.md)、[Hudi catalog](../../../data_source/catalog/hudi_catalog.md)、[Delta Lake catalog](../../../data_source/catalog/deltalake_catalog.md)、および[JDBC Catalog](../../../data_source/catalog/jdbc_catalog.md)を参照してください。 |

## 例

例1：`hive_metastore_catalog`という名前のHive catalogを作成します。対応するHiveクラスターはHive metastoreをメタデータサービスとして使用します。

```SQL
CREATE EXTERNAL CATALOG hive_metastore_catalog
PROPERTIES(
   "type"="hive", 
   "hive.metastore.uris"="thrift://xx.xx.xx.xx:9083"
);
```

例2：`hive_glue_catalog`という名前のHive catalogを作成します。対応するHiveクラスターはAWS Glueをメタデータサービスとして使用します。

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

例3：`iceberg_metastore_catalog`という名前のIceberg catalogを作成します。対応するIcebergクラスターはHive metastoreをメタデータサービスとして使用します。

```SQL
CREATE EXTERNAL CATALOG iceberg_metastore_catalog
PROPERTIES(
    "type"="iceberg",
    "iceberg.catalog.type"="hive",
    "iceberg.catalog.hive.metastore.uris"="thrift://xx.xx.xx.xx:9083"
);
```

例4：`iceberg_glue_catalog`という名前のIceberg catalogを作成します。対応するIcebergクラスターはAWS Glueをメタデータサービスとして使用します。

```SQL
CREATE EXTERNAL CATALOG iceberg_glue_catalog
PROPERTIES(
    "type"="iceberg", 
    "iceberg.catalog.type"="glue",
    "aws.hive.metastore.glue.aws-access-key"="xxxxx",
    "aws.hive.metastore.glue.aws-secret-key"="xxx",
    "aws.hive.metastore.glue.endpoint"="https://glue.x-x-x.amazonaws.com"
);
```

例5：`hudi_metastore_catalog`という名前のHudi catalogを作成します。対応するHudiクラスターはHive metastoreをメタデータサービスとして使用します。

```SQL
CREATE EXTERNAL CATALOG hudi_metastore_catalog
PROPERTIES(
    "type"="hudi",
    "hive.metastore.uris"="thrift://xx.xx.xx.xx:9083"
);
```

例6：`hudi_glue_catalog`という名前のHudi catalogを作成します。対応するHudiクラスターはAWS Glueをメタデータサービスとして使用します。

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

例7：`delta_metastore_catalog`という名前のDelta Lake catalogを作成します。対応するDelta LakeはHive metastoreをメタデータサービスとして使用します。

```SQL
CREATE EXTERNAL CATALOG delta_metastore_catalog
PROPERTIES(
    "type"="deltalake",
    "hive.metastore.uris"="thrift://xx.xx.xx.xx:9083"
);
```

例8：`delta_glue_catalog`という名前のDelta Lake catalogを作成します。対応するDelta LakeはAWS Glueをメタデータサービスとして使用します。

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

## 関連操作

- すべてのCatalogを表示するには、[SHOW CATALOGS](../../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を参照してください。
- 特定のExternal Catalogの作成ステートメントを表示するには、[SHOW CREATE CATALOG](../../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を参照してください。
- External Catalogを削除するには、[DROP CATALOG](../../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を参照してください。
