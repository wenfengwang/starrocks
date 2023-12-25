---
displayed_sidebar: English
---

# ALTER PIPE

## 説明

パイプのプロパティ設定を変更します。

## 構文

```SQL
ALTER PIPE [db_name.]<pipe_name> 
SET PROPERTY
(
    "<key>" = <value>[, "<key>" = "<value>" ...]
) 
```

## パラメーター

### db_name

パイプが属するデータベースの名前です。

### pipe_name

パイプの名前です。

### PROPERTIES

パイプのプロパティ設定を変更したい場合。フォーマット: `"key" = "value"`。サポートされるプロパティの詳細については、[CREATE PIPE](../../../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md)を参照してください。

## 例

`mydatabase` というデータベースにある `user_behavior_replica` という名前のパイプの `AUTO_INGEST` プロパティの設定を `FALSE` に変更します：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica
SET
(
    "AUTO_INGEST" = "FALSE"
);
```
