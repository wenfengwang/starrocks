---
displayed_sidebar: "Japanese"
---

# ADMIN SHOW REPLICA DISTRIBUTION（レプリカの分布を表示する）

## 説明

このステートメントは、テーブルまたはパーティションのレプリカの分布状況を表示するために使用されます。

構文:

```sql
ADMIN SHOW REPLICA DISTRIBUTION FROM [db_name.]tbl_name [PARTITION (p1, ...)]
```

注意:

結果のGraph列には、レプリカの分布比率がグラフィカルに表示されます。

## 例

1. テーブルのレプリカの分布を表示する

    ```sql
    ADMIN SHOW REPLICA DISTRIBUTION FROM tbl1;
    ```

2. テーブル内のパーティションのレプリカの分布を表示する

    ```sql
    ADMIN SHOW REPLICA DISTRIBUTION FROM db1.tbl1 PARTITION(p1, p2);
    ```
