---
displayed_sidebar: English
---

# ST_Y

## 描述

如果点是有效的 Point 类型，返回对应的 Y 坐标值。

## 语法

```Haskell
DOUBLE ST_Y(POINT point)
```

## 示例

```Plain
MySQL > SELECT ST_Y(ST_Point(24.7, 56.7));
+----------------------------+
| st_y(st_point(24.7, 56.7)) |
+----------------------------+
|                       56.7 |
+----------------------------+
```

## 关键字

ST_Y，ST，Y
