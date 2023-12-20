---
displayed_sidebar: English
---

# 求和

## 描述

返回表达式 expr 的非空值之和。您可以使用 DISTINCT 关键字计算不同非空值的和。

## 语法

```Haskell
SUM([DISTINCT] expr)
```

## 参数

expr：求值后为数值型的表达式。支持的数据类型包括 TINYINT、SMALLINT、INT、FLOAT、DOUBLE 或 DECIMAL。

## 返回值

输入值与返回值之间的数据类型对应关系：

- TINYINT -> BIGINT
- SMALLINT -> BIGINT
- INT -> BIGINT
- FLOAT -> DOUBLE
- DOUBLE -> DOUBLE
- DECIMAL -> DECIMAL

## 使用说明

- 此函数会忽略空值。
- 如果表达式 expr 不存在，则会返回错误。
- 如果传递的是 VARCHAR 类型的表达式，此函数会隐式地将其转换为 DOUBLE 类型的值。如果转换失败，则会返回错误。

## 示例

1. 创建一个名为 employees 的表。

   ```SQL
   CREATE TABLE IF NOT EXISTS employees (
       region_num    TINYINT        COMMENT "range [-128, 127]",
       id            BIGINT         COMMENT "range [-2^63 + 1 ~ 2^63 - 1]",
       hobby         STRING         NOT NULL COMMENT "upper limit value 65533 bytes",
       income        DOUBLE         COMMENT "8 bytes",
       sales       DECIMAL(12,4)  COMMENT ""
       )
       DISTRIBUTED BY HASH(region_num);
   ```

2. 向 employees 表中插入数据。

   ```SQL
   INSERT INTO employees VALUES
   (3,432175,'3',25600,1250.23),
   (4,567832,'3',37932,2564.33),
   (3,777326,'2',null,1932.99),
   (5,342611,'6',43727,45235.1),
   (2,403882,'4',36789,52872.4);
   ```

3. 从 employees 表中查询数据。

   ```Plain
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

4. 使用此函数来计算求和。

   示例 1：计算每个地区的总销售额。

   ```Plain
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

   示例 2：计算每个地区员工收入的总和。该函数会忽略空值，不包括员工编号为 777326 的员工收入。

   ```Plain
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

   示例 3：计算总共的爱好数量。爱好列的类型为 STRING，在计算过程中将被隐式转换为 DOUBLE 类型。

   ```Plain
   MySQL > select sum(DISTINCT hobby) from employees;
   
   +---------------------+
   | sum(DISTINCT hobby) |
   +---------------------+
   |                  15 |
   +---------------------+
   1 row in set (0.01 sec)
   ```

   示例 4：结合 WHERE 子句使用 sum 函数，计算月收入超过 30000 的员工的总收入。

   ```Plain
   MySQL > select sum(income) from employees
   WHERE income > 30000;
   
   +-------------+
   | sum(income) |
   +-------------+
   |      118448 |
   +-------------+
   1 row in set (0.00 sec)
   ```

## 关键字

SUM，求和
