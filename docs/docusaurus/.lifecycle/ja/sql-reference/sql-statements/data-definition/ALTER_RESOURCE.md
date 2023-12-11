---
displayed_sidebar: "Japanese"
---

# リソースの変更

## 説明

ALTER RESOURCE 文を使用してリソースのプロパティを変更できます。

## 構文

```SQL
ALTER RESOURCE 'resource_name' SET PROPERTIES ("key"="value", ...)
```

## パラメータ

- `resource_name`: 変更されるリソースの名前。

- `PROPERTIES ("key"="value", ...)`: リソースのプロパティ。リソースの種類に基づいて異なるプロパティを変更できます。現在、StarRocks は次のリソースの Hive メタストアの URI を変更することをサポートしています。
  - Apache Iceberg リソースは、以下のプロパティを変更できます。
    - `iceberg.catalog-impl`: [カスタムカタログ](../../../data_source/External_table.md)の完全修飾クラス名。
    - `iceberg.catalog.hive.metastore.uris`: Hive メタストアの URI。
  - Apache Hive™ リソースおよび Apache Hudi リソースは `hive.metastore.uris` を変更できます。これは Hive メタストアの URI を示します。

## 使用上の注意

外部テーブルを作成するためにリソースを参照した後、このリソースの Hive メタストアの URI を変更すると、外部テーブルは利用できなくなります。引き続き外部テーブルを使用してデータをクエリする場合は、新しいメタストアに、元のメタストアと同じ名前とスキーマのテーブルが含まれていることを確認してください。

## 例

Hive リソース `hive0` の Hive メタストアの URI を変更する。

```SQL
ALTER RESOURCE 'hive0' SET PROPERTIES ("hive.metastore.uris" = "thrift://10.10.44.91:9083")
```