---
displayed_sidebar: English
---

# ST_Circle

## 描述

将 WKT（Well Known Text）转换为地球球体上的圆。

## 语法

```Haskell
GEOMETRY ST_Circle(DOUBLE center_lng, DOUBLE center_lat, DOUBLE radius)
```

## 参数

`center_lng` 表示圆的中心经度。

`center_lat` 表示圆的中心纬度。

`radius` 表示圆的半径，单位为米。支持的最大半径为9999999。

## 例子

```Plain Text
MySQL > SELECT ST_AsText(ST_Circle(111, 64, 10000));
+--------------------------------------------+
| st_astext(st_circle(111.0, 64.0, 10000.0)) |
+--------------------------------------------+
| CIRCLE ((111 64), 10000)                   |
+--------------------------------------------+
```

## 关键词

ST_CIRCLE，ST，CIRCLE
