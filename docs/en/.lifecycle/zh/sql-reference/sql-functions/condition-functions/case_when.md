---
displayed_sidebar: English
---

# CASE

## 描述

CASE 是一个条件表达式。如果 WHEN 子句中的条件计算结果为 true，则返回 THEN 子句中的结果。如果没有任何条件为 true，则返回可选的 ELSE 子句中的结果。如果没有 ELSE 子句，则返回 NULL。

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

对于这种语法，`expression` 会与 WHEN 子句中的每个表达式进行比较。如果找到相等的表达式，则返回 THEN 子句中的结果。如果没有找到相等的表达式，且存在 ELSE 子句，则返回 ELSE 子句中的结果。

- 搜索 CASE

```SQL
CASE WHEN condition1 THEN result1
    [WHEN condition2 THEN result2]
    ...
    [WHEN conditionN THEN resultN]
    [ELSE result]
END
```

对于这种语法，WHEN 子句中的每个条件都会被评估，直到找到一个为 true 的条件，然后返回 THEN 子句中对应的结果。如果没有任何条件为 true，且存在 ELSE 子句，则返回 ELSE 子句中的结果。

第一个 CASE 形式等价于第二个形式如下：

```SQL
CASE WHEN expression = expression1 THEN result1
    [WHEN expression = expression2 THEN result2]
    ...
    [WHEN expression = expressionN THEN resultN]
    [ELSE result]
END
```

## 参数

- `expressionN`：要比较的表达式。多个表达式必须在数据类型上兼容。

- `conditionN`：可以计算为 BOOLEAN 值的条件。

- `resultN`：必须在数据类型上兼容。

## 返回值

返回值是 THEN 子句中所有结果类型的共同数据类型。

## 示例

假设表 `test_case` 有以下数据：

```SQL
CREATE TABLE test_case(
    name          STRING,
    gender        INT
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
+-------+--------+
```

### 使用简单 CASE

- 指定了 ELSE，如果没有找到相等的表达式，则返回 ELSE 中的结果。

```plain
mysql> select gender, case gender 
                    when 1 then 'male'
                    when 0 then 'female'
                    else 'error'
               end as gender_str
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

- 未指定 ELSE，如果没有条件为 true，则返回 NULL。

```plain
select gender, case gender 
                    when 1 then 'male'
                    when 0 then 'female'
               end as gender_str
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

### 使用搜索 CASE 且未指定 ELSE

```plain
mysql> select gender, case when gender = 1 then 'male'
                           when gender = 0 then 'female'
                      end as gender_str
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