---
displayed_sidebar: "Japanese"
---

# CREATE TABLE LIKE（同様のテーブルの作成）

## 説明

別のテーブルの定義に基づいて、同一の空のテーブルを作成します。定義には、カラムの定義、パーティション、およびテーブルのプロパティが含まれます。

## 構文

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] [データベース.]テーブル名 LIKE [データベース.]テーブル名
```

> **注意**

1. 元のテーブルに対する `SELECT` 権限が必要です。
2. MySQL など、外部テーブルをコピーすることができます。

## 例

1. test1 データベース内で、table1 と同じテーブル構造の空のテーブル table2 を作成します。

    ```sql
    CREATE TABLE test1.table2 LIKE test1.table1
    ```

2. test2 データベース内で、test1.table1 と同じテーブル構造の空のテーブル table2 を作成します。

    ```sql
    CREATE TABLE test2.table2 LIKE test1.table1
    ```

3. test1 データベース内で、MySQL の外部テーブルと同じテーブル構造の空のテーブル table2 を作成します。

    ```sql
    CREATE TABLE test1.table2 LIKE test1.table1
    ```