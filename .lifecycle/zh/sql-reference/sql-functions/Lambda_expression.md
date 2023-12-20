---
displayed_sidebar: English
---

# 拉姆达表达式

Lambda 表达式是匿名函数，能够作为参数传递给高阶 SQL 函数。Lambda 表达式使得代码开发变得更加简洁、优雅及具有可扩展性。

Lambda 表达式采用 "->" 运算符书写，意为“映射到”。"->" 的左侧是输入参数（如有），右侧是一个表达式。

从 v2.5 版本起，StarRocks 开始支持在以下高阶 SQL 函数中使用 Lambda 表达式：[array_map()](./array-functions/array_map.md)、[array_filter()](./array-functions/array_filter.md)、[array_sum()](./array-functions/array_sum.md) 以及 [array_sortby()](./array-functions/array_sortby.md)。

## 语法

```Haskell
parameter -> expression
```

## 参数

- 参数：Lambda 表达式的输入参数，可以是零个、一个或多个。两个或更多的输入参数需用括号括起来。

- 表达式：一个引用参数的简单表达式。对于输入参数，该表达式必须是有效的。

## 返回值

返回值的类型取决于表达式的结果类型。

## 使用须知

绝大多数标量函数都可以在 Lambda 表达式的主体中使用。但有几个例外：

- 不支持子查询，例如：x -> 5 + (SELECT 3)。
- 不支持聚合函数，例如：x -> min(y)。
- 不支持窗口函数。
- 不支持表函数。
- 不允许在 Lambda 函数中使用相关列。

## 示例

Lambda 表达式的简单例子：

```SQL
-- Accepts no parameters and returns 5.
() -> 5    
-- Takes x and returns the value of (x + 2).
x -> x + 2 
-- Takes x and y, and returns their sum.
(x, y) -> x + y 
-- Takes x and applies a function to x.
x -> COALESCE(x, 0)
x -> day(x)
x -> split(x,",")
x -> if(x>0,"positive","negative")
```

在高阶函数中使用 Lambda 表达式的例子：

```Haskell
select array_map((x,y,z) -> x + y, [1], [2], [4]);
+----------------------------------------------+
| array_map((x, y, z) -> x + y, [1], [2], [4]) |
+----------------------------------------------+
| [3]                                          |
+----------------------------------------------+
1 row in set (0.01 sec)
```
