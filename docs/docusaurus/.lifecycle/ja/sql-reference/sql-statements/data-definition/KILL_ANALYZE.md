---
displayed_sidebar: "Japanese"
---

# KILL ANALYZE

## 説明

手動およびカスタム自動タスクを含む**実行中**の収集タスクをキャンセルします。

このステートメントはv2.4からサポートされています。

## 構文

```SQL
KILL ANALYZE <ID>
```

手動収集タスクのタスクIDは、SHOW ANALYZE STATUSから取得できます。カスタム収集タスクのタスクIDは、SHOW ANALYZE SHOW ANALYZE JOBから取得できます。

## 参照

[SHOW ANALYZE STATUS](../data-definition/SHOW_ANALYZE_STATUS.md)

[SHOW ANALYZE JOB](../data-definition/SHOW_ANALYZE_JOB.md)

CBOの統計情報の収集に関する詳細は、[CBOのための統計情報の収集](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。