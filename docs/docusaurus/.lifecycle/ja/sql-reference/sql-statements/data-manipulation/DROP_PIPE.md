---
displayed_sidebar: "Japanese"
---

# ドロップ パイプ

## 説明

パイプと関連するジョブおよびメタデータを削除します。パイプに対してこのステートメントを実行しても、このパイプを介してロードされたデータは取り消されません。

## 構文

```SQL
DROP PIPE [IF EXISTS] [db_name.]<pipe_name>
```

## パラメーター

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