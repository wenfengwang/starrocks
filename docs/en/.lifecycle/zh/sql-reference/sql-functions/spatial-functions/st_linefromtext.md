---
displayed_sidebar: English
---

# ST_LineFromText，ST_LineStringFromText

## 描述

将 WKT（Well Known Text）转换为线的内存表示形式。

## 语法

```Haskell
GEOMETRY ST_LineFromText(VARCHAR wkt)
```

## 例子

```Plain Text
MySQL > SELECT ST_AsText(ST_LineFromText("LINESTRING (1 1, 2 2)"));
+---------------------------------------------------------+
| st_astext(st_linefromtext('LINESTRING (1 1, 2 2)'))     |
+---------------------------------------------------------+
| LINESTRING (1 1, 2 2)                                   |
+---------------------------------------------------------+
```

## 关键词

ST_LINEFROMTEXT，ST_LINESTRINGFROMTEXT，ST，LINEFROMTEXT，LINESTRINGFROMTEXT