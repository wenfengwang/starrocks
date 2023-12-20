---
displayed_sidebar: English
---

# ST_LineFromText、ST_LineStringFromText

## 描述

将 WKT（Well-Known Text，众所周知的文本）转换为线对象的内存表示形式。

## 语法

```Haskell
GEOMETRY ST_LineFromText(VARCHAR wkt)
```

## 示例

```Plain
MySQL > SELECT ST_AsText(ST_LineFromText("LINESTRING (1 1, 2 2)"));
+---------------------------------------------------------+
| st_astext(st_linefromtext('LINESTRING (1 1, 2 2)'))     |
+---------------------------------------------------------+
| LINESTRING (1 1, 2 2)                                   |
+---------------------------------------------------------+
```

## 关键字

ST_LINEFROMTEXT、ST_LINESTRINGFROMTEXT、ST、LINEFROMTEXT、LINESTRINGFROMTEXT