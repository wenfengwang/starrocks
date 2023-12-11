---
displayed_sidebar: "Japanese"
---

# 動的パーティションテーブルの表示

## 説明

このステートメントは、データベースに構成された動的パーティションプロパティを持つすべてのパーティションテーブルの状態を表示するために使用されます。

## 構文

```sql
SHOW DYNAMIC PARTITION TABLES FROM <db_name>
```

このステートメントは、次のフィールドを返します。

- TableName: テーブルの名前。
- Enable: 動的パーティションが有効かどうか。
- TimeUnit: パーティションの時間単位。
- Start: 動的パーティションの開始オフセット。
- End: 動的パーティションの終了オフセット。
- Prefix: パーティション名の接頭辞。
- Buckets: パーティションごとのバケツの数。
- ReplicationNum: テーブルのレプリカの数。
- StartOf
- LastUpdateTime: テーブルが最後に更新された時間。
- LastSchedulerTime: 表内のデータが最後にスケジュールされた時間。
- State: テーブルの状態。
- LastCreatePartitionMsg: 最新のパーティション作成操作のメッセージ。
- LastDropPartitionMsg: 最新のパーティション削除操作のメッセージ。

## 例

`db_test`で構成された動的パーティションプロパティのあるすべてのパーティションテーブルの状態を表示します。

```sql
SHOW DYNAMIC PARTITION TABLES FROM db_test;
```