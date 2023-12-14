---
displayed_sidebar: "Chinese"
---

# ST_LineFromText, ST_LineStringFromText

## Function

将一个 WKT（Well Known Text）转化为一个 Line 形式的内存表现形式。

## Syntax

```Haskell
ST_LineFromText(wkt)
```

## Parameter

`wkt`: 待转化的 WKT，支持的数据类型为 VARCHAR。

## Return Value

返回值的数据类型为 GEOMETRY。

## Example

```Plain Text
MySQL > SELECT ST_AsText(ST_LineFromText("LINESTRING (1 1, 2 2)"));
+---------------------------------------------------------+
| st_astext(st_linefromtext('LINESTRING (1 1, 2 2)'))     |
+---------------------------------------------------------+
| LINESTRING (1 1, 2 2)                                   |
+---------------------------------------------------------+
```