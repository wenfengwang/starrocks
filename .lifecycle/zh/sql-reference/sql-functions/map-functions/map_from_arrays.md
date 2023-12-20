---
displayed_sidebar: English
---

# 从数组中创建映射

## 描述

从给定的键数组和值数组中创建一个MAP值。

该函数从v3.1版本开始支持。

## 语法

```Haskell
MAP map_from_arrays(ARRAY keys, ARRAY values)
```

## 参数

- keys：用来构建结果MAP的键。请确保keys的元素是唯一的。
- values：用来构建结果MAP的值。

## 返回值

返回一个由输入的keys和values构建的MAP。

- keys和values必须长度一致，否则会返回错误。
- 如果key或value为NULL，该函数将返回NULL。
- 返回的MAP值拥有唯一的键。

## 示例

```Plaintext
select map_from_arrays([1, 2], ['Star', 'Rocks']);
+--------------------------------------------+
| map_from_arrays([1, 2], ['Star', 'Rocks']) |
+--------------------------------------------+
| {1:"Star",2:"Rocks"}                       |
+--------------------------------------------+
```

```Plaintext
select map_from_arrays([1, 2], NULL);
+-------------------------------+
| map_from_arrays([1, 2], NULL) |
+-------------------------------+
| NULL                          |
+-------------------------------+

select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']);
+--------------------------------------------------------------------------+
| map_from_arrays([1, 3, NULL, 2, NULL], ['ab', 'cdd', NULL, NULL, 'abc']) |
+--------------------------------------------------------------------------+
| {1:"ab",3:"cdd",2:null,null:"abc"}                                       |
+--------------------------------------------------------------------------+
```
