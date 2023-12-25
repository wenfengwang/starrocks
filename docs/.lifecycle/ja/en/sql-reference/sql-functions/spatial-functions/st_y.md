---
displayed_sidebar: English
---

# ST_Y

## 説明

point が有効な Point 型であれば、対応する Y 座標値を返します。

## 構文

```Haskell
DOUBLE ST_Y(POINT point)
```

## 例

```Plain Text
MySQL > SELECT ST_Y(ST_Point(24.7, 56.7));
+----------------------------+
| ST_Y(ST_Point(24.7, 56.7)) |
+----------------------------+
|                       56.7 |
+----------------------------+
```

## キーワード

ST_Y,ST,Y
