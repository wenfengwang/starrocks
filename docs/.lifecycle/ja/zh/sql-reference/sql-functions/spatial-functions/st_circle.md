---
displayed_sidebar: Chinese
---

# ST_Circle

## 機能

WKT（Well-Known Text）を地球の表面上の円に変換します。

## 文法

```Haskell
ST_Circle(center_lng, center_lat, radius)
```

## パラメータ説明

`center_lng`：円の中心の経度を示し、サポートされるデータ型はDOUBLEです。

`center_lat`：円の中心の緯度を示し、サポートされるデータ型はDOUBLEです。

`radius`：円の半径を示し、単位は「メートル」で、最大9999999までサポートされ、サポートされるデータ型はDOUBLEです。

## 戻り値の説明

戻り値のデータ型はGEOMETRYです。

## 例

```Plain Text
MySQL > SELECT ST_AsText(ST_Circle(111, 64, 10000));
+--------------------------------------------+
| st_astext(st_circle(111.0, 64.0, 10000.0)) |
+--------------------------------------------+
| CIRCLE ((111 64), 10000)                   |
+--------------------------------------------+
```
