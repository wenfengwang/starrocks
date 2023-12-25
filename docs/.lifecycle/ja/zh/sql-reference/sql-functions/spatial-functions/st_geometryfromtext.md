---
displayed_sidebar: Chinese
---

# ST_GeometryFromText、ST_GeomFromText

## 機能

WKT（Well Known Text）を対応するメモリ内のジオメトリ形式に変換します。

## 文法

```Haskell
ST_GeometryFromText(wkt)
```

## パラメータ説明

`wkt`: 変換される WKT。サポートされるデータ型は VARCHAR です。

## 戻り値の説明

戻り値のデータ型は GEOMETRY です。

## 例

```Plain Text
MySQL > SELECT ST_AsText(ST_GeometryFromText("LINESTRING (1 1, 2 2)"));
+---------------------------------------------------------+
| st_astext(st_geometryfromtext('LINESTRING (1 1, 2 2)')) |
+---------------------------------------------------------+
| LINESTRING (1 1, 2 2)                                   |
+---------------------------------------------------------+
```
