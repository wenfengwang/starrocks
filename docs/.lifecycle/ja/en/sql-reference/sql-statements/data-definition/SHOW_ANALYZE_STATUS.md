---
displayed_sidebar: English
---

# SHOW ANALYZE STATUS

## 説明

収集タスクのステータスを表示します。

このステートメントは、カスタム収集タスクのステータスを表示するために使用することはできません。カスタム収集タスクのステータスを表示するには、SHOW ANALYZE JOBを使用します。

このステートメントは v2.4 からサポートされています。

## 構文

```SQL
SHOW ANALYZE STATUS [WHERE]
```

`LIKE or WHERE` を使用して、返す情報をフィルタリングできます。

このステートメントは、次の列を返します。

| **リスト名** | **説明**                                              |
| ------------- | ------------------------------------------------------------ |
| Id            | 収集タスクの ID。                               |
| Database      | データベース名。                                           |
| Table         | テーブル名。                                              |
| Columns       | 列名。                                            |
| Type          | 統計のタイプで、FULL、SAMPLE、HISTOGRAMを含みます。 |
| Schedule      | スケジューリングのタイプ。 `ONCE` は手動を意味し、 `SCHEDULE` は自動を意味します。 |
| Status        | タスクのステータス。                                      |
| StartTime     | タスクの実行が開始される時刻。                     |
| EndTime       | タスクの実行が終了する時刻。                       |
| Properties    | カスタムパラメータ。                                           |
| Reason        | タスクが失敗した理由。実行が成功した場合はNULLが返されます。 |

## 参照

[ANALYZE TABLE](../data-definition/ANALYZE_TABLE.md): 手動収集タスクを作成します。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md): 実行中のカスタム収集タスクをキャンセルします。

CBOの統計を収集する詳細については、[CBOのための統計収集](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。
