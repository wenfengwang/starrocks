---
displayed_sidebar: Chinese
---

# SHOW DELETE の表示

## 機能

このステートメントは、現在のデータベースにおいて、詳細モデルテーブル（Duplicate Key）で成功した歴史的なDELETE操作を表示するために使用されます。

DELETE操作の詳細については、[DELETE](DELETE.md)を参照してください。

## 文法

```sql
SHOW DELETE [FROM <db_name>]
```

`db_name`: データベース名で、オプションです。指定されていない場合は、現在のデータベースがデフォルトになります。

コマンドが返すフィールド：

- TableName: テーブル名。どのテーブルからデータが削除されたかを示します。
- PartitionName: テーブルのパーティション名。どのパーティションからデータが削除されたかを示します。テーブルが**非パーティションテーブル**の場合は、`*`と表示されます。
- CreateTime: DELETE操作の作成時間。
- DeleteCondition: 指定された削除条件。
- State: DELETE操作の状態。

## 例

データベース `database` における全ての歴史的なDELETE操作を表示します。

```sql
SHOW DELETE FROM database;

+------------+---------------+---------------------+-----------------+----------+
| TableName  | PartitionName | CreateTime          | DeleteCondition | State    |
+------------+---------------+---------------------+-----------------+----------+
| mail_merge | *             | 2023-03-14 10:39:03 | name EQ "Peter" | FINISHED |
+------------+---------------+---------------------+-----------------+----------+
```
