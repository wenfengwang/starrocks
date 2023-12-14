---
displayed_sidebar: "Chinese"
---

# array_append

## 描述

向数组末尾添加一个新元素。返回一个数组。

## 语法

```Haskell
array_append(any_array, any_element)
```

## 举例

```plain text
mysql> select array_append([1, 2], 3);
+------------------------+
| array_append([1,2], 3) |
+------------------------+
| [1,2,3]                |
+------------------------+
1 row in set (0.00 sec)

```

您可以向数组中添加NULL。

```plain text
mysql> select array_append([1, 2], NULL);
+---------------------------+
| array_append([1,2], NULL) |
+---------------------------+
| [1,2,NULL]                |
+---------------------------+
1 row in set (0.01 sec)

```

## 关键词

ARRAY_APPEND,ARRAY