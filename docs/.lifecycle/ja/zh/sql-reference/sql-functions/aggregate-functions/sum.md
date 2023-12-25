---
displayed_sidebar: Chinese
---


# 合計

## 機能

指定された列の全ての値の合計を返します。この関数はNULL値を無視し、DISTINCT演算子と組み合わせて使用することができます。

## 文法

```Haskell
SUM(expr)
```

## パラメータ説明

`expr`: 合計計算に参加する列を指定します。列の値はTINYINT、SMALLINT、INT、FLOAT、DOUBLE、またはDECIMAL型であることができます。

## 戻り値の説明

列の値と戻り値の型のマッピングは以下の通りです：

- TINYINT -> BIGINT
- SMALLINT -> BIGINT
- INT -> BIGINT
- FLOAT -> DOUBLE
- DOUBLE -> DOUBLE
- DECIMAL -> DECIMAL

一致する列が見つからない場合は、エラーが返されます。
ある行の値がNULLの場合、その行は計算に含まれません。
列の値がSTRING型の数字である場合、計算時にDOUBLE型に暗黙的に変換されます。

## 例

1. `employees`テーブルを作成します。

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

2. テーブルにデータを挿入します。

    ```SQL
    INSERT INTO employees VALUES
    (3,432175,'3',25600,1250.23),
    (4,567832,'3',37932,2564.33),
    (3,777326,'2',null,1932.99),
    (5,342611,'6',43727,45235.1),
    (2,403882,'4',36789,52872.4);
    ```

3. テーブルのデータを照会します。

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

4. `sum`関数を使用して合計計算を行います。

    例1：`region_num`でグループ化して`sales`の合計を計算し、各地域の売上総額を計算します。

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

    例2：`region_num`でグループ化して`income`の合計を計算し、各地域の従業員の収入合計を計算します。`sum`関数はNULL値を無視するため、`id`が`777326`の従業員の収入は計算に含まれません。

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

    例3：従業員の趣味の数の合計を計算します。`hobby`列はSTRING型の数字であり、計算時にDOUBLE型に暗黙的に変換されます。

    ```Plain Text
    MySQL > select sum(DISTINCT hobby) from employees;

    +---------------------+
    | sum(DISTINCT hobby) |
    +---------------------+
    |                  15 |
    +---------------------+
    1 row in set (0.01 sec)
    ```

    例4：WHERE条件句を組み合わせて、月収が30000以上の従業員の収入合計を計算します。

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
