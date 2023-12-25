---
displayed_sidebar: Chinese
---

# DROP CATALOG（カタログの削除）

## 機能

指定された external catalog を削除します。現在、internal catalog の削除はサポートされていません。StarRocks クラスターには `default_catalog` という名前のデフォルトの internal catalog が1つだけ存在します。

## 文法

```SQL
DROP CATALOG catalog_name
```

## パラメータ説明

`catalog_name`：削除する external catalog の名前で、必須パラメータです。

## 例

`hive1` という名前の Hive catalog を作成します。

```SQL
CREATE EXTERNAL CATALOG hive1
PROPERTIES(
  "type"="hive", 
  "hive.metastore.uris"="thrift://xx.xx.xx.xx:9083"
);
```

`hive1` を削除します。

```SQL
DROP CATALOG hive1;
```
