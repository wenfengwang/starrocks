```yaml
---
displayed_sidebar: "Japanese"
---

# パイプの中断または再開

## 説明

パイプを中断または再開します：

- ロードジョブが進行中の場合（つまり、「RUNNING」の状態である場合）、ジョブのためにパイプを中断（「SUSPEND」）すると、ジョブが中断されます。
- ロードジョブでエラーが発生した場合は、そのジョブのためにパイプを再開（「RESUME」）すると、エラーを起こしたジョブが続行されます。

## 構文

```SQL
ALTER PIPE [ IF EXISTS ] <pipe_name> { SUSPEND | RESUME [ IF SUSPENDED ] }
```

## パラメータ

### pipe_name

パイプの名前。

## 例

### パイプを中断する

データベース「mydatabase」で（「RUNNING」の状態にある）名前が「user_behavior_replica」のパイプを中断します：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica SUSPEND;
```

[SHOW PIPES](../../../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md)を使用してパイプをクエリすると、その状態が「SUSPEND」に変更されていることがわかります。

### パイプを再開する

データベース「mydatabase」で名前が「user_behavior_replica」のパイプを再開します：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica RESUME;
```

[SHOW PIPES](../../../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md)を使用してパイプをクエリすると、その状態が「RUNNING」に変更されていることがわかります。
```