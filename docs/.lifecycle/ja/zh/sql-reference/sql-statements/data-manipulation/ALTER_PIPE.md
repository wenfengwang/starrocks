---
displayed_sidebar: Chinese
---

# ALTER PIPE（パイプの変更）

## 機能

Pipe の実行パラメータを変更します。

## 構文

```SQL
ALTER PIPE [db_name.]<pipe_name> 
SET
(
    "<key>" = <value>[, "<key>" = "<value>" ...]
) 
```

## パラメータ説明

### db_name

Pipe が属するデータベースの名前。

### pipe_name

Pipe の名前。

### **PROPERTIES**

変更する実行パラメータの設定。形式：`"key" = "value"`。サポートされている実行パラメータについては、[CREATE PIPE](../../../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md) を参照してください。

## 例

データベース `mydatabase` にある `user_behavior_replica` という名前の Pipe の `AUTO_INGEST` 属性を `FALSE` に変更する：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica
SET
(
    "AUTO_INGEST" = "FALSE"
);
```
