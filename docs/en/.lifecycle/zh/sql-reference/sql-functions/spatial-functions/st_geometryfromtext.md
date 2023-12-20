---
displayed_sidebar: English
---

# ST_GeometryFromText,ST_GeomFromText

## 描述

将 WKT（Well Known Text，通用文本标记）转换为相应的内存几何对象。

## 语法

```Haskell
GEOMETRY ST_GeometryFromText(VARCHAR wkt)
```

## 示例

```Plain
MySQL > SELECT ST_AsText(ST_GeometryFromText("LINESTRING (1 1, 2 2)"));
+---------------------------------------------------------+
| st_astext(st_geometryfromtext('LINESTRING (1 1, 2 2)')) |
+---------------------------------------------------------+
| LINESTRING (1 1, 2 2)                                   |
+---------------------------------------------------------+
```

## 关键字

ST_GEOMETRYFROMTEXT,ST_GEOMFROMTEXT,ST,GEOMETRYFROMTEXT,GEOMFROMTEXT