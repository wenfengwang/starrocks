---
displayed_sidebar: English
---

# バックアップのキャンセル

## 説明

指定したデータベースで進行中のBACKUPタスクをキャンセルします。詳細については、[データバックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

## 構文

```SQL
CANCEL BACKUP FROM <db_name>
```

## パラメーター

| **パラメーター** | **説明**                                       |
| ------------- | ----------------------------------------------------- |
| db_name       | BACKUPタスクが属するデータベースの名前。 |

## 例

例1: `example_db`データベースのBACKUPタスクをキャンセルします。

```SQL
CANCEL BACKUP FROM example_db;
```
