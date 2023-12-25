---
displayed_sidebar: Chinese
---

# SHOW TABLE STATUS

## 機能

このステートメントは、テーブルの情報を表示するために使用されます。

:::tip

この操作には権限が必要ありません。

:::

## 文法

```sql
SHOW TABLE STATUS
[FROM db] [LIKE "pattern"]
```

> **説明**
>
> このステートメントは、MySQLの文法との互換性を持たせるために主に使用され、現在はCommentなどの情報のみを表示します。

## 例

1. 現在のデータベースにあるすべてのテーブルの情報を表示

    ```SQL
    SHOW TABLE STATUS;
    ```

2. 指定されたデータベースで、名前に example を含むテーブルの情報を表示

    ```SQL
    SHOW TABLE STATUS FROM db LIKE "%example%";
    ```
