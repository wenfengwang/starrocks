---
displayed_sidebar: English
---

# ALTER RESOURCE

## 説明

ALTER RESOURCE ステートメントを使用して、リソースのプロパティを変更できます。

## 構文

```SQL
ALTER RESOURCE 'resource_name' SET PROPERTIES ("key"="value", ...)
```

## パラメーター

- `resource_name`: 変更するリソースの名前です。

- `PROPERTIES ("key"="value", ...)`: リソースのプロパティです。リソースタイプに基づいて異なるプロパティを変更することができます。現在、StarRocksは以下のリソースに対してHiveメタストアのURIを変更することをサポートしています。
  - Apache Iceberg リソースは以下のプロパティの変更をサポートしています：
    - `iceberg.catalog-impl`: [カスタムカタログ](../../../data_source/External_table.md)の完全修飾クラス名。
    - `iceberg.catalog.hive.metastore.uris`: HiveメタストアのURI。
  - Apache Hive™ リソースと Apache Hudi リソースは `hive.metastore.uris` を変更することをサポートしており、これはHiveメタストアのURIを指します。

## 使用上の注意

リソースを参照して外部テーブルを作成した後、このリソースのHiveメタストアのURIを変更すると、外部テーブルは使用できなくなります。外部テーブルを使用してデータをクエリする場合は、新しいメタストアに元のメタストアと同じ名前とスキーマを持つテーブルが含まれていることを確認してください。

## 例

Hiveリソース`hive0`のHiveメタストアのURIを変更します。

```SQL
ALTER RESOURCE 'hive0' SET PROPERTIES ("hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083")
```
