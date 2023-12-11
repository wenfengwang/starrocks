---
displayed_sidebar: "Japanese"
---

# SUM（合計）

## 説明

`expr`の非NULL値の合計を返します。DISTINCTキーワードを使用して、重複のない非NULL値の合計を計算できます。

## 構文

```Haskell
SUM([DISTINCT] expr)
```

## パラメーター

`expr`：数値の値に評価される式。TINYINT、SMALLINT、INT、FLOAT、DOUBLE、またはDECIMALのサポートされるデータ型です。

## 返り値

入力値と返り値の間のデータ型のマッピング：

- TINYINT -> BIGINT
- SMALLINT -> BIGINT
- INT -> BIGINT
- FLOAT -> DOUBLE
- DOUBLE -> DOUBLE
- DECIMAL -> DECIMAL

## 使用上の注意

- この関数はNULLを無視します。
- `expr`が存在しない場合、エラーが返されます。
- VARCHAR式が渡される場合、この関数は入力を暗黙的にDOUBLE値にキャストします。キャストに失敗した場合、エラーが返されます。

## 例

1. `employees`という名前のテーブルを作成します。

    ```SQL
    CREATE TABLE IF NOT EXISTS employees (
        region_num    TINYINT        COMMENT "範囲[-128, 127]",
        id            BIGINT         COMMENT "範囲[-2^63 + 1〜2^63 - 1]",
        hobby         STRING         NOT NULL COMMENT "上限値65533バイト",
        income        DOUBLE         COMMENT "8バイト",
        sales       DECIMAL(12,4)  COMMENT ""
        )
        DISTRIBUTED BY HASH(region_num);
    ```

2. `employees`にデータを挿入します。

    ```SQL
    INSERT INTO employees VALUES
    (3,432175,'3',25600,1250.23),
    (4,567832,'3',37932,2564.33),
    (3,777326,'2',null,1932.99),
    (5,342611,'6',43727,45235.1),
    (2,403882,'4',36789,52872.4);
    ```

3. `employees`からデータをクエリします。

    ```Plain Text
    MySQL > select * from employees;
    +------------+--------+-------+--------+------------+
    | region_num | id     | hobby | income | sales      |
    +------------+--------+-------+--------+------------+
    |          5 | 342611 | 6     |  43727 | 45235.1000 |
    |          2 | 403882 | 4     |  36789 | 52872.4000 |
    |          4 | 567832 | 3     |  37932 |  2564.3300 |
    |          3 | 432175 | 3     |  25600 |  1250.2300 |
    |          3 | 777326 | 2     |   NULL |  1932.9900 |
    +------------+--------+-------+--------+------------+
    5 rows in set (0.01 sec)
    ```

4. この関数を使用して合計を計算します。

    例1: 各地域の総売上を計算します。

    ```Plain Text
    MySQL > SELECT region_num, sum(sales) from employees
    group by region_num;

    +------------+------------+
    | region_num | sum(sales) |
    +------------+------------+
    |          2 | 52872.4000 |
    |          5 | 45235.1000 |
    |          4 |  2564.3300 |
    |          3 |  3183.2200 |
    +------------+------------+
    4 rows in set (0.01 sec)
    ```

    例2: 各地域の従業員の合計収入を計算します。この関数はNULLを無視し、従業員ID`777326`の収入は計上されません。

    ```Plain Text
    MySQL > select region_num, sum(income) from employees
    group by region_num;

    +------------+-------------+
    | region_num | sum(income) |
    +------------+-------------+
    |          2 |       36789 |
    |          5 |       43727 |
    |          4 |       37932 |
    |          3 |       25600 |
    +------------+-------------+
    4 rows in set (0.01 sec)
    ```

    例3: 趣味の合計数を計算します。`hobby`列はSTRING型であり、計算中に暗黙的にDOUBLEに変換されます。

    ```Plain Text
    MySQL > select sum(DISTINCT hobby) from employees;

    +---------------------+
    | sum(DISTINCT hobby) |
    +---------------------+
    |                  15 |
    +---------------------+
    1 row in set (0.01 sec)
    ```

    例4: WHERE句を使用して、月収が30000を超える従業員の合計収入を計算するために`sum`を使用します。

    ```Plain Text
    MySQL > select sum(income) from employees
    WHERE income > 30000;

    +-------------+
    | sum(income) |
    +-------------+
    |      118448 |
    +-------------+
    1 row in set (0.00 sec)
    ```

## キーワード

SUM, sum