---
displayed_sidebar: English
---

# ST_Polygon、ST_PolyFromText、ST_PolygonFromText

## 说明

将 WKT（标准文本格式）转换为相应的多边形内存表示形式。

## 语法

```Haskell
GEOMETRY ST_Polygon(VARCHAR wkt)
```

## 示例

```Plain
MySQL > SELECT ST_AsText(ST_Polygon("POLYGON ((0 0, 10 0, 10 10, 0 10, 0 0))"));
+------------------------------------------------------------------+
| st_astext(st_polygon('POLYGON ((0 0, 10 0, 10 10, 0 10, 0 0))')) |
+------------------------------------------------------------------+
| POLYGON ((0 0, 10 0, 10 10, 0 10, 0 0))                          |
+------------------------------------------------------------------+
```

## 关键字

ST_POLYGON、ST_POLYFROMTEXT、ST_POLYGONFROMTEXT、ST、POLYGON、POLYFROMTEXT、POLYGONFROMTEXT
