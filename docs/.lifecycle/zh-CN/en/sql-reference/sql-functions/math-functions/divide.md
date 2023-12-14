---
displayed_sidebar: "Chinese"
---

# 除法

## 描述

返回x除以y的商。如果y为0，则返回null。

## 语法

```Haskell
divide(x, y)
```

### 参数

- `x`: 支持的类型有DOUBLE、FLOAT、LARGEINT、BIGINT、INT、SMALLINT、TINYINT、DECIMALV2、DECIMAL32、DECIMAL64、DECIMAL128。

- `y`: 支持的类型与`x`相同。

## 返回值

返回一个DOUBLE数据类型的值。

## 使用注意事项

如果指定了非数字值，则此函数返回`NULL`。

## 示例

```Plain Text
mysql> select divide(3, 2);
+--------------+
| divide(3, 2) |
+--------------+
|          1.5 |
+--------------+
1 row in set (0.00 sec)
```