---
displayed_sidebar: English
---

# 除法

## 描述

返回 x 除以 y 的商。如果 y 是 0，则返回 null。

## 语法

```Haskell
divide(x, y)
```

### 参数

- x：支持的类型包括 DOUBLE、FLOAT、LARGEINT、BIGINT、INT、SMALLINT、TINYINT、DECIMALV2、DECIMAL32、DECIMAL64、DECIMAL128。

- y：支持的类型与 x 相同。

## 返回值

返回一个 DOUBLE 数据类型的值。

## 使用须知

如果指定的值不是数字类型，该函数将返回 NULL。

## 示例

```Plain
mysql> select divide(3, 2);
+--------------+
| divide(3, 2) |
+--------------+
|          1.5 |
+--------------+
1 row in set (0.00 sec)
```
