---
displayed_sidebar: "Japanese"
---

# テーブルステータスを表示

## 説明

この文は、テーブルの情報の一部を表示するために使用されます。

構文:

```sql
SHOW TABLE STATUS
[FROM db] [LIKE "pattern"]
```

> 注
>
> この文は主にMySQLの構文と互換性があります。現在は、コメントなどのわずかな情報しか表示しません。

## 例

1. 現在のデータベースにあるすべてのテーブルの情報を表示します。

    ```SQL
    SHOW TABLE STATUS;
    ```

2. 指定されたデータベース内で例という名前を含むテーブルの情報をすべて表示します。

    ```SQL
    SHOW TABLE STATUS FROM db LIKE "%example%";
    ```