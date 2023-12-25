---
displayed_sidebar: Chinese
---

# RESTORE

## 機能

指定されたデータベース、テーブル、またはパーティションのデータを復元します。現在のStarRocksはOLAPタイプのテーブルのみを復元することができます。詳細については、[バックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

データ復元は非同期操作です。[SHOW RESTORE](../data-manipulation/SHOW_RESTORE.md) ステートメントで復元ジョブの状態を確認するか、[CANCEL RESTORE](../data-definition/CANCEL_RESTORE.md) ステートメントで復元ジョブをキャンセルすることができます。

> **注意**
>
> 同一データベース内で、同時に一つのバックアップまたは復元ジョブのみを実行できます。それ以外の場合はシステムエラーが発生します。

## 権限要求

バージョン3.0以前では、admin_priv権限を持つユーザーのみがこの操作を実行できました。バージョン3.0以降では、データベース全体を復元するにはSystemレベルのREPOSITORY権限と、データベースの作成、テーブルの作成、データのインポート権限が必要です。特定のテーブルを復元する場合は、SystemレベルのREPOSITORY権限と、そのテーブルに対するインポート権限（INSERT）が必要です。例えば：

- 特定のテーブルのデータを復元する権限をロールに付与します。

    ```SQL
    GRANT REPOSITORY ON SYSTEM TO ROLE recover_tbl;
    GRANT INSERT ON TABLE <table_name> TO ROLE recover_tbl;
    ```

- 特定のデータベース内のすべてのデータを復元する権限をロールに付与します。

    ```SQL
    GRANT REPOSITORY ON SYSTEM TO ROLE recover_db;
    GRANT INSERT ON ALL TABLES IN ALL DATABASES TO ROLE recover_db;
    ```

- default_catalog内のすべてのデータベースのデータを復元する権限をロールに付与します。

    ```SQL
    GRANT REPOSITORY ON SYSTEM TO ROLE recover_db;
    GRANT CREATE DATABASE ON CATALOG default_catalog TO ROLE recover_db;
    ```

## 構文

```SQL
RESTORE SNAPSHOT <db_name>.<snapshot_name>
FROM <repository_name>
[ ON ( <table_name> [ PARTITION ( <partition_name> [, ...] ) ]
    [ AS <table_alias>] [, ...] ) ]
PROPERTIES ("key"="value", ...)
```

## パラメータ説明

| **パラメータ**  | **説明**                                                     |
| --------------- | ------------------------------------------------------------ |
| db_name         | このデータベースにデータを復元します。                         |
| snapshot_name   | データスナップショット名。                                   |
| repository_name | リポジトリ名。                                              |
| ON              | 復元するテーブル名。指定しない場合はデータベース全体を復元します。 |
| PARTITION       | 復元するパーティション名。指定しない場合は対応するテーブルのすべてのパーティションを復元します。パーティション名は [SHOW PARTITIONS](../data-manipulation/SHOW_PARTITIONS.md) ステートメントで確認できます。 |
| PROPERTIES      | 復元操作の属性。現在サポートされている属性は以下の通りです：<ul><li>`backup_timestamp`：バックアップのタイムスタンプ、**必須**。[SHOW SNAPSHOT](../data-manipulation/SHOW_SNAPSHOT.md) でバックアップのタイムスタンプを確認できます。</li><li>`replication_num`：復元するテーブルまたはパーティションのレプリカ数を指定します。デフォルトは `3`。</li><li>`meta_version`：このパラメータは一時的なソリューションとして使用され、旧バージョンのStarRocksのバックアップデータを復元するためにのみ使用されます。最新バージョンのバックアップデータには `meta version` が含まれているため、指定する必要はありません。</li><li>`timeout`：ジョブのタイムアウト時間。単位は秒。デフォルトは `86400`。</li></ul> |

## 例

例1：`example_repo` リポジトリから `snapshot_label1` のバックアップに含まれる `backup_tbl` テーブルを `example_db` データベースに復元します。バックアップのタイムスタンプは `2018-05-04-16-45-08`。レプリカは1つに復元します。

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

例2：`example_repo` リポジトリから `snapshot_label2` のバックアップに含まれる `backup_tbl` テーブルの `p1` と `p2` パーティション、および `backup_tbl2` テーブルを `example_db` データベースに復元し、`new_tbl` としてリネームします。バックアップのタイムスタンプは `2018-05-04-17-11-01`。デフォルトで3つのレプリカに復元します。

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
