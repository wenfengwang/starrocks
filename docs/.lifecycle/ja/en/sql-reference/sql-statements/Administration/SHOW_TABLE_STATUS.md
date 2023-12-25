---
displayed_sidebar: English
---

# SHOW TABLE STATUS

## 説明

このステートメントは、テーブル内の情報の一部を表示するために使用されます。

:::tip

この操作には権限は必要ありません。

:::

## 構文

```sql
SHOW TABLE STATUS
[FROM db] [LIKE "pattern"]
```

> 注意
>
> このステートメントは主にMySQL構文と互換性があります。現在、コメントなどのいくつかの情報のみを表示します。

## 例

1. 現在のデータベースにあるテーブルのすべての情報を表示します。

    ```SQL
    SHOW TABLE STATUS;
    ```

2. 指定されたデータベースにあって、名前に「example」を含むテーブルのすべての情報を表示します。

    ```SQL
    SHOW TABLE STATUS FROM db LIKE "%example%";
    ```
