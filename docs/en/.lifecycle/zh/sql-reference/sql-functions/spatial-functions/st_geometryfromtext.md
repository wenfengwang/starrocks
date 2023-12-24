---
displayed_sidebar: English
---

# ST_GeometryFromText, ST_GeomFromText

## 描述

将 WKT（Well Known Text）转换为相应的内存几何图形。

## 语法

```Haskell
GEOMETRY ST_GeometryFromText(VARCHAR wkt)
```

## 例子

```Plain Text
MySQL > SELECT ST_AsText(ST_GeometryFromText("LINESTRING (1 1, 2 2)"));
+---------------------------------------------------------+
| st_astext(st_geometryfromtext('LINESTRING (1 1, 2 2)')) |
+---------------------------------------------------------+
| LINESTRING (1 1, 2 2)                                   |
+---------------------------------------------------------+
```

## 关键词

ST_GEOMETRYFROMTEXT, ST_GEOMFROMTEXT, ST, GEOMETRYFROMTEXT, GEOMFROMTEXT