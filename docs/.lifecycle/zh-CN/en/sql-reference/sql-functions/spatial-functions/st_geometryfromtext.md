---
displayed_sidebar: "中文"
---

# ST_GeometryFromText,ST_GeomFromText

## Description

将WKT（Well Known Text）转换为相应的内存几何图形。

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

ST_GEOMETRYFROMTEXT,ST_GEOMFROMTEXT,ST,GEOMETRYFROMTEXT,GEOMFROMTEXT