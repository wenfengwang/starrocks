---
displayed_sidebar: English
---

# row

## 描述

从给定的值创建一个命名的 STRUCT 或 ROW 值。它支持无需命名的结构体。您无需指定字段名。StarRocks 会自动生成列名，如 `col1`, `col2` 等。

该函数从 v3.1 版本开始支持。

struct() 是 row() 的别名。

## 语法

```Haskell
STRUCT row(ANY val, ...)
```

## 参数

`val`：任何支持的类型的表达式。

这个函数是一个可变参数函数。您至少需要传递一个参数。`value` 可以为 null。用逗号 (`,`) 分隔多个值。

## 返回值

返回一个由输入值构成的 STRUCT 值。

## 示例

```Plaintext
select row(1,"Apple","Pear");
+-----------------------------------------+
| row(1, 'Apple', 'Pear')                 |
+-----------------------------------------+
| {"col1":1,"col2":"Apple","col3":"Pear"} |
+-----------------------------------------+

select row("Apple", NULL);
+------------------------------+
| row('Apple', NULL)           |
+------------------------------+
| {"col1":"Apple","col2":null} |
+------------------------------+

select struct(1,2,3);
+------------------------------+
| row(1, 2, 3)                 |
+------------------------------+
| {"col1":1,"col2":2,"col3":3} |
+------------------------------+
```

## 参考资料

- [STRUCT 数据类型](../../sql-statements/data-types/STRUCT.md)
- [named_struct](named_struct.md)