---
displayed_sidebar: "英文"
---

# CASE

## 描述

CASE 是一个条件表达式。如果WHEN子句中的条件求值为true，它将返回THEN子句中的结果。如果没有任何条件求值为true，它将返回可选的ELSE子句中的结果。如果没有ELSE子句，则返回NULL。

## 语法

CASE表达式有两种形式：

- 简单CASE

```SQL
CASE expression
    WHEN expression1 THEN result1
    [WHEN expression2 THEN result2]
    ...
    [WHEN expressionN THEN resultN]
    [ELSE result]
END
```

对于这种语法，将会把`expression`与WHEN子句中的每个表达式进行比较。如果找到相等的表达式，则返回THEN子句中的结果。如果未找到相等表达式且存在ELSE，则返回ELSE子句中的结果。

- 搜索CASE

```SQL
CASE WHEN condition1 THEN result1
    [WHEN condition2 THEN result2]
    ...
    [WHEN conditionN THEN resultN]
    [ELSE result]
END
```

对于这种语法，将对WHEN子句中的每个条件进行求值，直到有一个条件为true，然后返回THEN子句中对应的结果。如果没有条件为true且存在ELSE，则返回ELSE子句中的结果。

第一个CASE等同于第二个如下：

```SQL
CASE WHEN expression = expression1 THEN result1
    [WHEN expression = expression2 THEN result2]
    ...
    [WHEN expression = expressionN THEN resultN]
    [ELSE result]
END
```

## 参数

- `expressionN`：要进行比较的表达式。多个表达式的数据类型必须兼容。

- `conditionN`：可以求值为BOOLEAN值的条件。

- `resultN`：必须与数据类型兼容。

## 返回值

返回值为THEN子句中所有类型的公共类型。

## 示例

假设表`test_case`有以下数据：

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

### 使用简单CASE

- 指定ELSE，如果没有相等的表达式则返回ELSE中的结果。

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

- 未指定ELSE，如果没有条件为true则返回NULL。

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

### 使用未指定ELSE的搜索CASE

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