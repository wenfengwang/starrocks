```SQL
ALTER PIPE

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

## パラメーター

### db_name

パイプが所属するデータベースの名前。

### pipe_name

パイプの名前。

### PROPERTIES

パイプの設定を変更したいプロパティ。フォーマット: `"key" = "value"`。サポートされているプロパティの詳細については、[CREATE PIPE](../../../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md) を参照してください。

## 例

データベース `mydatabase` にある `user_behavior_replica` という名前のパイプの `AUTO_INGEST` プロパティの設定を `FALSE` に変更する例：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica
SET
(
    "AUTO_INGEST" = "FALSE"
);
```