---
displayed_sidebar: English
---

# 動的パーティションテーブルの表示

## 説明

このステートメントは、データベース内で動的パーティションプロパティが設定された全てのパーティションテーブルの状態を表示するために使用されます。

## 構文

```sql
SHOW DYNAMIC PARTITION TABLES FROM <db_name>
```

このステートメントは以下のフィールドを返します:

- TableName: テーブルの名前です。
- Enable: 動的パーティションが有効かどうか。
- TimeUnit: パーティションの時間単位。
- Start: 動的パーティションの開始オフセット。
- End: 動的パーティションの終了オフセット。
- Prefix: パーティション名の接頭辞。
- Buckets: 各パーティションのバケット数。
- ReplicationNum: テーブルのレプリケーション数。
- LastUpdateTime: テーブルが最後に更新された時間。
- LastSchedulerTime: テーブルのデータが最後にスケジュールされた時間。
- State: テーブルの状態。
- LastCreatePartitionMsg: 最新のパーティション作成操作のメッセージ。
- LastDropPartitionMsg: 最新のパーティション削除操作のメッセージ。

## 例

`db_test` で動的パーティションプロパティが設定された全てのパーティションテーブルの状態を表示します。

```sql
SHOW DYNAMIC PARTITION TABLES FROM db_test;
```
