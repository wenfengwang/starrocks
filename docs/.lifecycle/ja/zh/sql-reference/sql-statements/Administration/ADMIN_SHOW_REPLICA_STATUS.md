---
displayed_sidebar: Chinese
---

# ADMIN SHOW REPLICA STATUS

## 機能

このステートメントは、テーブルまたはパーティションのレプリカの状態情報を表示するために使用されます。

:::tip

この操作にはSYSTEMレベルのOPERATE権限が必要です。ユーザーに権限を付与するには[GRANT](../account-management/GRANT.md)を参照してください。

:::

## 文法

```sql
ADMIN SHOW REPLICA STATUS FROM [db_name.]tbl_name [PARTITION (p1, ...)]
[WHERE STATUS [!]= "replica_status"]
```

説明：

```plain text
replica_status:
OK:             レプリカが正常な状態です
DEAD:           レプリカが存在するBackendが利用不可です
VERSION_ERROR:  レプリカのデータバージョンに欠落があります
SCHEMA_ERROR:   レプリカのschema hashが正しくありません
MISSING:        レプリカが存在しません
```

## 例

1. テーブルの全レプリカの状態を確認します。

    ```sql
    ADMIN SHOW REPLICA STATUS FROM db1.tbl1;
    ```

2. 特定のパーティションのVERSION_ERROR状態のレプリカを確認します。

    ```sql
    ADMIN SHOW REPLICA STATUS FROM tbl1 PARTITION (p1, p2)
    WHERE STATUS = "VERSION_ERROR";
    ```

3. 健康でない状態の全レプリカを確認します。

    ```sql
    ADMIN SHOW REPLICA STATUS FROM tbl1
    WHERE STATUS != "OK";
    ```
