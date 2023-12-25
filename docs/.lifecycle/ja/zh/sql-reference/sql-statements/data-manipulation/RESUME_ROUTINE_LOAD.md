---
displayed_sidebar: Chinese
---

# RESUME ROUTINE LOAD の再開

import RoutineLoadPrivNote from '../../../assets/commonMarkdown/RoutineLoadPrivNote.md'

## 機能

Routine Load インポートジョブを再開します。インポートジョブは一時的に **NEED_SCHEDULE** 状態に入り、インポートジョブの再スケジュールが行われていることを示します。しばらくすると **RUNNING** 状態に戻り、データソースからのメッセージの消費とデータのインポートを続けます。再開されたインポートジョブを確認するには、[SHOW ROUTINE LOAD](./SHOW_ROUTINE_LOAD.md) ステートメントを実行します。

<RoutineLoadPrivNote />

## 文法

```SQL
RESUME ROUTINE LOAD FOR [db_name.]<job_name>
```

## パラメータ説明

| パラメータ名 | 必須 | 説明                        |
| ------------ | ---- | --------------------------- |
| db_name      |      | Routine Load インポートジョブが属するデータベース名。 |
| job_name     | ✅   | Routine Load インポートジョブ名。       |

## 例

`example_db` データベースにある `example_tbl1_ordertest1` という名前の Routine Load インポートジョブを再開します。

```SQL
RESUME ROUTINE LOAD FOR example_db.example_tbl1_ordertest1;
```
