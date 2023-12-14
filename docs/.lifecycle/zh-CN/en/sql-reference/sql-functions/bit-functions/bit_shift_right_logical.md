---
displayed_sidebar: "Chinese"
---

# bit_shift_right

## 描述

将数字表达式的二进制表示向右移动指定的位数。

此函数执行**算术右移**，在此过程中位长度不变，低位被舍弃，符号位被用作高位。例如，将`10101`向右移动一位的结果是`11010`。

## 语法

```Haskell
bit_shift_right(value, shift)
```

## 参数

`value`: 要进行移位的值或数字表达式。支持的数据类型为TINYINT、SMALLINT、INT、BIGINT和LARGEINT。

`shift`: 要移动的位数。支持的数据类型为BIGINT。

## 返回值

返回与`value`相同类型的值。

## 使用说明

- 如果任何输入参数为NULL，则返回NULL。
- 如果`shift`小于0，则返回0。
- 通过`0`移位`value`始终得到原始`value`。
- 通过`shift`移位`0`始终得到`0`。
- 如果`value`的数据类型为数字但不是整数，则该值将被转换为整数。参见[示例](#示例)。
- 如果`value`的数据类型为字符串，且该值如果可能将被转换为整数。例如，字符串"2.3"将被转换为2。如果值无法转换为整数，则该值将被视为NULL。参见[示例](#示例)。

## 示例

使用此函数来移动数字值。

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

## 参考

- [bit_shift_left](bit_shift_left.md)

- [bit_shift_right_logical](bit_shift_right_logical.md)