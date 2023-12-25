---
displayed_sidebar: Chinese
---

# ALTER RESOURCE

## 機能

リソースの属性を変更します。StarRocks 2.3 以降のバージョンでのみ、リソース属性の変更がサポートされています。

## 文法

```SQL
ALTER RESOURCE 'resource_name' SET PROPERTIES ("key"="value", ...)
```

## パラメータ説明

- `resource_name`：変更するリソースの名前。
- `PROPERTIES ("key"="value", ...)`：リソースの属性。異なるタイプのリソースは異なる属性の変更をサポートしており、現在は以下のリソースのHive metastoreアドレスを変更することがサポートされています。
  - Apache Icebergリソースは以下の属性の変更をサポートしています：
    - `iceberg.catalog-impl`：[custom catalog](../../../data_source/External_table.md#ステップ1-iceberg-リソースの作成) の完全修飾クラス名。
    - `iceberg.catalog.hive.metastore.uris`：Hive metastoreのアドレス。
  - Apache Hive™ および Apache Hudi リソースは `hive.metastore.uris`、つまりHive metastoreのアドレスの変更をサポートしています。

## 注意点

リソースを参照して外部テーブルを作成した後、そのリソースのHive metastoreアドレスを変更すると、その外部テーブルが使用不可になる可能性があります。引き続きその外部テーブルを使用してデータをクエリしたい場合は、新しいHive metastoreに、元のHive metastoreと同じ名前とテーブル構造を持つデータテーブルが存在することを確認する必要があります。

## 例

Apache Hive™ リソース `hive0` のHive metastoreアドレスを変更します。

```SQL
ALTER RESOURCE 'hive0' SET PROPERTIES ("hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083")
```
