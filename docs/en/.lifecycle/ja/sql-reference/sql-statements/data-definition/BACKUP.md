---
displayed_sidebar: "Japanese"
---

# バックアップ

## 説明

指定されたデータベース、テーブル、またはパーティションのデータをバックアップします。現在、StarRocksはOLAPテーブルのデータのバックアップのみをサポートしています。詳細については、[データのバックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

バックアップは非同期操作です。[SHOW BACKUP](../data-manipulation/SHOW_BACKUP.md)を使用してバックアップジョブのステータスを確認したり、[CANCEL BACKUP](../data-definition/CANCEL_BACKUP.md)を使用してバックアップジョブをキャンセルしたりすることができます。スナップショット情報は[SHOW SNAPSHOT](../data-manipulation/SHOW_SNAPSHOT.md)を使用して表示することができます。

> **注意**
>
> - 管理者権限を持つユーザーのみがデータをバックアップできます。
> - 各データベースでは、実行中のバックアップまたはリストアジョブは1つだけ許可されます。それ以外の場合、StarRocksはエラーを返します。
> - StarRocksはデータバックアップのためのデータ圧縮アルゴリズムの指定をサポートしていません。

## 構文

```SQL
BACKUP SNAPSHOT <db_name>.<snapshot_name>
TO <repository_name>
[ ON ( <table_name> [ PARTITION ( <partition_name> [, ...] ) ]
       [, ...] ) ]
[ PROPERTIES ("key"="value" [, ...] ) ]
```

## パラメータ

| **パラメータ**   | **説明**                                                     |
| --------------- | ------------------------------------------------------------ |
| db_name         | バックアップするデータを格納するデータベースの名前。             |
| snapshot_name   | データスナップショットの名前を指定します。グローバルに一意です。     |
| repository_name | リポジトリ名。[CREATE REPOSITORY](../data-definition/CREATE_REPOSITORY.md)を使用してリポジトリを作成できます。 |
| ON              | バックアップするテーブルの名前。このパラメータが指定されていない場合、データベース全体がバックアップされます。 |
| PARTITION       | バックアップするパーティションの名前。このパラメータが指定されていない場合、テーブル全体がバックアップされます。 |
| PROPERTIES      | データスナップショットのプロパティ。有効なキー:`type`: バックアップタイプ。現在、完全バックアップ `FULL` のみがサポートされています。デフォルト: `FULL`。`timeout`: タスクのタイムアウト。単位: 秒。デフォルト: `86400`。 |

## 例

例1: データベース `example_db` をリポジトリ `example_repo` にバックアップします。

```SQL
BACKUP SNAPSHOT example_db.snapshot_label1
TO example_repo
PROPERTIES ("type" = "full");
```

例2: `example_db` のテーブル `example_tbl` を `example_repo` にバックアップします。

```SQL
BACKUP SNAPSHOT example_db.snapshot_label2
TO example_repo
ON (example_tbl);
```

例3: `example_db` のテーブル `example_tbl` のパーティション `p1` と `p2`、およびテーブル `example_tbl2` を `example_repo` にバックアップします。

```SQL
BACKUP SNAPSHOT example_db.snapshot_label3
TO example_repo
ON(
    example_tbl PARTITION (p1, p2),
    example_tbl2
);
```
