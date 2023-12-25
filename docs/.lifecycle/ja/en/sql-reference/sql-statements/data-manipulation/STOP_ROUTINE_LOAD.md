---
displayed_sidebar: English
---

# STOP ROUTINE LOAD

import RoutineLoadPrivNote from '../../../assets/commonMarkdown/RoutineLoadPrivNote.md'

## 説明

ルーチンロードジョブを停止します。

<RoutineLoadPrivNote />

::: warning

- 停止したルーチンロードジョブは再開できません。したがって、このステートメントを実行する際には慎重に進めてください。
- ルーチンロードジョブを一時停止するだけの場合は、[PAUSE ROUTINE LOAD](./PAUSE_ROUTINE_LOAD.md)を実行してください。

:::

## 構文

```SQL
STOP ROUTINE LOAD FOR [db_name.]<job_name>
```

## パラメータ

| **パラメータ** | **必須** | **説明**                                              |
| ------------- | ------------ | ------------------------------------------------------------ |
| db_name       |              | ルーチンロードジョブが属するデータベースの名前です。 |
| job_name      | ✅            | ルーチンロードジョブの名前です。                            |

## 例

データベース`example_db`でルーチンロードジョブ`example_tbl1_ordertest1`を停止します。

```SQL
STOP ROUTINE LOAD FOR example_db.example_tbl1_ordertest1;
```
