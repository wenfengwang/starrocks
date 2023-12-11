---
displayed_sidebar: "Japanese"
---

# インデックスの表示

## 説明

このステートメントはテーブル内のインデックスに関する情報を表示するために使用されます。現在、ビットマップインデックスのみをサポートしています。

構文：

```sql
SHOW INDEX[ES] FROM [db_name.]table_name [FROM database]
または
SHOW KEY[S] FROM [db_name.]table_name [FROM database]
```

## 例

1. 指定されたtable_nameのすべてのインデックスを表示する：

    ```sql
    SHOW INDEX FROM example_db.table_name;
    ```