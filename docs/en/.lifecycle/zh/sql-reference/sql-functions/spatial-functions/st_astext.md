---
displayed_sidebar: English
---

# ST_AsText、ST_AsWKT

## 描述

将几何图形转换为 WKT（Well Known Text）格式。

## 语法

```Haskell
VARCHAR ST_AsText(GEOMETRY geo)
```

## 示例

```Plain
MySQL > SELECT ST_AsText(ST_Point(24.7, 56.7));
+---------------------------------+
| st_astext(st_point(24.7, 56.7)) |
+---------------------------------+
| POINT (24.7 56.7)               |
+---------------------------------+
```

## 关键字

ST_AsText、ST_AsWKT、ST、AsText、AsWKT