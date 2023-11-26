---
displayed_sidebar: "Japanese"
---

# DROP VIEW（ビューの削除）

## 説明

この文は、論理ビューVIEWを削除するために使用されます。

## 構文

```sql
DROP VIEW [IF EXISTS]
[db_name.]view_name
```

## 例

1. もし存在する場合、example_dbのexample_viewビューを削除します。

    ```sql
    DROP VIEW IF EXISTS example_db.example_view;
    ```
