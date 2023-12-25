---
displayed_sidebar: Chinese
---

# DROP PIPE

## 機能

Pipeを削除し、関連するジョブとメタデータも削除します。この操作は、既にインポートされたデータを削除しません。

## 構文

```SQL
DROP PIPE [IF EXISTS] [db_name.]<pipe_name>
```

## パラメータ説明

### db_name

Pipeが属するデータベースの名前。

### pipe_name

Pipeの名前。

## 例

データベース `mydatabase` にある `user_behavior_replica` という名前のPipeを削除する：

```SQL
USE mydatabase;
DROP PIPE user_behavior_replica;
```
