---
displayed_sidebar: "Chinese"
---

# array_remove

## 描述

从数组中删除一个元素。

## 语法

```Haskell
array_remove(any_array, any_element)
```

## 参数

- `any_array`: 要搜索的数组。
- `any_element`: 与数组中的元素匹配的表达式。

## 返回值

返回已删除指定元素的数组。

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

## 关键词

ARRAY_REMOVE, ARRAY