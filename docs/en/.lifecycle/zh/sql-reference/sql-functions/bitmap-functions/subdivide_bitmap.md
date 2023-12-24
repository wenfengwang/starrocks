---
displayed_sidebar: English
---

# subdivide_bitmap

## 描述

将一个大位图拆分为多个子位图。

此函数主要用于导出位图。太大的位图将超过MySQL协议中允许的最大数据包大小。

此功能从 v2.5 版本开始支持。

## 语法

```Haskell
BITMAP subdivide_bitmap(bitmap, length)
```

## 参数

`bitmap`：需要拆分的位图，必填。
`length`：每个子位图的最大长度，必填。大于此值的位图将被拆分为多个小位图。

## 返回值

返回多个不大于 `length` 的子位图。

## 例子

假设有一个表 `t1`，其中的 `c2` 列是 BITMAP 列。

```Plain
-- 使用 bitmap_to_string() 将 `c2` 中的值转换为字符串。
mysql> select c1, bitmap_to_string(c2) from t1;
+------+----------------------+
| c1   | bitmap_to_string(c2) |
+------+----------------------+
|    1 | 1,2,3,4,5,6,7,8,9,10 |
+------+----------------------+

-- 将 `c2` 拆分为最大长度为 3 的小位图。

mysql> select c1, bitmap_to_string(subdivide_bitmap(c2, 3)) from t1;
+------+------------------------------------+
| c1   | bitmap_to_string(subdivide_bitmap) |
+------+------------------------------------+
|    1 | 1,2,3                              |
|    1 | 4,5,6                              |
|    1 | 7,8,9                              |
|    1 | 10                                 |
+------+------------------------------------+
```