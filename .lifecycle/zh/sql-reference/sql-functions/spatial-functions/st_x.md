---
displayed_sidebar: English
---

# ST_X

## 描述

如果点是有效的 Point 类型，返回相应的 X 坐标值。

## 语法

```Haskell
DOUBLE ST_X(POINT point)
```

## 示例

```Plain
MySQL > SELECT ST_X(ST_Point(24.7, 56.7));
+----------------------------+
| st_x(st_point(24.7, 56.7)) |
+----------------------------+
|                       24.7 |
+----------------------------+
```

## 关键词

ST_X，ST，X
