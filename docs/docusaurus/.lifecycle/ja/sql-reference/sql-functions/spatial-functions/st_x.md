---
displayed_sidebar: "Japanese"
---

# ST_X

## 説明

ポイントが有効なポイントタイプである場合、対応するX座標値を返します。

## 構文

```Haskell
DOUBLE ST_X(POINT point)
```

## 例

```Plain Text
MySQL > SELECT ST_X(ST_Point(24.7, 56.7));
+----------------------------+
| st_x(st_point(24.7, 56.7)) |
+----------------------------+
|                       24.7 |
+----------------------------+
```

## キーワード

ST_X,ST,X