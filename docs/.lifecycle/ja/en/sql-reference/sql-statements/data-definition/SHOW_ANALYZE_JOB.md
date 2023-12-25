---
displayed_sidebar: English
---

# ANALYZE JOB の表示

## 説明

カスタム収集タスクの情報とステータスを表示します。

デフォルトでは、StarRocksはテーブルの完全な統計を自動的に収集します。5分ごとにデータの更新をチェックし、データ変更が検出された場合は、データ収集が自動的にトリガーされます。自動完全収集を使用しない場合は、FE構成項目`enable_collect_full_statistic`を`false`に設定して、収集タスクをカスタマイズできます。

このステートメントはv2.4からサポートされています。

## 構文

```SQL
SHOW ANALYZE JOB [WHERE]
```

WHERE句を使用して結果をフィルタリングできます。このステートメントは、以下の列を返します。

| **カラム**   | **説明**                                              |
| ------------ | ------------------------------------------------------------ |
| Id           | 収集タスクのID。                               |
| Database     | データベース名。                                           |
| Table        | テーブル名。                                              |
| Columns      | 列名。                                            |
| Type         | 統計のタイプ。`FULL`と`SAMPLE`を含みます。       |
| Schedule     | スケジューリングのタイプ。自動タスクの場合は`SCHEDULE`です。 |
| Properties   | カスタムパラメータ。                                           |
| Status       | タスクのステータス。PENDING、RUNNING、SUCCESS、FAILEDを含みます。 |
| LastWorkTime | 最後の収集時刻。                             |
| Reason       | タスクが失敗した理由。タスクの実行が成功した場合はNULLが返されます。 |

## 例

```SQL
-- すべてのカスタム収集タスクを表示します。
SHOW ANALYZE JOB

-- データベース`test`のカスタム収集タスクを表示します。
SHOW ANALYZE JOB WHERE `Database` = 'test';
```

## 参照

[CREATE ANALYZE](../data-definition/CREATE_ANALYZE.md): 自動収集タスクをカスタマイズします。

[DROP ANALYZE](../data-definition/DROP_ANALYZE.md): カスタム収集タスクを削除します。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md): 実行中のカスタム収集タスクをキャンセルします。

CBOの統計収集についての詳細は、[CBOのための統計収集](../../../using_starrocks/Cost_based_optimizer.md)をご覧ください。
