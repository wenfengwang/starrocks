---
displayed_sidebar: "English"
---

# カタログの作成を表示

## 説明

外部カタログ（Hive、Iceberg、Hudi、Delta Lake、JDBCカタログなど）の作成文を問い合わせます。[Hiveカタログ](../../../data_source/catalog/hive_catalog.md)、[Icebergカタログ](../../../data_source/catalog/iceberg_catalog.md)、[Hudiカタログ](../../../data_source/catalog/hudi_catalog.md)、[Delta Lakeカタログ](../../../data_source/catalog/deltalake_catalog.md)、および[JDBCカタログ](../../../data_source/catalog/jdbc_catalog.md)を参照してください。戻り結果の認証関連情報は匿名化されます。

このコマンドはv3.0からサポートされています。

## 構文

```SQL
SHOW CREATE CATALOG <catalog_name>;
```

## パラメータ

| **パラメータ** | **必須** | **説明**                          |
| --------------- | -------- | --------------------------------- |
| catalog_name    | Yes      | 表示したいカタログの名前。            |

## 戻り結果

```Plain
+------------+-----------------+
| カタログ    | カタログを作成     |
+------------+-----------------+
```

| **フィールド**  | **説明**                       |
| ---------------- | ---------------------------- |
| カタログ           | カタログの名前。               |
| カタログを作成     | カタログを作成した文。          |

## 例

次の例は、`hive_catalog_hms`という名前のHiveカタログの作成文を問い合わせます：

```SQL
SHOW CREATE CATALOG hive_catalog_hms;
```

戻り結果は次の通りです：

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