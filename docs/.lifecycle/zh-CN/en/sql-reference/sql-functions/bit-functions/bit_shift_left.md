---
displayed_sidebar: "Chinese"
---

# bit_shift_left（位左移）

## 描述

将数字表达式的二进制表示向左移动指定的位数。

该函数执行**算术左移**，在此过程中，位长度不变，0 附加到末尾，高位保持不变。例如，将 `10101` 向左移动一个位结果为 `11010`。

## 语法

```Haskell
bit_shift_left(value, shift)
```

## 参数

`value`：要移位的值或数字表达式。支持的数据类型有 TINYINT、SMALLINT、INT、BIGINT 和 LARGEINT。

`shift`：要移动的位数。支持的数据类型为 BIGINT。

## 返回值

返回与 `value` 相同类型的值。

## 使用注意事项

- 如果任何输入参数为 NULL，则返回 NULL。
- 如果 `shift` 小于 0，则返回 0。
- 将 `value` 移位 `0` 始终导致原始 `value`。
- 将 `0` 移位 `shift` 始终导致 `0`。
- 如果 `value` 的数据类型为数值但不是整数，则该值将被转换为整数。请参阅[示例](#examples)。
- 如果 `value` 的数据类型为字符串，且该值可能被转换为整数，则将其转换为整数。如果该值无法转换为整数，则将其处理为 NULL。请参阅[示例](#examples)。

## 示例

使用此函数来移位数值。

```Plain
SELECT bit_shift_left(2, 1);
+----------------------+
| bit_shift_left(2, 1) |
+----------------------+
|                    4 |
+----------------------+

SELECT bit_shift_left(2.2, 1);
+------------------------+
| bit_shift_left(2.2, 1) |
+------------------------+
|                      4 |
+------------------------+

SELECT bit_shift_left("2", 1);
+------------------------+
| bit_shift_left('2', 1) |
+------------------------+
|                      4 |
+------------------------+

SELECT bit_shift_left(-2, 1);
+-----------------------+
| bit_shift_left(-2, 1) |
+-----------------------+
|                    -4 |
+-----------------------+
```

## 参考

- [bit_shift_right（位右移）](bit_shift_right.md)
- [bit_shift_right_logical（逻辑右移）](bit_shift_right_logical.md)