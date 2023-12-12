---
displayed_sidebar: "Japanese"
---

# ADMIN SHOW REPLICA DISTRIBUTION

## 説明

この文は、テーブルまたはパーティションレプリカの分布状況を表示するために使用されます。

構文：

```sql
ADMIN SHOW REPLICA DISTRIBUTION FROM [db_name.]tbl_name [PARTITION (p1, ...)]
```

注：

結果のGraph列には、レプリカの分布比率がグラフで表示されます。

## 例

1. テーブルのレプリカ分布を表示する

    ```sql
    ADMIN SHOW REPLICA DISTRIBUTION FROM tbl1;
    ```

2. テーブル内のパーティションのレプリカ分布を表示する

    ```sql
    ADMIN SHOW REPLICA DISTRIBUTION FROM db1.tbl1 PARTITION(p1, p2);
    ```