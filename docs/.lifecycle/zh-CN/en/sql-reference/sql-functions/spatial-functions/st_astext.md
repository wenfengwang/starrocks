---
displayed_sidebar: "Chinese"
---

# ST_AsText,ST_AsWKT

## 描述

将几何图形转换为WKT（Well Known Text）格式。

## 语法

```Haskell
VARCHAR ST_AsText(GEOMETRY geo)
```

## 例子

```Plain Text
MySQL > SELECT ST_AsText(ST_Point(24.7, 56.7));
+---------------------------------+
| st_astext(st_point(24.7, 56.7)) |
+---------------------------------+
| POINT (24.7 56.7)               |
+---------------------------------+
```

## 关键词

ST_ASTEXT,ST_ASWKT,ST,ASTEXT,ASWKT