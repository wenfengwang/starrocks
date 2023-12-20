---
displayed_sidebar: English
---

# 案例

## 描述

CASE 是一个条件表达式。如果 WHEN 子句中的条件判断结果为真（true），则返回 THEN 子句中的结果。如果所有条件都不为真，则返回可选的 ELSE 子句中的结果。如果没有提供 ELSE 子句，则返回 NULL。

## 语法

CASE 表达式有两种形式：

- 简单 CASE

```SQL
CASE expression
    WHEN expression1 THEN result1
    [WHEN expression2 THEN result2]
    ...
    [WHEN expressionN THEN resultN]
    [ELSE result]
END
```

对于这种语法，将会把表达式与 WHEN 子句中的每个表达式进行比较。如果找到相等的表达式，则返回 THEN 子句中的结果。如果没有找到相等的表达式，并且存在 ELSE 子句，则返回 ELSE 子句中的结果。

- 搜索 CASE

```SQL
CASE WHEN condition1 THEN result1
    [WHEN condition2 THEN result2]
    ...
    [WHEN conditionN THEN resultN]
    [ELSE result]
END
```

对于这种语法，会依次评估 WHEN 子句中的每个条件，直到找到一个为真的条件，然后返回对应的 THEN 子句中的结果。如果没有任何条件为真，并且存在 ELSE 子句，则返回 ELSE 子句中的结果。

第一个 CASE 形式与第二个 CASE 形式相当如下：

```SQL
CASE WHEN expression = expression1 THEN result1
    [WHEN expression = expression2 THEN result2]
    ...
    [WHEN expression = expressionN THEN resultN]
    [ELSE result]
END
```

## 参数

- expressionN：要比较的表达式。多个表达式必须在数据类型上是兼容的。

- conditionN：能够计算为布尔值（BOOLEAN）的条件。

- resultN 必须在数据类型上是兼容的。

## 返回值

返回值是 THEN 子句中所有类型的通用类型。

## 示例

假设 test_case 表包含以下数据：

```SQL
CREATE TABLE test_case(
    name          STRING,
    gender         INT,
    ) DISTRIBUTED BY HASH(name);

INSERT INTO test_case VALUES
    ("Andy",1),
    ("Jules",0),
    ("Angel",-1),
    ("Sam",null);

SELECT * FROM test_case;
+-------+--------+
| name  | gender |
+-------+--------+
| Angel |     -1 |
| Andy  |      1 |
| Sam   |   NULL |
| Jules |      0 |
+-------+--------+-------+
```

### 使用简单 CASE

- 如果指定了 ELSE 并且没有找到相等的表达式，则返回 ELSE 子句中的结果。

```plain
mysql> select gender, case gender 
                    when 1 then 'male'
                    when 0 then 'female'
                    else 'error'
               end gender_str
from test_case;
+--------+------------+
| gender | gender_str |
+--------+------------+
|   NULL | error      |
|      0 | female     |
|      1 | male       |
|     -1 | error      |
+--------+------------+
```

- 如果未指定 ELSE 并且没有任何条件为真，则返回 NULL。

```plain
select gender, case gender 
                    when 1 then 'male'
                    when 0 then 'female'
               end gender_str
from test_case;
+--------+------------+
| gender | gender_str |
+--------+------------+
|      1 | male       |
|     -1 | NULL       |
|   NULL | NULL       |
|      0 | female     |
+--------+------------+
```

### 使用没有指定 ELSE 的搜索 CASE

```plain
mysql> select gender, case when gender = 1 then 'male'
                           when gender = 0 then 'female'
                      end gender_str
from test_case;
+--------+------------+
| gender | gender_str |
+--------+------------+
|   NULL | NULL       |
|     -1 | NULL       |
|      1 | male       |
|      0 | female     |
+--------+------------+
```

## 关键词

case when, case, case_when, case...when
