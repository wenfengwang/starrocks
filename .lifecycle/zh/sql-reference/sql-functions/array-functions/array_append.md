---
displayed_sidebar: English
---

# 数组追加

## 描述

在数组的末尾添加一个新元素。返回修改后的数组。

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
1 row in set (0.00 sec)

```

您可以在数组中添加 NULL 值。

```plain
mysql> select array_append([1, 2], NULL);
+---------------------------+
| array_append([1,2], NULL) |
+---------------------------+
| [1,2,NULL]                |
+---------------------------+
1 row in set (0.01 sec)

```

## 关键字

ARRAY_APPEND，数组
