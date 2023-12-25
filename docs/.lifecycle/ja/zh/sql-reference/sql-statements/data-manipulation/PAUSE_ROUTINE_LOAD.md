---
displayed_sidebar: Chinese
---

# PAUSE ROUTINE LOAD

import RoutineLoadPrivNote from '../../../assets/commonMarkdown/RoutineLoadPrivNote.md'

## 機能

Routine Load インポートジョブを一時停止し、ジョブは PAUSED 状態になりますが、終了していません。[RESUME ROUTINE LOAD](./RESUME_ROUTINE_LOAD.md) ステートメントを実行してインポートジョブを再開することができます。

Routine Load インポートジョブを一時停止した後、[SHOW ROUTINE LOAD](./SHOW_ROUTINE_LOAD.md) や [ALTER ROUTINE LOAD](./ALTER_ROUTINE_LOAD.md) ステートメントを実行して、一時停止したインポートジョブの情報を表示および変更することができます。

<RoutineLoadPrivNote />

## 文法

```SQL
PAUSE ROUTINE LOAD FOR [db_name.]<job_name>
```

## パラメータ説明

| パラメータ名 | 必須 | 説明                                                         |
| ------------ | ---- | ------------------------------------------------------------ |
| db_name      |      | Routine Load インポートジョブが属するデータベース名。                   |
| job_name     | ✅    | Routine Load インポートジョブ名。|

## 例

`example_db` データベース内の `example_tbl1_ordertest1` という名前の Routine Load インポートジョブを一時停止します。

```SQL
PAUSE ROUTINE LOAD FOR example_db.example_tbl1_ordertest1;
```
