---
displayed_sidebar: English
---

# ST_Point

## 描述

返回给定 X 坐标和 Y 坐标的对应 Point。目前这个值只在球面集上有意义。X/Y 对应于经度/纬度。

> **注意**
> 如果直接选择 `ST_Point()`，可能会导致查询卡死。

## 语法

```Haskell
POINT ST_Point(DOUBLE x, DOUBLE y)
```

## 示例

```Plain
MySQL > SELECT ST_AsText(ST_Point(24.7, 56.7));
+---------------------------------+
| st_astext(st_point(24.7, 56.7)) |
+---------------------------------+
| POINT (24.7 56.7)               |
+---------------------------------+
```

## 关键字

ST_POINT,ST,POINT