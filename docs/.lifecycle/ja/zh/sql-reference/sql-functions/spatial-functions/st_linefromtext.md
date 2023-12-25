---
displayed_sidebar: Chinese
---

# ST_LineFromText、ST_LineStringFromText

## 機能

WKT（Well Known Text）をライン形式のメモリ表現に変換します。

## 文法

```Haskell
ST_LineFromText(wkt)
```

## パラメータ説明

`wkt`: 変換するWKTで、サポートされるデータ型はVARCHARです。

## 戻り値の説明

戻り値のデータ型はGEOMETRYです。

## 例

```Plain Text
MySQL > SELECT ST_AsText(ST_LineFromText("LINESTRING (1 1, 2 2)"));
+---------------------------------------------------------+
| st_astext(st_linefromtext('LINESTRING (1 1, 2 2)'))     |
+---------------------------------------------------------+
| LINESTRING (1 1, 2 2)                                   |
+---------------------------------------------------------+
```
