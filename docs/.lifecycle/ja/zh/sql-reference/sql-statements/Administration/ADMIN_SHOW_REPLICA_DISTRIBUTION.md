---
displayed_sidebar: Chinese
---

# ADMIN SHOW REPLICA DISTRIBUTION

## 機能

このステートメントは、テーブルまたはパーティションのレプリカ分布状態を表示するために使用されます。

:::tip

この操作にはSYSTEMレベルのOPERATE権限が必要です。ユーザーに権限を付与するには、[GRANT](../account-management/GRANT.md) を参照してください。

:::

## 文法

```sql
ADMIN SHOW REPLICA DISTRIBUTION FROM [db_name.]tbl_name [PARTITION (p1, ...)]
```

説明：

結果の Graph 列はグラフィカルな形式でレプリカの分布比率を表示します。

## 例

1. テーブルのレプリカ分布を確認する

    ```sql
    ADMIN SHOW REPLICA DISTRIBUTION FROM tbl1;
    ```

2. テーブルのパーティションのレプリカ分布を確認する

    ```sql
    ADMIN SHOW REPLICA DISTRIBUTION FROM db1.tbl1 PARTITION(p1, p2);
    ```
