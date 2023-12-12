---
displayed_sidebar: "Japanese"
---

# 解析ジョブを表示

## 説明

カスタムコレクションタスクの情報と状態を表示します。

デフォルトで、StarRocksはテーブルの完全な統計を自動的に収集します。データの更新を5分ごとに確認します。データの変更が検出されると、データ収集が自動的にトリガーされます。自動的な完全なコレクションを使用したくない場合は、FE構成項目 `enable_collect_full_statistic` を `false` に設定し、カスタムのコレクションタスクを作成できます。

このステートメントはv2.4からサポートされています。

## 構文

```SQL
SHOW ANALYZE JOB [WHERE]
```

WHERE句を使用して結果をフィルタリングできます。このステートメントは次の列を返します。

| **列名**       | **説明**                                    |
| -------------- | -------------------------------------------- |
| Id             | コレクションタスクのID。                    |
| Database       | データベース名。                             |
| Table          | テーブル名。                                 |
| Columns        | 列名。                                       |
| Type           | 統計の種類（ `FULL` と `SAMPLE` を含む）。    |
| Schedule       | スケジューリングの種類。自動タスクの場合は `SCHEDULE`。 |
| Properties     | カスタムパラメータ。                         |
| Status         | タスクの状態（PENDING、RUNNING、SUCCESS、FAILED を含む）。 |
| LastWorkTime   | 最後の収集の時刻。                         |
| Reason         | タスクが失敗した理由。タスクの実行が成功した場合はNULLが返されます。 |

## 例

```SQL
-- すべてのカスタムコレクションタスクを表示します。
SHOW ANALYZE JOB

-- データベース `test` のカスタムコレクションタスクを表示します。
SHOW ANALYZE JOB where `database` = 'test';
```

## 参照

[CREATE ANALYZE](../data-definition/CREATE_ANALYZE.md): 自動的なコレクションタスクをカスタマイズします。

[DROP ANALYZE](../data-definition/DROP_ANALYZE.md): カスタムのコレクションタスクを削除します。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md): 実行中のカスタムのコレクションタスクをキャンセルします。

CBOの統計情報の収集に関する詳細は、[CBOの統計情報の収集](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。