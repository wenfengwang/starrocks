---
displayed_sidebar: "Chinese"
---

# bit_shift_right_logical

## 描述

将数字表达式的二进制表示向右移动指定数量的位。

此函数执行**逻辑右移操作**，在此期间，位长度不变，低位被丢弃，并且0被附加到高位，无论原始位是正还是负。**逻辑**移位是无符号移位。例如，将`10101`向右移动一位的结果是`00101`。

对于正值，bit_shift_right()和bit_shift_right_logical()返回相同的结果。

## 语法

```Haskell
bit_shift_right_logical(value, shift)
```

## 参数

`value`：要移位的值或数字表达式。支持的数据类型为TINYINT、SMALLINT、INT、BIGINT和LARGEINT。

`shift`：要移动的位数。支持的数据类型为BIGINT。

## 返回值

返回与`value`相同类型的值。

## 使用注意事项

- 如果任何输入参数为NULL，则返回NULL。
- 如果`shift`小于0，则返回0。
- 将`value`按`0`移动始终导致原始`value`。
- 将`0`按`shift`移动始终导致`0`。
- 如果`value`的数据类型为数字但不是整数，则该值将被转换为整数。参见[示例](#示例)。
- 如果`value`的数据类型是字符串，且该值如果可能将被转换为整数。例如，字符串"2.3"将被转换为2。如果该值无法转换为整数，则该值将被视为NULL。参见[示例](#示例)。

## 示例

使用此函数来移动数值。

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

## 参考

- [bit_shift_left](bit_shift_left.md)
- [bit_shift_right](bit_shift_right.md)