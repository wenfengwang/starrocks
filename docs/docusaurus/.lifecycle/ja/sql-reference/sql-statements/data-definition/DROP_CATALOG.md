---
displayed_sidebar: "Japanese"
---

# カタログの削除

## 説明

外部カタログを削除します。内部カタログは削除できません。StarRocksクラスタには、`default_catalog`という名前の内部カタログが1つだけあります。

## 構文

```SQL
DROP CATALOG <catalog_name>
```

## パラメータ

`<catalog_name>`: 外部カタログの名前。

## 例

`hive1`という名前のHiveカタログを作成します。

```SQL
CREATE EXTERNAL CATALOG hive1
PROPERTIES(
  "type"="hive", 
  "hive.metastore.uris"="thrift://x.x.x.x:9083"
);
```

Hiveカタログを削除します。

```SQL
DROP CATALOG hive1;
```