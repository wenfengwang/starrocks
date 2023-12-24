---
displayed_sidebar: English
---

# array_append

## 描述

将一个新元素添加到数组末尾。返回一个数组。

## 语法

```Haskell
array_append(any_array, any_element)
```

## 例子

```plain text
mysql> select array_append([1, 2], 3);
+------------------------+
| array_append([1,2], 3) |
+------------------------+
| [1,2,3]                |
+------------------------+
1 行受影响 (0.00 秒)

```

您可以将 NULL 添加到数组中。

```plain text
mysql> select array_append([1, 2], NULL);
+---------------------------+
| array_append([1,2], NULL) |
+---------------------------+
| [1,2,NULL]                |
+---------------------------+
1 行受影响 (0.01 秒)

```

## 关键词

ARRAY_APPEND, ARRAY