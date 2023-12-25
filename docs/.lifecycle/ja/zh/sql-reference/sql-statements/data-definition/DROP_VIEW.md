---
displayed_sidebar: Chinese
---

# DROP VIEW

## 機能

このステートメントは、論理ビューを削除するために使用されます。

## 文法

```sql
DROP VIEW [IF EXISTS] [db_name.]view_name
```

## 例

1. もし存在するなら、example_db 上の論理ビュー example_view を削除します。

    ```sql
    DROP VIEW IF EXISTS example_db.example_view;
    ```

## 関連操作

論理ビューを作成する場合は、[CREATE VIEW](../data-definition/CREATE_VIEW.md)を参照してください。
