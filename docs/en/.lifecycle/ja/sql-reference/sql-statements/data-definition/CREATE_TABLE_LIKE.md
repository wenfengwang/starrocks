---
displayed_sidebar: "Japanese"
---

# CREATE TABLE LIKE

## 説明

他のテーブルの定義に基づいて、同じ構造の空のテーブルを作成します。定義には、列の定義、パーティション、およびテーブルのプロパティが含まれます。

## 構文

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] [database.]table_name LIKE [database.]table_name
```

> **注意**

1. 元のテーブルには `SELECT` 権限が必要です。
2. MySQL のような外部テーブルをコピーすることができます。

## 例

1. test1 データベースの table1 と同じテーブル構造の空のテーブル table2 を作成します。

    ```sql
    CREATE TABLE test1.table2 LIKE test1.table1
    ```

2. test2 データベースの test1.table1 と同じテーブル構造の空のテーブル table2 を作成します。

    ```sql
    CREATE TABLE test2.table2 LIKE test1.table1
    ```

3. test1 データベースの MySQL 外部テーブルと同じテーブル構造の空のテーブル table2 を作成します。

    ```sql
    CREATE TABLE test1.table2 LIKE test1.table1
    ```
