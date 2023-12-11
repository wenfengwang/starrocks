---
displayed_sidebar: "Japanese"
---

# ST_Y

## Description

ポイントが有効なポイントタイプである場合、対応するY座標値を返します。

## Syntax

```Haskell
DOUBLE ST_Y(POINT point)
```

## Examples

```Plain Text
MySQL > SELECT ST_Y(ST_Point(24.7, 56.7));
+----------------------------+
| st_y(st_point(24.7, 56.7)) |
+----------------------------+
|                       56.7 |
+----------------------------+
```

## keyword

ST_Y,ST,Y