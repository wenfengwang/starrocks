---
displayed_sidebar: Chinese
---

# STOP ROUTINE LOAD

import RoutineLoadPrivNote from '../../../assets/commonMarkdown/RoutineLoadPrivNote.md'

## 機能

Routine Load インポートジョブを停止します。

<RoutineLoadPrivNote />

::: warning

インポートジョブは停止され、復旧することはできません。したがって、このコマンドは慎重に実行してください。

一時的にインポートジョブを停止するだけの場合は、[PAUSE ROUTINE LOAD](./PAUSE_ROUTINE_LOAD.md)を実行してください。

:::

## 文法

```SQL
STOP ROUTINE LOAD FOR [db_name.]<job_name>
```

## パラメータ説明

| パラメータ名 | 必須 | 説明                        |
| ------------ | ---- | --------------------------- |
| db_name      |      | Routine Load インポートジョブが属するデータベース名。 |
| job_name     | ✅    | Routine Load インポートジョブの名前。 |

## 例

`example_db` データベースにある `example_tbl1_ordertest1` という名前の Routine Load インポートジョブを停止します。

```SQL
STOP ROUTINE LOAD FOR example_db.example_tbl1_ordertest1;
```
