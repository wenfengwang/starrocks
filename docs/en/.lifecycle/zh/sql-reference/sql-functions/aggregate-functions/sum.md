---
displayed_sidebar: English
---

# 求和

## 描述

返回`expr`的非空值的总和。您可以使用DISTINCT关键字来计算不同的非空值的总和。

## 语法

```Haskell
SUM([DISTINCT] expr)
```

## 参数

`expr`：计算结果为数值的表达式。支持的数据类型为TINYINT、SMALLINT、INT、FLOAT、DOUBLE或DECIMAL。

## 返回值

输入值和返回值之间的数据类型映射：

- TINYINT -> BIGINT
- SMALLINT -> BIGINT
- INT -> BIGINT
- FLOAT -> DOUBLE
- DOUBLE -> DOUBLE
- DECIMAL -> DECIMAL

## 使用说明

- 此函数忽略空值。
- 如果`expr`不存在，则返回错误。
- 如果传递了VARCHAR表达式，则此函数将输入隐式转换为DOUBLE值。如果转换失败，则返回错误。

## 例子

1. 创建一个名为`employees`的表。

    ```SQL
    CREATE TABLE IF NOT EXISTS employees (
        region_num    TINYINT        COMMENT "范围[-128, 127]",
        id            BIGINT         COMMENT "范围[-2^63 + 1 ~ 2^63 - 1]",
        hobby         STRING         NOT NULL COMMENT "上限值65533字节",
        income        DOUBLE         COMMENT "8字节",
        sales         DECIMAL(12,4)  COMMENT ""
        )
        DISTRIBUTED BY HASH(region_num);
    ```

2. 将数据插入到`employees`中。

    ```SQL
    INSERT INTO employees VALUES
    (3,432175,'3',25600,1250.23),
    (4,567832,'3',37932,2564.33),
    (3,777326,'2',null,1932.99),
    (5,342611,'6',43727,45235.1),
    (2,403882,'4',36789,52872.4);
    ```

3. 从`employees`查询数据。

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

4. 使用此函数计算总和。

    示例1：计算每个区域的总销售额。

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

    示例2：计算每个区域的员工总收入。此函数忽略空值，员工ID为`777326`的收入不计算在内。

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

    示例3：计算爱好的总数。`hobby`列是STRING类型，在计算过程中将被隐式转换为DOUBLE。

    ```Plain Text
    MySQL > select sum(DISTINCT hobby) from employees;

    +---------------------+
    | sum(DISTINCT hobby) |
    +---------------------+
    |                  15 |
    +---------------------+
    1 row in set (0.01 sec)
    ```

    示例4：使用`sum`和WHERE子句计算月收入高于30000的员工的总收入。

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

## 关键词

SUM，总和
