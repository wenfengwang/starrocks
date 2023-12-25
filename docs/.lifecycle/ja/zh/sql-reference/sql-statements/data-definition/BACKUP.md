---
displayed_sidebar: Chinese
---

# バックアップ

## 機能

指定されたデータベース、テーブル、またはパーティションのデータをバックアップします。現在のStarRocksは、OLAPタイプのテーブルのバックアップのみをサポートしています。詳細については、[バックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

データバックアップは非同期操作です。[SHOW BACKUP](../data-manipulation/SHOW_BACKUP.md) ステートメントを使用してバックアップジョブの状態を確認するか、[CANCEL BACKUP](../data-definition/CANCEL_BACKUP.md) ステートメントを使用してバックアップジョブをキャンセルすることができます。ジョブが成功した後、[SHOW SNAPSHOT](../data-manipulation/SHOW_SNAPSHOT.md) を使用して特定のリポジトリに対応するデータスナップショット情報を確認できます。

> **注意**
>
> - 単一のデータベース内では、一度に一つのバックアップまたは復元ジョブのみを実行でき、それ以外の場合はシステムエラーが発生します。
> - 現在、StarRocksはデータをバックアップする際に圧縮アルゴリズムを使用することはサポートしていません。

## 権限要求

バージョン3.0以前では、admin_priv権限を持つユーザーのみがこの操作を実行できました。バージョン3.0以降では、特定のテーブルまたはデータベース全体をバックアップするには、SystemレベルのREPOSITORY権限と、対応するテーブルまたは対応するデータベース内のすべてのテーブルに対するEXPORT権限が必要です。例えば：

- 特定のテーブルからデータをエクスポートする権限をロールに付与します。

    ```SQL
    GRANT REPOSITORY ON SYSTEM TO ROLE backup_tbl;
    GRANT EXPORT ON TABLE <table_name> TO ROLE backup_tbl;
    ```

- 特定のデータベース内のすべてのテーブルからデータをエクスポートする権限をロールに付与します。

    ```SQL
    GRANT REPOSITORY ON SYSTEM TO ROLE backup_db;
    GRANT EXPORT ON ALL TABLES IN DATABASE <database_name> TO ROLE backup_db;
    ```

- すべてのデータベース内のすべてのテーブルからデータをエクスポートする権限をロールに付与します。

    ```SQL
    GRANT REPOSITORY ON SYSTEM TO ROLE backup;
    GRANT EXPORT ON ALL TABLES IN ALL DATABASES TO ROLE backup;
    ```

## 文法

```SQL
BACKUP SNAPSHOT <db_name>.<snapshot_name>
TO <repository_name>
[ ON ( <table_name> [ PARTITION ( <partition_name> [, ...] ) ]
       [, ...] ) ]
[ PROPERTIES ("key"="value" [, ...] ) ]
```

## パラメータ説明

| **パラメータ**  | **説明**                                                       |
| --------------- | -------------------------------------------------------------- |
| db_name         | バックアップするデータが属するデータベース名。                         |
| snapshot_name   | データスナップショット名を指定します。グローバルにおいて、スナップショット名は重複できません。 |
| repository_name | リポジトリ名。[CREATE REPOSITORY](../data-definition/CREATE_REPOSITORY.md) を使用してリポジトリを作成できます。 |
| ON              | バックアップするテーブル名。指定しない場合は、データベース全体をバックアップします。 |
| PARTITION       | バックアップするパーティション名。指定しない場合は、対応するテーブルのすべてのパーティションをバックアップします。 |
| PROPERTIES      | データスナップショットの属性。現在サポートされている属性は以下の通りです：`type`：バックアップのタイプ。現在は `FULL`、つまり全量バックアップのみをサポートしています。デフォルトは `FULL`。`timeout`：ジョブのタイムアウト時間。単位は秒。デフォルトは `86400`。 |

## 例

例1：`example_db` データベースを `example_repo` リポジトリに全量バックアップします。

```SQL
BACKUP SNAPSHOT example_db.snapshot_label1
TO example_repo
PROPERTIES ("type" = "full");
```

例2：`example_db` データベースの `example_tbl` テーブルを `example_repo` リポジトリに全量バックアップします。

```SQL
BACKUP SNAPSHOT example_db.snapshot_label2
TO example_repo
ON (example_tbl);
```

例3：`example_db` データベースの `example_tbl` テーブルの `p1`、`p2` パーティションと `example_tbl2` テーブルを `example_repo` リポジトリに全量バックアップします。

```SQL
BACKUP SNAPSHOT example_db.snapshot_label3
TO example_repo
ON(
    example_tbl PARTITION (p1, p2),
    example_tbl2
);
```
