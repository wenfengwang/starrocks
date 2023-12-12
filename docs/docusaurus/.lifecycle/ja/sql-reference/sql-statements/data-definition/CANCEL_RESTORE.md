---
displayed_sidebar: "Japanese"
---

# RESTORE キャンセル

## 説明

指定されたデータベース内の実行中の RESTORE タスクをキャンセルします。詳細は、[データのバックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

> **注意**
>
> COMMIT フェーズ中に RESTORE タスクをキャンセルすると、復元されたデータが破損し、アクセスできなくなります。この場合、RESTORE オペレーションを再実行し、ジョブの完了を待つしかありません。

## 構文

```SQL
CANCEL RESTORE FROM <db_name>
```

## パラメータ

| **パラメータ** | **説明**                        |
| ------------- | ------------------------------ |
| db_name       | RESTORE タスクが属するデータベースの名前。  |

## 例

例 1: `example_db` データベース内の RESTORE タスクをキャンセルします。

```SQL
CANCEL RESTORE FROM example_db;
```