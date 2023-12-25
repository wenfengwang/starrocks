---
displayed_sidebar: Chinese
---

# CREATE TABLE LIKE

## 機能

このステートメントは、他のテーブルと完全に同じ構造を持つ空のテーブルを作成するために使用されます。

## 文法

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] [database.]table_name LIKE [database.]table_name
```

説明:

1. コピーされるテーブル構造には、Column Definition、Partitions、Table Properties などが含まれます。
2. ユーザーはコピー元のテーブルに対する `SELECT` 権限が必要です。権限管理については [GRANT](../account-management/GRANT.md) の章を参照してください。
3. MySQL などの外部テーブルのコピーがサポートされています。

## 例

1. test1 データベースに、table1 と同じ構造の空のテーブルを table2 という名前で作成します。

    ```sql
    CREATE TABLE test1.table2 LIKE test1.table1;
    ```

2. test2 データベースに、test1.table1 と同じ構造の空のテーブルを table2 という名前で作成します。

    ```sql
    CREATE TABLE test2.table2 LIKE test1.table1;
    ```

3. test1 データベースに、MySQL の外部テーブル table1 と同じ構造の空のテーブルを table2 という名前で作成します。

    ```sql
    CREATE TABLE test1.table2 LIKE test1.table1;
    ```
