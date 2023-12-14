---
displayed_sidebar: "Chinese"
---

# ST_Y

## 描述

如果点是有效的点类型，则返回相应的Y坐标值。

## 语法

```Haskell
DOUBLE ST_Y(POINT point)
```

## 示例

```Plain Text
MySQL > SELECT ST_Y(ST_Point(24.7, 56.7));
+----------------------------+
| st_y(st_point(24.7, 56.7)) |
+----------------------------+
|                       56.7 |
+----------------------------+
```

## 关键词

ST_Y,ST,Y