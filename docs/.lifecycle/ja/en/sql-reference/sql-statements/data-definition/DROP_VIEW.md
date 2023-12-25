---
displayed_sidebar: English
---

# DROP VIEW

## 説明

このステートメントは、論理ビューVIEWを削除するために使用されます。

## 構文

```sql
DROP VIEW [IF EXISTS]
[db_name.]view_name
```

## 例

1. もし存在するなら、example_dbのビューexample_viewを削除します。

    ```sql
    DROP VIEW IF EXISTS example_db.example_view;
    ```
