---
displayed_sidebar: English
---

# CREATE TABLE LIKE

## 説明

別のテーブルの定義に基づいて、同一の空のテーブルを作成します。この定義には、列の定義、パーティション、およびテーブルプロパティが含まれます。

## 構文

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] [database.]table_name LIKE [database.]table_name
```

> **注記**

1. 元のテーブルに対する`SELECT`権限が必要です。
2. MySQLのような外部テーブルをコピーすることができます。

## 例

1. test1データベースにおいて、table1と同じテーブル構造を持つ空のテーブルをtable2として作成します。

    ```sql
    CREATE TABLE test1.table2 LIKE test1.table1
    ```

2. test2データベースにおいて、test1.table1と同じテーブル構造を持つ空のテーブルをtable2として作成します。

    ```sql
    CREATE TABLE test2.table2 LIKE test1.table1
    ```

3. test1データベースにおいて、MySQLの外部テーブルと同じテーブル構造を持つ空のテーブルをtable2として作成します。

    ```sql
    CREATE TABLE test1.table2 LIKE test1.table1
    ```
