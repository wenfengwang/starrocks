---
displayed_sidebar: "Japanese"
---

# TRUNCATE TABLE（テーブルの切り捨て）

## 説明

この文は指定されたテーブルとパーティションのデータを切り捨てるために使用されます。

構文：

```sql
TRUNCATE TABLE [db.]tbl[ PARTITION(p1, p2, ...)]
```

注意：

1. この文はテーブルまたはパーティションを保持したままデータを切り捨てるために使用されます。
2. DELETEとは異なり、この文は指定されたテーブルまたはパーティションをまるごと空にすることしかできず、フィルタ条件を追加することはできません。
3. DELETEとは異なり、この方法でデータをクリアするとクエリのパフォーマンスに影響しません。
4. この文はデータを直接削除します。削除されたデータは回復できません。
5. この操作を行うテーブルはNORMAL状態である必要があります。たとえば、SCHEMA CHANGE中のテーブル上でTRUNCATE TABLEを実行することはできません。

## 例

1. `example_db`の下の`tbl`テーブルを切り捨てます。

    ```sql
    TRUNCATE TABLE example_db.tbl;
    ```

2. `tbl`テーブル内の`p1`および`p2`のパーティションを切り捨てます。

    ```sql
    TRUNCATE TABLE tbl PARTITION(p1, p2);
    ```