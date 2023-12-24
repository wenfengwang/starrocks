---
displayed_sidebar: English
---

# bit_shift_right_logical

## 描述

将数值表达式的二进制表示向右移动指定的位数。

此函数执行**逻辑右移**，在此过程中，位数不会改变，最低位被丢弃，并且无论原始位是正数还是负数，高位都会附加0。**逻辑**移位是无符号移位。例如，将 `10101` 右移一位会得到 `00101`。

对于正值，bit_shift_right() 和 bit_shift_right_logical() 返回相同的结果。

## 语法

```Haskell
bit_shift_right_logical(value, shift)
```

## 参数

`value`：要移位的值或数值表达式。支持的数据类型包括 TINYINT、SMALLINT、INT、BIGINT 和 LARGEINT。

`shift`：要移位的位数。支持的数据类型为 BIGINT。

## 返回值

返回与 `value` 相同类型的值。

## 使用说明

- 如果任何输入参数为 NULL，则返回 NULL。
- 如果 `shift` 小于 0，则返回 0。
- 将 `value` 右移 `0` 位总是会得到原始的 `value`。
- 将 `0` 右移 `shift` 位总是会得到 `0`。
- 如果 `value` 的数据类型是数值但不是整数，则该值将被转换为整数。请参阅[示例](#examples)。
- 如果 `value` 的数据类型是字符串，则将尝试将该值转换为整数。例如，字符串 "2.3" 将被转换为 2。如果无法将该值转换为整数，则该值将被视为 NULL。请参阅[示例](#examples)。

## 例子

使用此函数可对数值进行移位操作。

```Plain
SELECT bit_shift_right_logical(2, 1);
+-------------------------------+
| bit_shift_right_logical(2, 1) |
+-------------------------------+
|                             1 |
+-------------------------------+

SELECT bit_shift_right_logical(2.2, 1);
+---------------------------------+
| bit_shift_right_logical(2.2, 1) |
+---------------------------------+
|                               1 |
+---------------------------------+

SELECT bit_shift_right_logical("2", 1);
+---------------------------------+
| bit_shift_right_logical('2', 1) |
+---------------------------------+
|                               1 |
+---------------------------------+

SELECT bit_shift_right_logical(cast('-2' AS INTEGER(32)), 1);
+-----------------------------------------------+
| bit_shift_right_logical(CAST('-2' AS INT), 1) |
+-----------------------------------------------+
|                                    2147483647 |
+-----------------------------------------------+
```

## 引用

- [bit_shift_left](bit_shift_left.md)
- [bit_shift_right](bit_shift_right.md)
