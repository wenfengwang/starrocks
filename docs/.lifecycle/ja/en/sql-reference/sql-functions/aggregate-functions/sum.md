---
displayed_sidebar: English
---

# sum

## 説明

`expr`のNULL以外の値の合計を返します。DISTINCTキーワードを使用して、異なる非NULL値の合計を計算できます。

## 構文

```Haskell
SUM([DISTINCT] expr)
```

## パラメーター

`expr`: 数値に評価される式。サポートされているデータ型は、TINYINT、SMALLINT、INT、FLOAT、DOUBLE、またはDECIMALです。

## 戻り値

入力値と戻り値の間のデータ型マッピング:

- TINYINT -> BIGINT
- SMALLINT -> BIGINT
- INT -> BIGINT
- FLOAT -> DOUBLE
- DOUBLE -> DOUBLE
- DECIMAL -> DECIMAL

## 使用上の注意

- この関数はnullを無視します。
- `expr`が存在しない場合はエラーが返されます。
- VARCHAR式が渡された場合、この関数は入力を暗黙的にDOUBLE値にキャストします。キャストが失敗すると、エラーが返されます。

## 例

1. `employees`という名前のテーブルを作成します。

    ```SQL
    CREATE TABLE IF NOT EXISTS employees (
        region_num    TINYINT        COMMENT "range [-128, 127]",
        id            BIGINT         COMMENT "range [-2^63 + 1 ~ 2^63 - 1]",
        hobby         STRING         NOT NULL COMMENT "upper limit value 65533 bytes",
        income        DOUBLE         COMMENT "8 bytes",
        sales         DECIMAL(12,4)  COMMENT ""
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

    例 1: 各地域の総売上を計算します。

    ```Plain Text
    MySQL > SELECT region_num, SUM(sales) FROM employees
    GROUP BY region_num;

    +------------+------------+
    | region_num | SUM(sales) |
    +------------+------------+
    |          2 | 52872.4000 |
    |          5 | 45235.1000 |
    |          4 |  2564.3300 |
    |          3 |  3183.2200 |
    +------------+------------+
    4 rows in set (0.01 sec)
    ```

    例 2: 各地域の総従業員所得を計算します。この関数はnullを無視し、従業員ID `777326` の収入はカウントされません。

    ```Plain Text
    MySQL > SELECT region_num, SUM(income) FROM employees
    GROUP BY region_num;

    +------------+-------------+
    | region_num | SUM(income) |
    +------------+-------------+
    |          2 |       36789 |
    |          5 |       43727 |
    |          4 |       37932 |
    |          3 |       25600 |
    +------------+-------------+
    4 rows in set (0.01 sec)
    ```

    例 3: 趣味の総数を計算します。`hobby`列はSTRING型であり、計算中に暗黙的にDOUBLEに変換されます。

    ```Plain Text
    MySQL > SELECT SUM(DISTINCT hobby) FROM employees;

    +---------------------+
    | SUM(DISTINCT hobby) |
    +---------------------+
    |                  15 |
    +---------------------+
    1 row in set (0.01 sec)
    ```

    例 4: `sum`をWHERE句と共に使用して、月収が30000を超える従業員の総収入を計算します。

    ```Plain Text
    MySQL > SELECT SUM(income) FROM employees
    WHERE income > 30000;

    +-------------+
    | SUM(income) |
    +-------------+
    |      118448 |
    +-------------+
    1 row in set (0.00 sec)
    ```

## キーワード

SUM, sum
