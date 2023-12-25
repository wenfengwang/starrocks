---
displayed_sidebar: Chinese
---

# ST_Point

## 機能

指定されたX座標値とY座標値から対応するPointを返します。この値は現在、球面集合上でのみ意味を持ちます。X/Yは経度/緯度（longitude/latitude）に対応しています。

## 文法

```Haskell
ST_Point(x, y)
```

## パラメータ説明

`x`: X座標値。DOUBLE型のデータをサポートします。

`y`: Y座標値。DOUBLE型のデータをサポートします。

## 戻り値の説明

戻り値のデータ型はPOINTです。

## 例

```Plain Text
MySQL > SELECT ST_AsText(ST_Point(24.7, 56.7));
+---------------------------------+
| st_astext(st_point(24.7, 56.7)) |
+---------------------------------+
| POINT (24.7 56.7)               |
+---------------------------------+
```
