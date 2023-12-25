---
displayed_sidebar: English
---

# ST_Circle

## 説明

WKT（Well-Known Text）を地球上の円に変換します。

## 構文

```Haskell
GEOMETRY ST_Circle(DOUBLE center_lng, DOUBLE center_lat, DOUBLE radius)
```

## パラメーター

`center_lng` は円の中心の経度を示します。

`center_lat` は円の中心の緯度を示します。

`radius` は円の半径をメートル単位で示し、最大9999999メートルの半径がサポートされています。

## 例

```Plain Text
MySQL > SELECT ST_AsText(ST_Circle(111, 64, 10000));
+--------------------------------------------+
| st_astext(st_circle(111.0, 64.0, 10000.0)) |
+--------------------------------------------+
| CIRCLE ((111 64), 10000)                   |
+--------------------------------------------+
```

## キーワード

ST_CIRCLE, ST, CIRCLE
