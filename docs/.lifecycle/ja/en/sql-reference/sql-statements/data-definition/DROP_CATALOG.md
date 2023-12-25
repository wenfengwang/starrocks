---
displayed_sidebar: English
---

# DROP CATALOG

## 説明

外部カタログを削除します。内部カタログは削除できません。StarRocks クラスタには `default_catalog` という名前の内部カタログが1つだけ存在します。

## 構文

```SQL
DROP CATALOG <catalog_name>
```

## パラメーター

`catalog_name`: 外部カタログの名前です。

## 例

`hive1` という名前のHiveカタログを作成します。

```SQL
CREATE EXTERNAL CATALOG hive1
PROPERTIES(
  "type"="hive", 
  "hive.metastore.uris"="thrift://xx.xx.xx.xx:9083"
);
```

Hiveカタログを削除します。

```SQL
DROP CATALOG hive1;
```
