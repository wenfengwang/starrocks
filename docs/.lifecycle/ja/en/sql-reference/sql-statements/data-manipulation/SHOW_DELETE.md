---
displayed_sidebar: English
---

# SHOW DELETE

## 説明

このステートメントは、現在のデータベース内のDuplicate Keyテーブルに対して正常に実行された履歴DELETEタスクを表示するために使用されます。データ削除の詳細については、[DELETE](DELETE.md)を参照してください。

## 構文

```sql
SHOW DELETE [FROM <db_name>]
```

`db_name`: データベース名、オプション。このパラメータが指定されていない場合、デフォルトで現在のデータベースが使用されます。

戻り値のフィールド:

- TableName: データが削除されたテーブル。
- PartitionName: データが削除されたパーティション。テーブルが非パーティションテーブルの場合、`*`が表示されます。
- CreateTime: DELETEタスクが作成された時刻。
- DeleteCondition: 指定されたDELETE条件。
- State: DELETEタスクの状態。

## 例

`database`のすべての履歴DELETEタスクを表示します。

```sql
SHOW DELETE FROM database;

+------------+---------------+---------------------+-----------------+----------+
| TableName  | PartitionName | CreateTime          | DeleteCondition | State    |
+------------+---------------+---------------------+-----------------+----------+
| mail_merge | *             | 2023-03-14 10:39:03 | name EQ "Peter" | FINISHED |
+------------+---------------+---------------------+-----------------+----------+
```
