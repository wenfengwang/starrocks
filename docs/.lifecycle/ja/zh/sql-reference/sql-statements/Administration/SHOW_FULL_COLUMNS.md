---
displayed_sidebar: Chinese
---

# SHOW FULL COLUMNS

## 機能

このステートメントは、指定されたテーブルの列情報を表示するために使用されます。

:::tip

この操作には権限が必要ありません。

:::

## 文法

```sql
SHOW FULL COLUMNS FROM <tbl_name>
```

## 例

1. 指定されたテーブルの列情報を表示します。

    ```sql
    SHOW FULL COLUMNS FROM tbl;
    ```
