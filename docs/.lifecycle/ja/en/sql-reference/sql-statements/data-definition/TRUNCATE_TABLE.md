---
displayed_sidebar: English
---

# TRUNCATE TABLE

## 説明

このステートメントは、指定されたテーブルとパーティションデータを切り捨てるために使用されます。

構文：

```sql
TRUNCATE TABLE [db.]tbl[ PARTITION(p1, p2, ...)]
```

注意：

1. このステートメントは、テーブルまたはパーティションを保持したままデータを切り捨てるために使用されます。
2. DELETEとは異なり、このステートメントは指定されたテーブルまたはパーティションを全体として空にすることしかできず、フィルタリング条件を追加することはできません。
3. DELETEとは異なり、この方法でデータをクリアしてもクエリパフォーマンスに影響しません。
4. このステートメントはデータを直接削除します。削除されたデータは復元できません。
5. この操作を行うテーブルはNORMAL状態でなければなりません。たとえば、SCHEMA CHANGEが進行中のテーブルでTRUNCATE TABLEを実行することはできません。

## 例

1. `example_db`の`tbl`テーブルを切り捨てます。

    ```sql
    TRUNCATE TABLE example_db.tbl;
    ```

2. `tbl`テーブルの`p1`と`p2`パーティションを切り捨てます。

    ```sql
    TRUNCATE TABLE tbl PARTITION(p1, p2);
    ```
