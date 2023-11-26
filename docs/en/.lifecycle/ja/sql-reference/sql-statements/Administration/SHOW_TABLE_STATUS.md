---
displayed_sidebar: "Japanese"
---

# SHOW TABLE STATUS

## 説明

この文は、テーブルの一部の情報を表示するために使用されます。

構文:

```sql
SHOW TABLE STATUS
[FROM db] [LIKE "pattern"]
```

> 注意
>
> この文は主にMySQLの構文と互換性があります。現在はコメントなどの一部の情報のみを表示します。

## 例

1. 現在のデータベースのテーブルのすべての情報を表示します。

    ```SQL
    SHOW TABLE STATUS;
    ```

2. 指定されたデータベースに属し、名前に"example"を含むテーブルのすべての情報を表示します。

    ```SQL
    SHOW TABLE STATUS FROM db LIKE "%example%";
    ```
