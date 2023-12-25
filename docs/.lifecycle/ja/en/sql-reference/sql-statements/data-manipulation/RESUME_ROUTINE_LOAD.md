---
displayed_sidebar: English
---

# RESUME ROUTINE LOAD

import RoutineLoadPrivNote from '../../../assets/commonMarkdown/RoutineLoadPrivNote.md'

## 説明

ルーチンロードジョブを再開します。ジョブは再スケジュールされるため、一時的に**NEED_SCHEDULE**状態に入ります。その後、ジョブは**RUNNING**状態に戻り、データソースからのメッセージの消費とデータのロードを続けます。ジョブの情報は、[SHOW ROUTINE LOAD](./SHOW_ROUTINE_LOAD.md) ステートメントを使用して確認できます。

<RoutineLoadPrivNote />

## 構文

```SQL
RESUME ROUTINE LOAD FOR [db_name.]<job_name>
```

## パラメーター

| **パラメーター** | **必須** | **説明**                                              |
| ------------- | ------------ | ------------------------------------------------------------ |
| db_name       |              | ルーチンロードジョブが属するデータベースの名前。 |
| job_name      | ✅            | ルーチンロードジョブの名前。                            |

## 例

データベース`example_db`でルーチンロードジョブ`example_tbl1_ordertest1`を再開します。

```SQL
RESUME ROUTINE LOAD FOR example_db.example_tbl1_ordertest1;
```
