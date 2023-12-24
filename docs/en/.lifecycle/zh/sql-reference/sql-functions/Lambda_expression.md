---
displayed_sidebar: English
---

# Lambda 表达式

Lambda 表达式是匿名函数，可以作为参数传递到高阶 SQL 函数中。Lambda 表达式允许您开发更简洁、更优雅且可扩展的代码。

Lambda 表达式是用`->`运算符编写的，读作“goes to”。`->`的左侧是输入参数（如果有），右侧是一个表达式。

从 v2.5 开始，StarRocks支持在以下高阶 SQL函数中使用lambda表达式：[array_map](./array-functions/array_map.md)、[array_filter](./array-functions/array_filter.md)、[array_sum](./array-functions/array_sum.md)和[array_sortby](./array-functions/array_sortby.md)。

## 语法

```Haskell
parameter -> expression
```

## 参数

- `parameter`：lambda表达式的输入参数，可以接受零个、一个或多个参数。两个或多个输入参数应该用括号括起来。

- `expression`：一个简单的表达式，引用了`parameter`。表达式必须对输入参数有效。

## 返回值

返回值的类型由`expression`的结果类型确定。

## 使用说明

几乎所有标量函数都可以在lambda体中使用。但也有一些例外：

- 不支持子查询，例如 `x -> 5 + (SELECT 3)`。
- 不支持聚合函数，例如 `x -> min(y)`。
- 不支持窗口函数。
- 不支持表函数。
- 相关列不能出现在lambda函数中。

## 例子

lambda表达式的简单示例：

```SQL
-- 不接受任何参数，返回5。
() -> 5    
-- 接受x并返回(x + 2)的值。
x -> x + 2 
-- 接受x和y，并返回它们的和。
(x, y) -> x + y 
-- 接受x并对x应用一个函数。
x -> COALESCE(x, 0)
x -> day(x)
x -> split(x,",")
x -> if(x>0,"positive","negative")
```

在高阶函数中使用lambda表达式的示例：

```Haskell
select array_map((x,y,z) -> x + y, [1], [2], [4]);
+----------------------------------------------+
| array_map((x, y, z) -> x + y, [1], [2], [4]) |
+----------------------------------------------+
| [3]                                          |
+----------------------------------------------+
1 row in set (0.01 sec)