```yaml
---
displayed_sidebar: "Chinese"
---

# ST_Point

## 功能

返回给定的X坐标和Y坐标对应的点，这个函数只在球面集合上有意义，X/Y对应的是经度/纬度(longitude/latitude)。

## 语法

```Haskell
ST_Point(x, y)
```

## 参数说明

`x`: X坐标的值，支持的数据类型为DOUBLE。

`y`: Y坐标的值，支持的数据类型为DOUBLE。

## 返回值说明

返回值的数据类型为POINT。

## 示例

```Plain Text
MySQL > SELECT ST_AsText(ST_Point(24.7, 56.7));
+---------------------------------+
| st_astext(st_point(24.7, 56.7)) |
+---------------------------------+
| POINT (24.7 56.7)               |
+---------------------------------+
```