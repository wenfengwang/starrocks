---
displayed_sidebar: English
---

# sum

## 描述

返回 `expr` 的非空值之和。您可以使用 DISTINCT 关键字来计算不同非空值的和。

## 语法

```Haskell
SUM([DISTINCT] expr)
```

## 参数

`expr`：计算结果为数值的表达式。支持的数据类型包括 TINYINT、SMALLINT、INT、FLOAT、DOUBLE 或 DECIMAL。

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
- 如果 `expr` 不存在，则返回错误。
- 如果传递的是 VARCHAR 表达式，则此函数会隐式地将输入转换为 DOUBLE 值。如果转换失败，则返回错误。

## 示例

1. 创建一个名为 `employees` 的表。

   ```SQL
   CREATE TABLE IF NOT EXISTS employees (
       region_num    TINYINT        COMMENT "范围 [-128, 127]",
       id            BIGINT         COMMENT "范围 [-2^63 + 1, 2^63 - 1]",
       hobby         STRING         NOT NULL COMMENT "上限值 65533 字节",
       income        DOUBLE         COMMENT "8 字节",
       sales         DECIMAL(12,4)  COMMENT ""
       )
       DISTRIBUTED BY HASH(region_num);
   ```

2. 向 `employees` 表插入数据。

   ```SQL
   INSERT INTO employees VALUES
   (3,432175,'3',25600,1250.23),
   (4,567832,'3',37932,2564.33),
   (3,777326,'2',NULL,1932.99),
   (5,342611,'6',43727,45235.1),
   (2,403882,'4',36789,52872.4);
   ```

3. 从 `employees` 表查询数据。

   ```Plain
   MySQL > SELECT * FROM employees;
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

   示例 1：计算每个区域的总销售额。

   ```Plain
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

   示例 2：计算各区域员工的总收入。此函数忽略空值，因此不计算员工 ID `777326` 的收入。

   ```Plain
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

   示例 3：计算不同爱好的总数。`hobby` 列是 STRING 类型，在计算过程中会隐式转换为 DOUBLE。

   ```Plain
   MySQL > SELECT SUM(DISTINCT hobby) FROM employees;
   
   +---------------------+
   | SUM(DISTINCT hobby) |
   +---------------------+
   |                  15 |
   +---------------------+
   1 row in set (0.01 sec)
   ```

   示例 4：使用 `SUM` 和 WHERE 子句计算月收入超过 30000 的员工的总收入。

   ```Plain
   MySQL > SELECT SUM(income) FROM employees
   WHERE income > 30000;
   
   +-------------+
   | SUM(income) |
   +-------------+
   |      118448 |
   +-------------+
   1 row in set (0.00 sec)
   ```

## 关键词

SUM, sum