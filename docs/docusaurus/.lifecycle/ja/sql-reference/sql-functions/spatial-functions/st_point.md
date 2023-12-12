---
displayed_sidebar: "Japanese"
---

# ST_Point（ST_ポイント）

## 説明

指定されたX座標とY座標で対応するポイントを返します。現時点では、この値は球状セットにのみ意味があります。X/Yは経度/緯度に対応しています。

> **注意**
>
> 直接ST_Point()を選択すると、動作が停止する場合があります。

## 構文

```Haskell
POINT ST_Point(DOUBLE x, DOUBLE y)
```

## 例

```Plain Text
MySQL > SELECT ST_AsText(ST_Point(24.7, 56.7));
+---------------------------------+
| st_astext(st_point(24.7, 56.7)) |
+---------------------------------+
| POINT (24.7 56.7)               |
+---------------------------------+
```

## キーワード

ST_POINT,ST,POINT