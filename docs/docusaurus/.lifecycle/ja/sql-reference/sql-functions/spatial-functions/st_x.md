---
displayed_sidebar: "Japanese"
---

# ST_X

## Description

ポイントが有効なポイントタイプの場合、対応するX座標値を返します。

## Syntax

```Haskell
DOUBLE ST_X(POINT point)
```

## Examples

```Plain Text
MySQL > SELECT ST_X(ST_Point(24.7, 56.7));
+----------------------------+
| st_x(st_point(24.7, 56.7)) |
+----------------------------+
|                       24.7 |
+----------------------------+
```

## keyword

ST_X,ST,X