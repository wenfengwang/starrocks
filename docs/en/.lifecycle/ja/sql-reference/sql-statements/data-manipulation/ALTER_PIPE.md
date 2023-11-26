---
displayed_sidebar: "Japanese"
---

# ALTER PIPE

## 説明

パイプのプロパティの設定を変更します。

## 構文

```SQL
ALTER PIPE [db_name.]<pipe_name> 
SET PROPERTY
(
    "<key>" = <value>[, "<key>" = "<value>" ...]
) 
```

## パラメータ

### db_name

パイプが所属するデータベースの名前です。

### pipe_name

パイプの名前です。

### PROPERTIES

パイプの設定を変更したいプロパティです。フォーマット: `"key" = "value"`。サポートされているプロパティの詳細については、[CREATE PIPE](../../../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md)を参照してください。

## 例

データベース名が`mydatabase`で、パイプ名が`user_behavior_replica`のパイプの`AUTO_INGEST`プロパティの設定を`FALSE`に変更します。

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica
SET
(
    "AUTO_INGEST" = "FALSE"
);
```
