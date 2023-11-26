---
displayed_sidebar: "Japanese"
---

# バックアップのキャンセル

## 説明

指定されたデータベースの進行中のバックアップタスクをキャンセルします。詳細については、[データのバックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

## 構文

```SQL
CANCEL BACKUP FROM <db_name>
```

## パラメータ

| **パラメータ** | **説明**                                       |
| ------------- | ----------------------------------------------------- |
| db_name       | バックアップタスクが所属するデータベースの名前。 |

## 例

例1: `example_db` データベースのバックアップタスクをキャンセルします。

```SQL
CANCEL BACKUP FROM example_db;
```
