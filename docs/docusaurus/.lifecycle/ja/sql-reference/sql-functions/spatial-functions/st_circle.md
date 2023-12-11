---
displayed_sidebar: "English"
---

# ST_Circle（STサークル）

## Description（説明）

Converts a WKT (WEll Known Text) to a circle on the sphere of the earth.（WKT（Well Known Text）を地球の球体上の円に変換します。）

## Syntax（構文）

```Haskell
GEOMETRY ST_Circle(DOUBLE center_lng, DOUBLE center_lat, DOUBLE radius)（GEOMETRY ST_Circle(中心経度, 中心緯度, 半径)）
```

## Parameters（パラメータ）

`center_lng` indicates the longitude of the center of the circle.（`center_lng`は円の中心の経度を示します。）

`center_lat` indicates the latitude of the center of the circle.（`center_lat`は円の中心の緯度を示します。）

`radius` indicates the radius of a circle, in meters. A maximum of 9999999 radius is supported.（`radius`は円の半径（メートル）を示します。9999999以下の半径がサポートされています。）

## Examples（例）

```Plain Text
MySQL > SELECT ST_AsText(ST_Circle(111, 64, 10000));
+--------------------------------------------+
| st_astext(st_circle(111.0, 64.0, 10000.0)) |
+--------------------------------------------+
| CIRCLE ((111 64), 10000)                   |
+--------------------------------------------+
```

## keyword（キーワード）

ST_CIRCLE,ST,CIRCLE（ST_CIRCLE,ST,CIRCLE）