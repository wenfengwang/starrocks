---
displayed_sidebar: English
---

# bit_shift_right

## 描述

将数值表达式的二进制表示向右移动指定的位数。

此函数执行**算术右移**，在此过程中位数不变，低位被丢弃，符号位被用作高位。例如，将 `10101` 向右移动一位会得到 `11010`。

## 语法

```Haskell
bit_shift_right(value, shift)
```

## 参数

`value`：要移位的值或数值表达式。支持的数据类型包括 TINYINT、SMALLINT、INT、BIGINT 和 LARGEINT。

`shift`：要移位的位数。支持的数据类型为 BIGINT。

## 返回值

返回与 `value` 相同类型的值。

## 使用说明

- 如果任何输入参数为 NULL，则返回 NULL。
- 如果 `shift` 小于 0，则返回 0。
- 将 `value` 右移 0 位总是得到原始的 `value`。
- 将 0 右移 `shift` 位总是得到 0。
- 如果 `value` 的数据类型是数值而不是整数，则该值将被转换为整数。请参阅 [示例](#examples)。
- 如果 `value` 的数据类型是字符串，则如果可能，该值将被转换为整数。例如，字符串 "2.3" 将被转换为 2。如果该值无法转换为整数，则该值将被视为 NULL。请参阅[示例](#examples)。

## 例子

使用此函数可对数值进行移位。

```Plain
SELECT bit_shift_right(2, 1);
+-----------------------+
| bit_shift_right(2, 1) |
+-----------------------+
|                     1 |
+-----------------------+

SELECT bit_shift_right(2.2, 1);
+-------------------------+
| bit_shift_right(2.2, 1) |
+-------------------------+
|                       1 |
+-------------------------+

SELECT bit_shift_right("2", 1);
+-------------------------+
| bit_shift_right('2', 1) |
+-------------------------+
|                       1 |
+-------------------------+

SELECT bit_shift_right(-2, 1);
+------------------------+
| bit_shift_right(-2, 1) |
+------------------------+
|                     -1 |
+------------------------+
```

## 引用

- [bit_shift_left](bit_shift_left.md)

- [bit_shift_right_logical](bit_shift_right_logical.md)
