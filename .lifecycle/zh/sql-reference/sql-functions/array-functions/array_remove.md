---
displayed_sidebar: English
---

# 数组移除

## 描述

从数组中移除一个元素。

## 语法

```Haskell
array_remove(any_array, any_element)
```

## 参数

- any_array：要操作的数组。
- any_element：与数组中元素相匹配的表达式。

## 返回值

返回已移除指定元素的数组。

## 示例

```plaintext
mysql> select array_remove([1,2,3,null,3], 3);

+---------------------------------+

| array_remove([1,2,3,NULL,3], 3) |

+---------------------------------+

| [1,2,null]                      |

+---------------------------------+

1 row in set (0.01 sec)
```

## 关键字

ARRAY_REMOVE，数组
