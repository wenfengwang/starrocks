---
displayed_sidebar: Chinese
---

# ST_AsText、ST_AsWKT

## 機能

幾何図形をWKT（Well Known Text）表記に変換します。

## 文法

```Haskell
ST_AsText(geo)
```

## パラメータ説明

`geo`: 変換対象のパラメータで、サポートされるデータ型はGEOMETRYです。

## 戻り値の説明

戻り値のデータ型はVARCHARです。

## 例

```Plain Text
MySQL > SELECT ST_AsText(ST_Point(24.7, 56.7));
+---------------------------------+
| st_astext(st_point(24.7, 56.7)) |
+---------------------------------+
| POINT (24.7 56.7)               |
+---------------------------------+
```
