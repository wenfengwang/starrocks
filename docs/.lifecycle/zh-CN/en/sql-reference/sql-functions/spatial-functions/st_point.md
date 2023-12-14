---
displayed_sidebar: "Chinese"
---

# ST_Point

## 描述

返回具有给定X坐标和Y坐标的对应点。目前，此值仅在球形集上有意义。X/Y 对应经度/纬度。

> **注意**
>
> 如果直接选择ST_Point()，可能会卡住。

## 语法

```Haskell
POINT ST_Point(DOUBLE x, DOUBLE y)
```

## 例子

```Plain Text
MySQL > SELECT ST_AsText(ST_Point(24.7, 56.7));
+---------------------------------+
| st_astext(st_point(24.7, 56.7)) |
+---------------------------------+
| POINT (24.7 56.7)               |
+---------------------------------+
```

## 关键词

ST_POINT,ST,POINT