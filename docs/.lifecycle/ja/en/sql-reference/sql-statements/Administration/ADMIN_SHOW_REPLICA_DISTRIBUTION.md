---
displayed_sidebar: English
---

# ADMIN SHOW REPLICA DISTRIBUTION

## 説明

このステートメントは、テーブルまたはパーティションのレプリカの分布状態を表示するために使用されます。

:::tip

この操作にはSYSTEMレベルのOPERATE権限が必要です。[GRANT](../account-management/GRANT.md)の指示に従って、この権限を付与することができます。

:::

## 構文

```sql
ADMIN SHOW REPLICA DISTRIBUTION FROM [db_name.]tbl_name [PARTITION (p1, ...)]
```

注記:

結果におけるGraph列は、レプリカの分布比率をグラフィカルに表示します。

## 例

1. テーブルのレプリカ分布を確認する

    ```sql
    ADMIN SHOW REPLICA DISTRIBUTION FROM tbl1;
    ```

2. テーブル内のパーティションのレプリカ分布を確認する

    ```sql
    ADMIN SHOW REPLICA DISTRIBUTION FROM db1.tbl1 PARTITION(p1, p2);
    ```
