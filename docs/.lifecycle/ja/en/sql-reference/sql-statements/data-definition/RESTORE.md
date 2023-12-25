---
displayed_sidebar: English
---

# RESTORE

## 説明

指定されたデータベース、テーブル、またはパーティションにデータを復元します。現在、StarRocksはOLAPテーブルへのデータ復元のみをサポートしています。詳細は[データバックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

RESTOREは非同期操作です。[SHOW RESTORE](../data-manipulation/SHOW_RESTORE.md)を使用してRESTOREジョブのステータスを確認するか、[CANCEL RESTORE](../data-definition/CANCEL_RESTORE.md)を使用してRESTOREジョブをキャンセルできます。

> **注意**
>
> - データを復元できるのはADMIN権限を持つユーザーのみです。
> - 各データベースでは、同時に1つのBACKUPまたはRESTOREジョブのみが許可されます。それ以外の場合、StarRocksはエラーを返します。

## 構文

```SQL
RESTORE SNAPSHOT <db_name>.<snapshot_name>
FROM <repository_name>
[ ON ( <table_name> [ PARTITION ( <partition_name> [, ...] ) ]
    [ AS <table_alias>] [, ...] ) ]
PROPERTIES ("key"="value", ...)
```

## パラメーター

| **パラメーター**   | **説明**                                              |
| --------------- | ------------------------------------------------------------ |
| db_name         | データが復元されるデータベースの名前です。           |
| snapshot_name   | データスナップショットの名前です。                                  |
| repository_name | リポジトリの名前です。                                             |
| ON              | 復元されるテーブルの名前です。このパラメーターが指定されていない場合、データベース全体が復元されます。 |
| PARTITION       | 復元されるパーティションの名前です。このパラメーターが指定されていない場合、テーブル全体が復元されます。パーティション名は[SHOW PARTITIONS](../data-manipulation/SHOW_PARTITIONS.md)を使用して確認できます。 |
| PROPERTIES      | RESTORE操作のプロパティです。有効なキーは以下の通りです：<ul><li>`backup_timestamp`：バックアップのタイムスタンプ。**必須**。バックアップタイムスタンプは[SHOW SNAPSHOT](../data-manipulation/SHOW_SNAPSHOT.md)を使用して確認できます。</li><li>`replication_num`：復元されるレプリカの数を指定します。デフォルトは`3`です。</li><li>`meta_version`：このパラメーターは、以前のバージョンのStarRocksによってバックアップされたデータを復元するための一時的な解決策としてのみ使用されます。バックアップされたデータの最新バージョンには既に`meta_version`が含まれているため、指定する必要はありません。</li><li>`timeout`：タスクのタイムアウト。単位は秒です。デフォルトは`86400`です。</li></ul> |

## 例

例 1: `example_repo`リポジトリから`snapshot_label1`スナップショット内のテーブル`backup_tbl`をデータベース`example_db`に復元します。バックアップタイムスタンプは`2018-05-04-16-45-08`です。1つのレプリカを復元します。

```SQL
RESTORE SNAPSHOT example_db.snapshot_label1
FROM example_repo
ON ( backup_tbl )
PROPERTIES
(
    "backup_timestamp"="2018-05-04-16-45-08",
    "replication_num" = "1"
);
```

例 2: `example_repo`から`snapshot_label2`スナップショット内のテーブル`backup_tbl`のパーティション`p1`と`p2`、およびテーブル`backup_tbl2`をデータベース`example_db`に復元し、`backup_tbl2`を`new_tbl`としてリネームします。バックアップタイムスタンプは`2018-05-04-17-11-01`です。デフォルトで3つのレプリカを復元します。

```SQL
RESTORE SNAPSHOT example_db.snapshot_label2
FROM example_repo
ON(
    backup_tbl PARTITION (p1, p2),
    backup_tbl2 AS new_tbl
)
PROPERTIES
(
    "backup_timestamp"="2018-05-04-17-11-01"
);
```
