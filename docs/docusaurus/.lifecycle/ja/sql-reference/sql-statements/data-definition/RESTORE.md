---
displayed_sidebar: "Japanese"
---

# RESTORE（リストア）

## 説明

指定されたデータベース、テーブル、またはパーティションにデータを復元します。現時点では、StarRocksはOLAPテーブルにのみデータを復元することができます。詳細については、[データのバックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

RESTOREは非同期操作です。[SHOW RESTORE](../data-manipulation/SHOW_RESTORE.md)を使用してRESTOREのジョブのステータスを確認したり、[CANCEL RESTORE](../data-definition/CANCEL_RESTORE.md)を使用してRESTOREのジョブをキャンセルすることができます。

> **注意**
>
> - ADMIN権限を持つユーザーのみがデータを復元できます。
> - 各データベースでは、実行中のBACKUPまたはRESTOREジョブは1つだけ許可されています。そうでない場合、StarRocksはエラーを返します。

## 構文

```SQL
RESTORE SNAPSHOT <db_name>.<snapshot_name>
FROM <repository_name>
[ ON ( <table_name> [ PARTITION ( <partition_name> [, ...] ) ]
    [ AS <table_alias>] [, ...] ) ]
PROPERTIES ("key"="value", ...)
```

## パラメーター

| **パラメーター** | **説明**                                                     |
| --------------- | ------------------------------------------------------------ |
| db_name         | データを復元するデータベースの名前。                            |
| snapshot_name   | データスナップショットの名前。                                 |
| repository_name | リポジトリの名前。                                            |
| ON              | 復元するテーブルの名前。このパラメーターが指定されていない場合、データベース全体が復元されます。 |
| PARTITION       | 復元するパーティションの名前。このパラメーターが指定されていない場合、テーブル全体が復元されます。パーティション名は[SHOW PARTITIONS](../data-manipulation/SHOW_PARTITIONS.md)を使用して表示できます。 |
| PROPERTIES      | RESTORE操作のプロパティ。有効なキー：<ul><li>`backup_timestamp`：バックアップのタイムスタンプ。**必須**。[SHOW SNAPSHOT](../data-manipulation/SHOW_SNAPSHOT.md)を使用してバックアップタイムスタンプを表示できます。</li><li>`replication_num`：復元するレプリカの数を指定します。デフォルト：`3`。</li><li>`meta_version`：このパラメーターは、StarRocksの以前のバージョンでバックアップされたデータを復元するための一時的な解決策としてのみ使用されます。最新バージョンのバックアップデータにはすでに`meta version`が含まれており、これを指定する必要はありません。</li><li>`timeout`：タスクのタイムアウト。単位：秒。デフォルト：`86400`。</li></ul> |

## 例

例1：`example_repo`リポジトリからデータベース`example_db`に含まれるスナップショット`snapshot_label1`のテーブル`backup_tbl`をバックアップタイムスタンプ`2018-05-04-16-45-08`で1つのレプリカを復元します。

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

例2：`example_repo`から`snapshot_label2`のテーブル`backup_tbl`のパーティション`p1`および`p2`、および`backup_tbl2`テーブルをデータベース`example_db`に、`new_tbl`に名前変更してバックアップタイムスタンプ`2018-05-04-17-11-01`でデフォルトで3つのレプリカを復元します。

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