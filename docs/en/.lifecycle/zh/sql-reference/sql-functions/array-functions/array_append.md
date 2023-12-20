---
displayed_sidebar: English
---

# array_append

## 描述

向数组末尾添加一个新元素。返回一个数组。

## 语法

```Haskell
array_append(any_array, any_element)
```

## 示例

```plain
mysql> select array_append([1, 2], 3);
+------------------------+
| array_append([1,2], 3) |
+------------------------+
| [1,2,3]                |
+------------------------+
1 行在集合中 (0.00 秒)

```

您可以向数组中添加 NULL。

```plain
mysql> select array_append([1, 2], NULL);
+---------------------------+
| array_append([1,2], NULL) |
+---------------------------+
| [1,2,NULL]                |
+---------------------------+
1 行在集合中 (0.01 秒)

```

## 关键字

ARRAY_APPEND, ARRAY