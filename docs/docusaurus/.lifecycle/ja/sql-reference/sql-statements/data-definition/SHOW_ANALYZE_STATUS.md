---
displayed_sidebar: "Japanese"
---

# ANALYZE STATUSの表示

## 説明

コレクションタスクの状態を表示します。

このステートメントは、カスタムコレクションタスクの状態を表示するためには使用できません。カスタムコレクションタスクの状態を表示するには、SHOW ANALYZE JOBを使用してください。

このステートメントはv2.4からサポートされています。

## 構文

```SQL
SHOW ANALYZE STATUS [WHERE]
```

`LILE`または`WHERE`を使用して情報をフィルタリングできます。

このステートメントは、次の列を返します。

| **リスト名** | **説明**                                                     |
| ------------ | ------------------------------------------------------------ |
| Id           | コレクションタスクのID。                                      |
| Database     | データベース名。                                             |
| Table        | テーブル名。                                                 |
| Columns      | カラム名。                                                  |
| Type         | 統計のタイプ（完全、サンプル、ヒストグラムなど）。        |
| Schedule     | スケジューリングのタイプ。`ONCE`は手動、`SCHEDULE`は自動。|
| Status       | タスクの状態。                                               |
| StartTime    | タスクの開始時刻。                                           |
| EndTime      | タスクの終了時刻。                                           |
| Properties   | カスタムパラメータ。                                         |
| Reason       | タスクが失敗した理由。実行が成功した場合はNULLが返されます。   |

## 参照

[ANALYZE TABLE](../data-definition/ANALYZE_TABLE.md): 手動コレクションタスクの作成。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md): 実行中のカスタムコレクションタスクをキャンセルします。

CBOの統計情報の収集に関する詳細は、「[CBO用統計情報の収集](../../../using_starrocks/Cost_based_optimizer.md)」を参照してください。