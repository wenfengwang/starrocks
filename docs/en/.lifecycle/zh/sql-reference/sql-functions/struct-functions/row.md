---
displayed_sidebar: English
---

# 行

## 描述

从给定的值创建一个命名的STRUCT或ROW值。它支持未命名的struct。您不需要指定字段名称。StarRocks会自动生成列名，例如 `col1, col2,...`。

此功能从v3.1开始支持。

struct()是row()的别名。

## 语法

```Haskell
STRUCT row(ANY val, ...)
```

## 参数

`val`：任何受支持的类型的表达式。

此函数是一个可变参数函数。您必须至少传递一个参数。`value`可以为null。用逗号（`,`）分隔多个值。

## 返回值

返回一个由输入值组成的STRUCT值。

## 例子

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

## 引用

- [STRUCT数据类型](../../sql-statements/data-types/STRUCT.md)
- [named_struct](named_struct.md)
