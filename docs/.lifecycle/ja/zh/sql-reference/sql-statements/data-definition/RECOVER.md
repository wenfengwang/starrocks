---
displayed_sidebar: Chinese
---

# リカバリ

## 機能

以前に削除されたデータベース、テーブル、またはパーティションを復元します。[TRUNCATE TABLE](./TRUNCATE_TABLE.md) コマンドで削除されたデータは復元できません。

> **注意**
>
> default_catalog の CREATE DATABASE 権限を持つユーザーのみがデータベースを復元できます。また、対応するデータベースの CREATE TABLE 権限と対応するテーブルの DROP 権限が必要です。

## 文法

### データベースを復元

```sql
RECOVER DATABASE <db_name>
```

### テーブルを復元

```sql
RECOVER TABLE [db_name.]table_name;
```

### パーティションを復元

```sql
RECOVER PARTITION partition_name FROM [db_name.]table_name;
```

説明：

1. この操作は、以前に削除されたメタデータのみを一定期間内に復元できます。デフォルトは1日です。（`catalog_trash_expire_second` パラメータを fe.conf で設定できます）

2. メタデータを削除した後に同じ名前とタイプの新しいメタデータを作成した場合、以前に削除されたメタデータは復元できません。

## 例

1. example_db という名前のデータベースを復元します。

    ```sql
    RECOVER DATABASE example_db;
    ```

2. example_tbl という名前のテーブルを復元します。

    ```sql
    RECOVER TABLE example_db.example_tbl;
    ```

3. example_tbl テーブル内の p1 という名前のパーティションを復元します。

    ```sql
    RECOVER PARTITION p1 FROM example_tbl;
    ```
