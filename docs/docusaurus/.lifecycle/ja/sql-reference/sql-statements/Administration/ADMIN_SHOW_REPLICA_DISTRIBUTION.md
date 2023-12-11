---
displayed_sidebar: "Japanese"
---

# 管理者のレプリカ配布の表示

## 説明

この文は、テーブルまたはパーティションのレプリカの配布状況を表示するために使用されます。

構文：

```sql
ADMIN SHOW REPLICA DISTRIBUTION FROM [db_name.]tbl_name [PARTITION (p1, ...)]
```

注意：

結果のGraph列は、レプリカの配布比率をグラフで表示します。

## 例

1. テーブルのレプリカ配布を表示

    ```sql
    ADMIN SHOW REPLICA DISTRIBUTION FROM tbl1;
    ```

2. テーブル内のパーティションのレプリカ配布を表示

    ```sql
    ADMIN SHOW REPLICA DISTRIBUTION FROM db1.tbl1 PARTITION(p1, p2);
    ```