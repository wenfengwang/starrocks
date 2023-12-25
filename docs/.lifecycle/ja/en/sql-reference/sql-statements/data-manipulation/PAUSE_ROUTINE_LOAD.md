---
displayed_sidebar: English
---

# PAUSE ROUTINE LOAD

import RoutineLoadPrivNote from '../../../assets/commonMarkdown/RoutineLoadPrivNote.md'

## 説明

ルーチンロードジョブを一時停止しますが、このジョブを終了するわけではありません。[RESUME ROUTINE LOAD](./RESUME_ROUTINE_LOAD.md)を実行して再開することができます。ロードジョブが一時停止された後、[SHOW ROUTINE LOAD](./SHOW_ROUTINE_LOAD.md)と[ALTER ROUTINE LOAD](./ALTER_ROUTINE_LOAD.md)を実行して、一時停止したロードジョブの情報を表示および変更することができます。

<RoutineLoadPrivNote />

## 構文

```SQL
PAUSE ROUTINE LOAD FOR [db_name.]<job_name>;
```

## パラメータ

| パラメータ | 必須 | 説明                                                  |
| --------- | ---- | ----------------------------------------------------- |
| db_name   |      | ルーチンロードジョブが属するデータベースの名前です。 |
| job_name  | ✅   | ルーチンロードジョブの名前です。テーブルには複数のルーチンロードジョブが存在する可能性があるため、Kafkaトピック名やロードジョブを作成した時刻などの識別可能な情報を用いて意味のあるルーチンロードジョブ名を設定することを推奨します。ルーチンロードジョブの名前は同一データベース内でユニークでなければなりません。 |

## 例

データベース`example_db`でルーチンロードジョブ`example_tbl1_ordertest1`を一時停止します。

```sql
PAUSE ROUTINE LOAD FOR example_db.example_tbl1_ordertest1;
```
