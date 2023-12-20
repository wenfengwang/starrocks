---
displayed_sidebar: English
---

# ST_GeometryFromText、ST_GeomFromText

## 说明

将 WKT（广为人知的文本格式）转换成相应的内存几何对象。

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

ST_GEOMETRYFROMTEXT、ST_GEOMFROMTEXT、ST、GEOMETRYFROMTEXT、GEOMFROMTEXT
