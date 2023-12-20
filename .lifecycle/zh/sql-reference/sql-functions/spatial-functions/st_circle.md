---
displayed_sidebar: English
---

# ST_Circle

## 描述

将 WKT（广为人知的文本）转换成地球表面上的一个圆形。

## 语法

```Haskell
GEOMETRY ST_Circle(DOUBLE center_lng, DOUBLE center_lat, DOUBLE radius)
```

## 参数

center_lng 表示圆心的经度。

center_lat 表示圆心的纬度。

radius 表示圆的半径，单位是米。支持的最大半径为 9999999 米。

## 示例

```Plain
MySQL > SELECT ST_AsText(ST_Circle(111, 64, 10000));
+--------------------------------------------+
| st_astext(st_circle(111.0, 64.0, 10000.0)) |
+--------------------------------------------+
| CIRCLE ((111 64), 10000)                   |
+--------------------------------------------+
```

## 关键字

ST_Circle, ST, Circle
