---
displayed_sidebar: Chinese
---

# ST_Y

## 機能

`point` が有効な POINT 型の場合、対応する Y 座標値を返します。

## 文法

```Haskell
ST_Y(point)
```

## パラメータ説明

`point`: 対応するデータ型は POINT です。

## 戻り値の説明

戻り値のデータ型は DOUBLE です。

## 例

```Plain Text
MySQL > SELECT ST_Y(ST_Point(24.7, 56.7));
+----------------------------+
| st_y(st_point(24.7, 56.7)) |
+----------------------------+
|                       56.7 |
+----------------------------+
```
