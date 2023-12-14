---
displayed_sidebar: "中文"
---

# any_value

## 描述

从每个聚合组中获取任意行。您可以使用此函数优化具有`GROUP BY`子句的查询。

## 语法

```Haskell
ANY_VALUE(expr)
```

## 参数

`expr`：被聚合的表达式。

## 返回值

从每个聚合组返回一个任意行。返回值是不确定的。

## 示例

```plaintext
// 原始数据
mysql> select * from any_value_test;
+------+------+------+
| a    | b    | c    |
+------+------+------+
|    1 |    1 |    1 |
|    1 |    2 |    1 |
|    2 |    1 |    1 |
|    2 |    2 |    2 |
|    3 |    1 |    1 |
+------+------+------+
5 rows in set (0.01 sec)

// 使用了ANY_VALUE后
mysql> select a,any_value(b),sum(c) from any_value_test group by a;
+------+----------------+----------+
| a    | any_value(`b`) | sum(`c`) |
+------+----------------+----------+
|    1 |              1 |        2 |
|    2 |              1 |        3 |
|    3 |              1 |        1 |
+------+----------------+----------+
3 rows in set (0.01 sec)

mysql> select c,any_value(a),sum(b) from any_value_test group by c;
+------+----------------+----------+
| c    | any_value(`a`) | sum(`b`) |
+------+----------------+----------+
|    1 |              1 |        5 |
|    2 |              2 |        2 |
+------+----------------+----------+
2 rows in set (0.01 sec)
```

## 关键词

ANY_VALUE