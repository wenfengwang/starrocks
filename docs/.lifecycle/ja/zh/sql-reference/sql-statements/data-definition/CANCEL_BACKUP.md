---
displayed_sidebar: Chinese
---

# バックアップのキャンセル

## 機能

指定したデータベースの進行中のバックアップタスクをキャンセルします。詳細については、[バックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

## 文法

```SQL
CANCEL BACKUP FROM <db_name>
```

## パラメータ説明

| **パラメータ** | **説明**               |
| -------------- | ---------------------- |
| db_name        | キャンセルするバックアップタスクが属するデータベース名。 |

## 例

例1：`example_db` データベースのバックアップタスクをキャンセルします。

```SQL
CANCEL BACKUP FROM example_db;
```
