---
displayed_sidebar: Chinese
---

# KILL ANALYZE(キル アナライズ)

## 機能

実行中の統計情報収集タスクをキャンセルします。これには、手動での収集タスクとカスタムの自動収集タスクが含まれます。このステートメントはバージョン2.4からサポートされています。

## 文法

```SQL
KILL ANALYZE <ID>
```

手動での収集タスクのIDはSHOW ANALYZE STATUSで確認できます。カスタムの自動収集タスクのIDはSHOW ANALYZE JOBで確認できます。

## 関連文書

[SHOW ANALYZE STATUS](../data-definition/SHOW_ANALYZE_STATUS.md)

[SHOW ANALYZE JOB](../data-definition/SHOW_ANALYZE_JOB.md)
