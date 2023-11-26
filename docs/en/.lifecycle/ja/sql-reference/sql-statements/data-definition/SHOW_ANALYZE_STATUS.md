---
displayed_sidebar: "Japanese"
---

# ANALYZE STATUSの表示

## 説明

コレクションタスクのステータスを表示します。

このステートメントは、カスタムコレクションタスクのステータスを表示するためには使用できません。カスタムコレクションタスクのステータスを表示するには、SHOW ANALYZE JOBを使用してください。

このステートメントは、v2.4からサポートされています。

## 構文

```SQL
SHOW ANALYZE STATUS [WHERE]
```

情報をフィルタリングするために`LILEまたはWHERE`を使用することができます。

このステートメントは、以下の列を返します。

| **リスト名** | **説明**                                                     |
| ------------ | ------------------------------------------------------------ |
| Id           | コレクションタスクのIDです。                                  |
| Database     | データベース名です。                                          |
| Table        | テーブル名です。                                              |
| Columns      | カラム名です。                                                |
| Type         | 統計のタイプです。FULL、SAMPLE、HISTOGRAMなどが含まれます。   |
| Schedule     | スケジューリングのタイプです。`ONCE`は手動、`SCHEDULE`は自動です。 |
| Status       | タスクのステータスです。                                      |
| StartTime    | タスクの実行を開始した時刻です。                              |
| EndTime      | タスクの実行が終了した時刻です。                              |
| Properties   | カスタムパラメータです。                                      |
| Reason       | タスクが失敗した理由です。実行が成功した場合はNULLが返されます。 |

## 参照

[ANALYZE TABLE](../data-definition/ANALYZE_TABLE.md): 手動のコレクションタスクを作成します。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md): 実行中のカスタムコレクションタスクをキャンセルします。

CBOの統計情報の収集に関する詳細は、[CBOのための統計情報の収集](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。
