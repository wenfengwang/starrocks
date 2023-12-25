---
displayed_sidebar: Chinese
---

# DROP ANALYZE

## 機能

カスタム自動収集タスクを削除します。カスタム自動収集タスクは、CBOの統計情報を収集するために使用されます。

デフォルトでは、StarRocksは定期的にテーブルの全統計情報を自動収集します。デフォルトのチェック更新時間は5分ごとで、データ更新があると自動的に収集がトリガーされます。自動全量収集を使用したくない場合は、FEの設定項目 `enable_collect_full_statistic` を `false` に設定することで、システムは自動全量収集を停止し、作成したカスタムタスクに基づいてカスタマイズされた収集を行います。

## 文法

```SQL
DROP ANALYZE <ID>
```

SHOW ANALYZE JOB を使用してタスクのIDを確認できます。

## 例

```SQL
DROP ANALYZE 266030;
```

## 関連文書

[CREATE ANALYZE](../data-definition/CREATE_ANALYZE.md)：カスタム自動収集タスクを作成します。

[SHOW ANALYZE JOB](../data-definition/SHOW_ANALYZE_JOB.md)：カスタム自動収集タスクを表示します。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md)：実行中（Running）の統計情報収集タスクをキャンセルします。

CBOの統計情報収集についてもっと知りたい場合は、[CBO統計情報](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。
