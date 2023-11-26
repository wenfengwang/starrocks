---
displayed_sidebar: "Japanese"
---

# ダイナミックパーティションテーブルの表示

## 説明

このステートメントは、データベースで動的パーティションプロパティが設定されているすべてのパーティションテーブルの状態を表示するために使用されます。

## 構文

```sql
SHOW DYNAMIC PARTITION TABLES FROM <db_name>
```

このステートメントは、次のフィールドを返します：

- TableName: テーブルの名前。
- Enable: ダイナミックパーティショニングが有効かどうか。
- TimeUnit: パーティションの時間の粒度。
- Start: ダイナミックパーティショニングの開始オフセット。
- End: ダイナミックパーティショニングの終了オフセット。
- Prefix: パーティション名のプレフィックス。
- Buckets: パーティションごとのバケット数。
- ReplicationNum: テーブルのレプリカ数。
- StartOf
- LastUpdateTime: テーブルが最後に更新された時刻。
- LastSchedulerTime: テーブルのデータが最後にスケジュールされた時刻。
- State: テーブルの状態。
- LastCreatePartitionMsg: 最新のパーティション作成操作のメッセージ。
- LastDropPartitionMsg: 最新のパーティション削除操作のメッセージ。

## 例

`db_test` で動的パーティションプロパティが設定されているすべてのパーティションテーブルの状態を表示する。

```sql
SHOW DYNAMIC PARTITION TABLES FROM db_test;
```
