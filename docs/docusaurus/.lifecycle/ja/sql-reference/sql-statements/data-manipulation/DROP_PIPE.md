---
displayed_sidebar: "English"
---

# パイプの削除

## 説明

パイプとそれに関連するジョブおよびメタデータを削除します。このステートメントをパイプに対して実行しても、このパイプを介してロードされたデータは取り消されません。

## 構文

```SQL
DROP PIPE [IF EXISTS] [db_name.]<pipe_name>
```

## パラメータ

### db_name

パイプが所属するデータベースの名前。

### pipe_name

パイプの名前。

## 例

データベース`mydatabase`内の`user_behavior_replica`という名前のパイプを削除する場合:

```SQL
USE mydatabase;
DROP PIPE user_behavior_replica;
```