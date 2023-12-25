---
displayed_sidebar: Chinese
---

# SUSPEND または RESUME PIPE

## 機能

Pipeの一時停止または再開:

- インポートジョブが進行中の場合（つまり、`RUNNING` 状態にある場合）、`SUSPEND` でPipeを一時停止すると、実行中のジョブが中断されます。
- インポートエラーが発生した場合、`RESUME` でPipeを再開すると、エラーが発生したジョブの実行が続行されます。

## 文法

```SQL
ALTER PIPE [ IF EXISTS ] <pipe_name> { SUSPEND | RESUME [ IF SUSPENDED ] }
```

## パラメータ説明

### pipe_name

Pipeの名前。

## 例

### Pipeの一時停止

データベース `mydatabase` にある `user_behavior_replica` という名前のPipeを一時停止します（Pipeは現在 `RUNNING` 状態です）：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica SUSPEND;
```

[SHOW PIPES](../../../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md) を使用してそのPipeを確認すると、Pipeの状態が `SUSPEND` になっていることがわかります。

### Pipeの再開

データベース `mydatabase` にある `user_behavior_replica` という名前のPipeを再開します：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica RESUME;
```

[SHOW PIPES](../../../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md) を使用してそのPipeを確認すると、Pipeの状態が `RUNNING` になっていることがわかります。
