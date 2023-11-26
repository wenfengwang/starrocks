---
displayed_sidebar: "Japanese"
---

# ST_Y

## 説明

ポイントが有効なポイントタイプである場合、対応するY座標値を返します。

## 構文

```Haskell
DOUBLE ST_Y(POINT point)
```

## 例

```Plain Text
MySQL > SELECT ST_Y(ST_Point(24.7, 56.7));
+----------------------------+
| st_y(st_point(24.7, 56.7)) |
+----------------------------+
|                       56.7 |
+----------------------------+
```

## キーワード

ST_Y,ST,Y
