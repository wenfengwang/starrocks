---
displayed_sidebar: Chinese
---

# SHOW CREATE CATALOG

## 機能

特定のExternal Catalog（例：Hive Catalog、Iceberg Catalog、Hudi Catalog、Delta Lake Catalog、JDBC Catalog）の作成SQL文を表示します。参照：[Hive Catalog](../../../data_source/catalog/hive_catalog.md)、[Iceberg Catalog](../../../data_source/catalog/iceberg_catalog.md)、[Hudi Catalog](../../../data_source/catalog/hudi_catalog.md)、[Delta Lake Catalog](../../../data_source/catalog/deltalake_catalog.md)、[JDBC Catalog](../../../data_source/catalog/jdbc_catalog.md)。認証に関連する秘密鍵情報はマスキングされており、表示されません。

このコマンドはバージョン3.0からサポートされています。

## 文法

```SQL
SHOW CREATE CATALOG <catalog_name>;
```

## パラメータ説明

| **パラメータ** | **必須** | **説明**               |
| -------------- | -------- | ---------------------- |
| catalog_name   | はい     | 表示するCatalogの名前。 |

## 戻り値の説明

```Plain
+------------+-----------------+
| Catalog    | Create Catalog  |
+------------+-----------------+
```

| **フィールド** | **説明**             |
| -------------- | -------------------- |
| Catalog        | Catalogの名前。      |
| Create Catalog | Catalogの作成SQL文。 |

## 例

`hive_catalog_glue` という名前のHive Catalogの作成SQL文を問い合わせる例：

```SQL
SHOW CREATE CATALOG hive_catalog_glue;
```

以下の情報が返されます：

```SQL
CREATE EXTERNAL CATALOG `hive_catalog_hms`
PROPERTIES ("aws.s3.access_key"  =  "AK******M4",
"hive.metastore.type"  =  "glue",
"aws.s3.secret_key"  =  "iV******iD",
"aws.glue.secret_key"  =  "iV******iD",
"aws.s3.use_instance_profile"  =  "false",
"aws.s3.region"  =  "us-west-1",
"aws.glue.region"  =  "us-west-1",
"type"  =  "hive",
"aws.glue.access_key"  =  "AK******M4"
)
```
