---
displayed_sidebar: Chinese
---

# ALTER VIEW（ビューの変更）

## 機能

このステートメントは、論理ビューの定義を変更するために使用されます。

## 文法

```sql
ALTER VIEW
[db_name.]view_name
(column1[ COMMENT "col comment"][, column2, ...])
AS query_stmt
```

説明：

1. 論理ビューのデータは物理媒体には保存されず、クエリ時には論理ビューがステートメント内のサブクエリとして機能するため、論理ビューの定義を変更することは query_stmt の変更と同等です。
2. query_stmt はサポートされている任意の SQL です。

## 例

1. example_db 上の論理ビュー example_view を変更します。

    ```sql
    ALTER VIEW example_db.example_view
    (
        c1 COMMENT "column 1",
        c2 COMMENT "column 2",
        c3 COMMENT "column 3"
    )
    AS SELECT k1, k2, SUM(v1) 
    FROM example_table
    GROUP BY k1, k2;
    ```
