---
displayed_sidebar: Chinese
---

# ST_Polygon、ST_PolyFromText、ST_PolygonFromText

## 機能

WKT（Well Known Text）を対応する多角形のメモリ形式に変換します。

## 文法

```Haskell
ST_Polygon(wkt)
```

## パラメータ説明

`wkt`: 変換される WKT。サポートされるデータ型は VARCHAR です。

## 戻り値の説明

戻り値のデータ型は GEOMETRY です。

## 例

```Plain Text
MySQL > SELECT ST_AsText(ST_Polygon("POLYGON ((0 0, 10 0, 10 10, 0 10, 0 0))"));
+------------------------------------------------------------------+
| st_astext(st_polygon('POLYGON ((0 0, 10 0, 10 10, 0 10, 0 0))')) |
+------------------------------------------------------------------+
| POLYGON ((0 0, 10 0, 10 10, 0 10, 0 0))                          |
+------------------------------------------------------------------+
```
