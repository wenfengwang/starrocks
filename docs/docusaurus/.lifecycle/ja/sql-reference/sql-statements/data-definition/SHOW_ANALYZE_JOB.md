---
displayed_sidebar: "Japanese"
---

# SHOW ANALYZE JOB

## 説明

カスタムコレクションタスクの情報とステータスを表示します。

デフォルトではStarRocksはテーブルの完全な統計情報を自動的に収集します。データの更新を5分ごとに確認します。データの変更が検出された場合、データ収集が自動的にトリガーされます。自動的な完全な収集を使用したくない場合は、FE構成項目`enable_collect_full_statistic`を`false`に設定し、カスタムコレクションタスクをカスタマイズすることができます。

このステートメントはv2.4からサポートされています。

## 構文

```SQL
SHOW ANALYZE JOB [WHERE]
```

WHERE句を使用して結果をフィルタリングすることができます。このステートメントは、以下の列を返します。

| **列**       | **説明**                                                     |
| ------------ | ------------------------------------------------------------ |
| Id           | コレクションタスクのID。                                     |
| Database     | データベース名。                                             |
| Table        | テーブル名。                                                 |
| Columns      | 列名。                                                       |
| Type         | 統計情報のタイプ。「FULL」と「SAMPLE」を含む。               |
| Schedule     | スケジューリングのタイプ。「SCHEDULE」は自動タスクのタイプです。 |
| Properties   | カスタムパラメータ。                                         |
| Status       | タスクのステータス。「PENDING」、「RUNNING」、「SUCCESS」、「FAILED」を含む。 |
| LastWorkTime | 最後の収集の時間。                                          |
| Reason       | タスクが失敗した理由。タスクの実行が成功した場合はNULLが返されます。 |

## 例

```SQL
-- すべてのカスタムコレクションタスクを表示します。
SHOW ANALYZE JOB

-- データベース`test`のカスタムコレクションタスクを表示します。
SHOW ANALYZE JOB where `database` = 'test';
```

## 参照

[CREATE ANALYZE](../data-definition/CREATE_ANALYZE.md): 自動的なコレクションタスクをカスタマイズします。

[DROP ANALYZE](../data-definition/DROP_ANALYZE.md): カスタムコレクションタスクを削除します。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md): 実行中のカスタムコレクションタスクをキャンセルします。

CBOの統計情報の収集に関する詳細情報については、[CBOの統計情報の収集](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。