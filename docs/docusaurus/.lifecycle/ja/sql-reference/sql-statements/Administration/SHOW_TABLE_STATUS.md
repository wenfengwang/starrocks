---
displayed_sidebar: "Japanese"
---

# SHOW TABLE STATUS（テーブルの状態を表示）

## 説明

この文は、テーブルの一部の情報を表示するために使用されます。

構文：

```sql
SHOW TABLE STATUS
[FROM db] [LIKE "pattern"]
```

> 注意
>
> この文は主にMySQLの構文と互換性があります。現在はコメントなどのいくつかの情報しか表示しません。

## 例

1. 現在のデータベースにあるテーブルのすべての情報を表示します。

    ```SQL
    SHOW TABLE STATUS;
    ```

2. 指定されたデータベース内にあり、名前に「example」が含まれるテーブルのすべての情報を表示します。

    ```SQL
    SHOW TABLE STATUS FROM db LIKE "%example%";
    ```