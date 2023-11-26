---
displayed_sidebar: "Japanese"
---

# パイプの中断または再開

## 説明

パイプを中断または再開します：

- ロードジョブが進行中（つまり、`RUNNING`状態）の場合、ジョブの中断のためにパイプを中断（`SUSPEND`）します。
- ロードジョブがエラーに遭遇した場合、ジョブを再開（`RESUME`）すると、エラーが発生したジョブが継続して実行されます。

## 構文

```SQL
ALTER PIPE [ IF EXISTS ] <pipe_name> { SUSPEND | RESUME [ IF SUSPENDED ] }
```

## パラメータ

### pipe_name

パイプの名前。

## 例

### パイプの中断

データベース`mydatabase`の`RUNNING`状態にある`user_behavior_replica`という名前のパイプを中断します：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica SUSPEND;
```

[SHOW PIPES](../../../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md)を使用してパイプをクエリすると、その状態が`SUSPEND`に変わったことが確認できます。

### パイプの再開

データベース`mydatabase`の`user_behavior_replica`という名前のパイプを再開します：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica RESUME;
```

[SHOW PIPES](../../../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md)を使用してパイプをクエリすると、その状態が`RUNNING`に変わったことが確認できます。
