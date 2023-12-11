---
displayed_sidebar: "Japanese"
---

# 解析ステータスの表示

## 説明

コレクションタスクのステータスを表示します。

このステートメントは、カスタムコレクションタスクのステータスを表示するために使用することはできません。 カスタムコレクションタスクのステータスを表示するには、SHOW ANALYZE JOB を使用してください。

このステートメントは、v2.4 からサポートされています。

## 構文

```SQL
SHOW ANALYZE STATUS [WHERE]
```

`LILEまた`は`WHERE`を使用して、返す情報をフィルタリングできます。

このステートメントは、次の列を返します。

| **リスト名** | **説明**                                                     |
| ------------- | ------------------------------------------------------------ |
| Id            | コレクションタスクのID。                                     |
| Database      | データベース名。                                              |
| Table         | テーブル名。                                                  |
| Columns       | カラム名。                                                    |
| Type          | 統計の種類（FULL、SAMPLE、HISTOGRAMなど）。                       |
| Schedule      | スケジュールの種類。`ONCE`は手動、`SCHEDULE`は自動です。          |
| Status        | タスクのステータス。                                           |
| StartTime     | タスクの開始実行時間。                                          |
| EndTime       | タスクの終了実行時間。                                          |
| Properties    | カスタムパラメータ。                                            |
| Reason        | タスクが失敗した理由。実行に成功した場合はNULLが返されます。      |

## 参照

[ANALYZE TABLE](../data-definition/ANALYZE_TABLE.md): 手動コレクションタスクを作成します。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md): 実行中のカスタムコレクションタスクをキャンセルします。

CBOの統計情報の収集に関する詳細情報は、[CBO用統計情報の収集](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。