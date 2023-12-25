---
displayed_sidebar: English
---

# DROP PIPE

## 説明

パイプおよび関連するジョブとメタデータを削除します。このステートメントをパイプに対して実行しても、このパイプを介してロードされたデータは取り消されません。

## 構文

```SQL
DROP PIPE [IF EXISTS] [db_name.]<pipe_name>
```

## パラメーター

### db_name

パイプが属するデータベースの名前です。

### pipe_name

パイプの名前です。

## 例

データベース `mydatabase` にある `user_behavior_replica` という名前のパイプを削除する:

```SQL
USE mydatabase;
DROP PIPE user_behavior_replica;
```
