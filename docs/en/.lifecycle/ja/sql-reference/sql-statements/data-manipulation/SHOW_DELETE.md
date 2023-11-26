---
displayed_sidebar: "Japanese"
---

# SHOW DELETE（削除の表示）

## 説明

このステートメントは、現在のデータベースで正常に実行された重複キーのテーブルに対する過去のDELETEタスクを表示するために使用されます。データの削除に関する詳細については、[DELETE](DELETE.md)を参照してください。

## 構文

```sql
SHOW DELETE [FROM <db_name>]
```

`db_name`: データベース名、オプションです。このパラメータが指定されていない場合、デフォルトで現在のデータベースが使用されます。

返されるフィールド:

- TableName: データが削除されたテーブル。
- PartitionName: データが削除されたパーティション。テーブルがパーティション化されていない場合、`*`が表示されます。
- CreateTime: DELETEタスクが作成された時刻。
- DeleteCondition: 指定されたDELETE条件。
- State: DELETEタスクの状態。

## 例

`database`のすべての過去のDELETEタスクを表示します。

```sql
SHOW DELETE FROM database;

+------------+---------------+---------------------+-----------------+----------+
| TableName  | PartitionName | CreateTime          | DeleteCondition | State    |
+------------+---------------+---------------------+-----------------+----------+
| mail_merge | *             | 2023-03-14 10:39:03 | name EQ "Peter" | FINISHED |
+------------+---------------+---------------------+-----------------+----------+
```
