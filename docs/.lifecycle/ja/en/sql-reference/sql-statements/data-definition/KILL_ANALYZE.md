---
displayed_sidebar: English
---

# KILL ANALYZE

## 説明

実行中の**収集タスク**をキャンセルします。これには手動タスクとカスタム自動タスクが含まれます。

このステートメントはv2.4からサポートされています。

## 構文

```SQL
KILL ANALYZE <ID>
```

手動収集タスクのタスクIDは、SHOW ANALYZE STATUSから取得できます。カスタム収集タスクのタスクIDは、SHOW ANALYZE JOBから取得できます。

## 参照

[SHOW ANALYZE STATUS](../data-definition/SHOW_ANALYZE_STATUS.md)

[SHOW ANALYZE JOB](../data-definition/SHOW_ANALYZE_JOB.md)

CBOの統計情報を収集する詳細については、[CBOのための統計情報の収集](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。
