---
displayed_sidebar: English
---

# RESTORE のキャンセル

## 説明

指定されたデータベースで進行中の RESTORE タスクをキャンセルします。詳細については、[データバックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

> **注意**
>
> COMMIT フェーズで RESTORE タスクがキャンセルされた場合、復元されたデータは破損し、アクセス不可能になります。この場合、RESTORE 操作を再び実行し、ジョブが完了するまで待つしかありません。

## 構文

```SQL
CANCEL RESTORE FROM <db_name>
```

## パラメータ

| **パラメータ** | **説明**                                              |
| ------------- | ------------------------------------------------------ |
| db_name       | RESTORE タスクに関連するデータベースの名前です。       |

## 例

例 1: `example_db` データベースの RESTORE タスクをキャンセルします。

```SQL
CANCEL RESTORE FROM example_db;
```
