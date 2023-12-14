---
displayed_sidebar: "Chinese"
---

# ST_X

## 描述

如果点是有效的点类型，则返回相应的X坐标值。

## 语法

```Haskell
DOUBLE ST_X(POINT point)
```

## 示例

```Plain Text
MySQL > SELECT ST_X(ST_Point(24.7, 56.7));
+----------------------------+
| st_x(st_point(24.7, 56.7)) |
+----------------------------+
|                       24.7 |
+----------------------------+
```

## 关键词

ST_X,ST,X