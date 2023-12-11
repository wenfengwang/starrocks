```yaml
---
displayed_sidebar: "Japanese"
---

# RESTORE キャンセル

## 説明

指定されたデータベース内の進行中の RESTORE タスクをキャンセルします。詳細については、[データのバックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

> **注意**
>
> RESTORE タスクが COMMIT フェーズ中にキャンセルされると、復元されたデータは破損し、アクセスできなくなります。この場合、RESTORE 操作を再実行し、ジョブの完了を待つことしかできません。

## 構文

```SQL
CANCEL RESTORE FROM <db_name>
```

## パラメータ

| **パラメータ** | **説明**                                                     |
| ------------- | ------------------------------------------------------------- |
| db_name       | RESTORE タスクが属するデータベースの名前。                   |

## 例

例 1: データベース `example_db` の RESTORE タスクをキャンセルします。

```SQL
CANCEL RESTORE FROM example_db;
```