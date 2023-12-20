---
displayed_sidebar: English
---

# ST_Polygon、ST_PolyFromText、ST_PolygonFromText

## 描述

将 WKT（Well Known Text）转换为相应的多边形内存形式。

## 语法

```Haskell
GEOMETRY ST_Polygon(VARCHAR wkt)
```

## 示例

```Plain
MySQL > SELECT ST_AsText(ST_Polygon("POLYGON ((0 0, 10 0, 10 10, 0 10, 0 0))"));
+------------------------------------------------------------------+
| ST_AsText(ST_Polygon('POLYGON ((0 0, 10 0, 10 10, 0 10, 0 0))')) |
+------------------------------------------------------------------+
| POLYGON ((0 0, 10 0, 10 10, 0 10, 0 0))                          |
+------------------------------------------------------------------+
```

## 关键字

ST_POLYGON、ST_POLYFROMTEXT、ST_POLYGONFROMTEXT、ST、POLYGON、POLYFROMTEXT、POLYGONFROMTEXT