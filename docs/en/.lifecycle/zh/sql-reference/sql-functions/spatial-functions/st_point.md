---
displayed_sidebar: English
---

# ST_Point

## 描述

返回具有给定 X 坐标和 Y 坐标的相应点。目前，此数值仅在球面集上才有意义。X/Y 对应于经度/纬度。

> **注意**
>
> 如果直接选择 ST_Point()，可能会卡住。

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

ST_POINT，ST，POINT
