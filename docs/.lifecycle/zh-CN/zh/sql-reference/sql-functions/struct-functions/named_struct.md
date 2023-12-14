---
displayed_sidebar: "中文"
---

# named_struct

## 功能

根据给定的字段名和字段值构造 STRUCT。参数支持命名结构（named struct），使用该函数时需要指定字段名称。

该函数从 3.1 版本开始支持。

## 语法

```Haskell
STRUCT named_struct({STRING name1, ANY val1} [, ...] )
```

## 参数说明

- `nameN`: STRING 类型的字段名。

- `valN`: 可以是任意类型的值，也可以是 NULL。

name 和 value 表达式必须成对出现，否则无法创建。至少要输入一对 name 和 value，并且用逗号隔开。

## 返回值说明

返回一个 STRUCT。

## 示例

```plain
SELECT named_struct('a', 1, 'b', 2, 'c', 3);
+--------------------------------------+
| named_struct('a', 1, 'b', 2, 'c', 3) |
+--------------------------------------+
| {"a":1,"b":2,"c":3}                  |
+--------------------------------------+

SELECT named_struct('a', null, 'b', 2, 'c', 3);
+-----------------------------------------+
| named_struct('a', null, 'b', 2, 'c', 3) |
+-----------------------------------------+
| {"a":null,"b":2,"c":3}                 |
+-----------------------------------------+
```

## 相关文档

- [STRUCT 数据类型](../../sql-statements/data-types/STRUCT.md)
- [row/struct](row.md)