---
displayed_sidebar: English
---

# divide

## 描述

返回 x 除以 y 的商。如果 y 为 0，则返回 null。

## 语法

```Haskell
divide(x, y)
```

### 参数

- `x`：支持的类型有 DOUBLE、FLOAT、LARGEINT、BIGINT、INT、SMALLINT、TINYINT、DECIMALV2、DECIMAL32、DECIMAL64、DECIMAL128。

- `y`：支持的类型与 `x` 相同。

## 返回值

返回 DOUBLE 数据类型的值。

## 使用说明

如果指定的是非数值类型，则此函数返回 `NULL`。

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