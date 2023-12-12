---
displayed_sidebar: "Japanese"
---

# 動的パーティションテーブルを表示

## 説明

このステートメントは、データベース内で動的パーティションプロパティが構成されたすべてのパーティションテーブルのステータスを表示するために使用されます。

## 構文

```sql
SHOW DYNAMIC PARTITION TABLES FROM <db_name>
```

このステートメントは、以下のフィールドを返します：

- TableName：テーブルの名前。
- Enable：動的パーティションが有効かどうか。
- TimeUnit：パーティションの時間粒度。
- Start：動的パーティションの開始オフセット。
- End：動的パーティションの終了オフセット。
- Prefix：パーティション名の接頭辞。
- Buckets：パーティションごとのバケツの数。
- ReplicationNum：テーブルのレプリカの数。
- StartOf
- LastUpdateTime：テーブルが最後に更新された時刻。
- LastSchedulerTime：テーブル内のデータが最後にスケジュールされた時刻。
- State：テーブルのステータス。
- LastCreatePartitionMsg：最新のパーティション作成操作のメッセージ。
- LastDropPartitionMsg：最新のパーティション削除操作のメッセージ。

## 例

`db_test`で動的パーティションプロパティが構成されたすべてのパーティションテーブルのステータスを表示します。

```sql
SHOW DYNAMIC PARTITION TABLES FROM db_test;
```