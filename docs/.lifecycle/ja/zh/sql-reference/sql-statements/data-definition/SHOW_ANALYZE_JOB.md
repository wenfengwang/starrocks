---
displayed_sidebar: Chinese
---

# ANALYZE JOB の表示

## 機能

カスタム自動収集タスクの情報と状態を表示します。カスタム自動収集タスクは、CBO 統計情報の収集に使用されます。このステートメントはバージョン 2.4 からサポートされています。

デフォルトでは、StarRocks は定期的にテーブルの全統計情報を自動収集します。デフォルトの更新チェック時間は 5 分ごとで、データの更新が検出された場合、自動的に収集がトリガーされます。自動全量収集を使用したくない場合は、FE の設定項目 `enable_collect_full_statistic` を `false` に設定することで、システムは自動全量収集を停止し、作成したカスタムタスクに基づいてカスタマイズされた収集を行います。

## 文法

```SQL
SHOW ANALYZE JOB [WHERE predicate]
```

WHERE 句を使用してフィルタ条件を設定し、結果をフィルタリングすることができます。このステートメントは以下の列を返します。

| **列名**     | **説明**                                                     |
| ------------ | ------------------------------------------------------------ |
| Id           | 収集タスクの ID。                                             |
| Database     | データベース名。                                             |
| Table        | テーブル名。                                                 |
| Columns      | 列名のリスト。                                               |
| Type         | 統計情報のタイプ。取りうる値: FULL、SAMPLE。                 |
| Schedule     | スケジュールのタイプ。自動収集タスクは `SCHEDULE` に固定されています。 |
| Properties   | カスタムパラメータ情報。                                     |
| Status       | タスクの状態。PENDING（保留中）、RUNNING（実行中）、SUCCESS（成功）、FAILED（失敗）を含みます。 |
| LastWorkTime | 最後の収集時間。                                             |
| Reason       | タスク失敗の原因。成功した場合は `NULL`。                    |

## 例

```SQL
-- クラスターのすべてのカスタム収集タスクを表示します。
SHOW ANALYZE JOB

-- データベース `test` のカスタム収集タスクを表示します。
SHOW ANALYZE JOB where `database` = 'test';
```

## 関連ドキュメント

[CREATE ANALYZE](../data-definition/CREATE_ANALYZE.md)：カスタム自動収集タスクを作成します。

[DROP ANALYZE](../data-definition/DROP_ANALYZE.md)：自動収集タスクを削除します。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md)：実行中（Running）の統計情報収集タスクをキャンセルします。

CBO 統計情報収集の詳細については、[CBO 統計情報](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。
