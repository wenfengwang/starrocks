---
displayed_sidebar: "Japanese"
---

# ALTER RESOURCE（リソースの変更）

## 説明

ALTER RESOURCEステートメントを使用して、リソースのプロパティを変更することができます。

## 構文

```SQL
ALTER RESOURCE 'リソース名' SET PROPERTIES ("キー"="値", ...)
```

## パラメータ

- `リソース名`：変更するリソースの名前。

- `PROPERTIES ("キー"="値", ...)`：リソースのプロパティ。リソースの種類に基づいて異なるプロパティを変更することができます。現在、StarRocksは次のリソースのHiveメタストアのURIを変更することができます。
  - Apache Icebergリソースは、次のプロパティを変更することができます：
    - `iceberg.catalog-impl`：[カスタムカタログ](../../../data_source/External_table.md)の完全修飾クラス名。
    - `iceberg.catalog.hive.metastore.uris`：HiveメタストアのURI。
  - Apache Hive™リソースとApache Hudiリソースは、`hive.metastore.uris`を変更することができます。これはHiveメタストアのURIを示します。

## 使用上の注意

外部テーブルを作成するためにリソースを参照した後、このリソースのHiveメタストアのURIを変更すると、外部テーブルは使用できなくなります。データをクエリするために引き続き外部テーブルを使用する場合は、新しいメタストアに、元のメタストアと同じ名前とスキーマを持つテーブルが含まれていることを確認してください。

## 例

Hiveリソース `hive0` のHiveメタストアのURIを変更します。

```SQL
ALTER RESOURCE 'hive0' SET PROPERTIES ("hive.metastore.uris" = "thrift://10.10.44.91:9083")
```
