---
displayed_sidebar: English
---

# SUSPEND または RESUME PIPE

## 説明

パイプを中断または再開します:

- ロードジョブが進行中（つまり、`RUNNING` 状態）の場合、パイプを `SUSPEND` すると、ジョブが中断されます。
- ロードジョブでエラーが発生した場合、パイプを `RESUME` すると、エラーのあるジョブの実行が再開されます。

## 構文

```SQL
ALTER PIPE [ IF EXISTS ] <pipe_name> { SUSPEND | RESUME [ IF SUSPENDED ] }
```

## パラメータ

### pipe_name

パイプの名前です。

## 例

### パイプを中断する

`mydatabase` というデータベース内の `user_behavior_replica` という名前のパイプ（`RUNNING` 状態）を中断します:

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica SUSPEND;
```

[SHOW PIPES](../../../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md) を使用してパイプを照会すると、その状態が `SUSPEND` に変化したことがわかります。

### パイプを再開する

`mydatabase` というデータベースで `user_behavior_replica` という名前のパイプを再開します:

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica RESUME;
```

[SHOW PIPES](../../../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md) を使用してパイプを照会すると、その状態が `RUNNING` に変化したことがわかります。
