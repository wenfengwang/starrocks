---
displayed_sidebar: English
---

# ADMIN SHOW REPLICA STATUS

## 説明

このステートメントは、テーブルまたはパーティションのレプリカの状態を表示するために使用されます。

:::tip

この操作にはSYSTEMレベルのOPERATE権限が必要です。[GRANT](../account-management/GRANT.md)の指示に従って、この権限を付与することができます。

:::

## 構文

```sql
ADMIN SHOW REPLICA STATUS FROM [db_name.]tbl_name [PARTITION (p1, ...)]
[where_clause]
```

```sql
where_clause:
WHERE STATUS [!]= "replica_status"
```

```plain text
replica_status:
OK:            レプリカは正常です
DEAD:          レプリカのBackendが利用できません
VERSION_ERROR: レプリカのデータバージョンがありません
SCHEMA_ERROR:  レプリカのスキーマハッシュが正しくありません
MISSING:       レプリカが存在しません
```

## 例

1. テーブルの全レプリカの状態を表示します。

    ```sql
    ADMIN SHOW REPLICA STATUS FROM db1.tbl1;
    ```

2. ステータスがVERSION_ERRORのパーティションのレプリカを表示します。

    ```sql
    ADMIN SHOW REPLICA STATUS FROM tbl1 PARTITION (p1, p2)
    WHERE STATUS = "VERSION_ERROR";
    ```

3. テーブルの正常でない全レプリカを表示します。

    ```sql
    ADMIN SHOW REPLICA STATUS FROM tbl1
    WHERE STATUS != "OK";
    ```
