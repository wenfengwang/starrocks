---
displayed_sidebar: "Japanese"
---

# ドロップパイプ

## 説明

パイプと関連するジョブとメタデータを削除します。パイプに対してこのステートメントを実行しても、このパイプを介してロードされたデータは取り消されません。

## 構文

```SQL
DROP PIPE [IF EXISTS] [db_name.]<pipe_name>
```

## パラメータ

### db_name

パイプが所属するデータベースの名前です。

### pipe_name

パイプの名前です。

## 例

データベース`mydatabase`内の名前が`user_behavior_replica`のパイプを削除します。

```SQL
USE mydatabase;
DROP PIPE user_behavior_replica;
```
