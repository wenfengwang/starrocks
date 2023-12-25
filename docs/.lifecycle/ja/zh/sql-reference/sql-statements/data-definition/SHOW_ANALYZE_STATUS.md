---
displayed_sidebar: Chinese
---

# ANALYZE STATUS の表示

## 機能

現在実行中のすべての統計情報収集タスクの状態を確認します。**この文はカスタム統計情報収集タスクの状態を確認することはできません。確認するには、SHOW ANALYZE JOB; を使用してください。**この文はバージョン 2.4 からサポートされています。

## 文法

```SQL
SHOW ANALYZE STATUS [LIKE | WHERE predicate]
```

`LIKE` または `WHERE` を使用して、返される情報をフィルタリングできます。

現在 `SHOW ANALYZE STATUS` は以下の列を返します。

| **列名**   | **説明**                                                     |
| ---------- | ------------------------------------------------------------ |
| Id         | 統計情報収集タスクの ID。                                     |
| Database   | データベース名。                                             |
| Table      | テーブル名。                                                 |
| Columns    | 列名のリスト。                                               |
| Type       | 統計情報のタイプ。FULL、SAMPLE、HISTOGRAM を含みます。       |
| Schedule   | スケジュールのタイプ。`ONCE` は手動を意味し、`SCHEDULE` は自動を意味します。 |
| Status     | タスクの状態。RUNNING（実行中）、SUCCESS（成功）、FAILED（失敗）を含みます。 |
| StartTime  | タスクが開始された時間。                                      |
| EndTime    | タスクが終了した時間。                                        |
| Properties | カスタムパラメータ情報。                                      |
| Reason     | タスクが失敗した理由。成功した場合は NULL。                  |

## 関連文書

[ANALYZE TABLE](../data-definition/ANALYZE_TABLE.md)：手動で統計情報収集タスクを作成します。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md)：実行中（Running）の統計情報収集タスクをキャンセルします。

CBO の統計情報収集についての詳細は、[CBO 統計情報](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。
