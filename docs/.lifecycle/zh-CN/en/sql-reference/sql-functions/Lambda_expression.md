```
---
displayed_sidebar: "Chinese"
---

# Lambda 表达式

Lambda表达式是可以作为参数传递到高阶SQL函数中的匿名函数。Lambda表达式使您能够编写更简洁、优雅和可扩展的代码。

Lambda表达式使用`->`运算符编写，它的意思是"指向"。`->`的左侧是输入参数（如果有的话），右侧是一个表达式。

从v2.5开始，StarRocks支持在以下高阶SQL函数中使用lambda表达式：[array_map()](./array-functions/array_map.md)、[array_filter()](./array-functions/array_filter.md)、[array_sum()](./array-functions/array_sum.md)和[array_sortby()](./array-functions/array_sortby.md)。

## 语法

```Haskell
parameter -> expression
```

## 参数

- `parameter`：lambda表达式的输入参数，可以接受零个、一个或多个参数。两个或更多输入参数需用括号括起来。

- `expression`：引用`parameter`的简单表达式。该表达式必须对输入参数有效。

## 返回值

返回值的类型由`expression`的结果类型确定。

## 使用注意事项

几乎所有标量函数都可以在lambda体中使用。但有一些例外情况：

- 不支持子查询，例如 `x -> 5 + (SELECT 3)`。
- 不支持聚合函数，例如 `x -> min(y)`。
- 不支持窗口函数。
- 不支持表函数。
- lambda函数中不能出现相关列。

## 示例

Lambda表达式的简单示例：

```SQL
-- 不接受任何参数，返回5。
() -> 5    
-- 接受x并返回(x + 2)的值。
x -> x + 2 
-- 接受x和y，并返回它们的和。
(x, y) -> x + y 
-- 接受x并对x应用函数。
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
```