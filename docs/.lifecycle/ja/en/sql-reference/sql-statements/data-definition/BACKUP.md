---
displayed_sidebar: English
---

# バックアップ

## 説明

指定されたデータベース、テーブル、またはパーティションのデータをバックアップします。現在、StarRocksはOLAPテーブルのデータバックアップのみをサポートしています。詳細については、[データバックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

BACKUPは非同期操作です。[SHOW BACKUP](../data-manipulation/SHOW_BACKUP.md)を使用してBACKUPジョブのステータスを確認するか、[CANCEL BACKUP](../data-definition/CANCEL_BACKUP.md)を使用してBACKUPジョブをキャンセルできます。スナップショット情報は[SHOW SNAPSHOT](../data-manipulation/SHOW_SNAPSHOT.md)を使用して表示できます。

> **注意**
>
> - ADMIN権限を持つユーザーのみがデータをバックアップできます。
> - 各データベースでは、同時に1つのBACKUPまたはRESTOREジョブのみが許可されます。それ以外の場合、StarRocksはエラーを返します。
> - StarRocksはデータバックアップのためのデータ圧縮アルゴリズムを指定することをサポートしていません。

## 構文

```SQL
BACKUP SNAPSHOT <db_name>.<snapshot_name>
TO <repository_name>
[ ON ( <table_name> [ PARTITION ( <partition_name> [, ...] ) ]
       [, ...] ) ]
[ PROPERTIES ("key"="value" [, ...] ) ]
```

## パラメーター

| **パラメーター**   | **説明**                                              |
| --------------- | ------------------------------------------------------------ |
| db_name         | バックアップするデータを格納しているデータベースの名前。   |
| snapshot_name   | データスナップショットに指定する名前。グローバルにユニークである必要があります。       |
| repository_name | リポジトリ名。[CREATE REPOSITORY](../data-definition/CREATE_REPOSITORY.md)を使用してリポジトリを作成できます。 |
| ON              | バックアップするテーブルの名前。このパラメータが指定されていない場合、データベース全体がバックアップされます。 |
| PARTITION       | バックアップするパーティションの名前。このパラメータが指定されていない場合、テーブル全体がバックアップされます。 |
| PROPERTIES      | データスナップショットのプロパティ。有効なキーは以下の通りです:`type`: バックアップタイプ。現在、フルバックアップ `FULL` のみがサポートされています。デフォルトは `FULL`。`timeout`: タスクのタイムアウト時間。単位は秒。デフォルトは `86400`秒です。 |

## 例

例 1: データベース `example_db` をリポジトリ `example_repo` にバックアップします。

```SQL
BACKUP SNAPSHOT example_db.snapshot_label1
TO example_repo
PROPERTIES ("type" = "full");
```

例 2: `example_db` のテーブル `example_tbl` をリポジトリ `example_repo` にバックアップします。

```SQL
BACKUP SNAPSHOT example_db.snapshot_label2
TO example_repo
ON (example_tbl);
```

例 3: `example_db` のテーブル `example_tbl` のパーティション `p1` と `p2` およびテーブル `example_tbl2` をリポジトリ `example_repo` にバックアップします。

```SQL
BACKUP SNAPSHOT example_db.snapshot_label3
TO example_repo
ON(
    example_tbl PARTITION (p1, p2),
    example_tbl2
);
```
