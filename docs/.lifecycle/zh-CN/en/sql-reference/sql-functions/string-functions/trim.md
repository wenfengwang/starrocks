---
displayed_sidebar: "English"
---

# trim

## Description

删除`str`参数的开头和结尾的连续空格或指定字符。从StarRocks 2.5.0起支持删除指定字符。

## Syntax

```Haskell
VARCHAR trim(VARCHAR str[, VARCHAR characters])
```

## Parameters

`str`: 必需，要修剪的字符串，它必须求值为VARCHAR值。

`characters`: 可选，要删除的字符，必须是VARCHAR值。如果未指定此参数，默认会从字符串中删除空格。如果将此参数设置为空字符串，则会返回错误。

## Return value

返回一个VARCHAR值。

## Examples

示例1：删除字符串开头和结尾的五个空格。

```Plain Text
MySQL > SELECT trim("   ab c  ");
+-------------------+
| trim('   ab c  ') |
+-------------------+
| ab c              |
+-------------------+
1 row in set (0.00 sec)
```

示例2：从字符串开头和结尾删除指定字符。

```Plain Text
MySQL > SELECT trim("abcd", "ad");
+--------------------+
| trim('abcd', 'ad') |
+--------------------+
| bc                 |
+--------------------+

MySQL > SELECT trim("xxabcdxx", "x");
+-----------------------+
| trim('xxabcdxx', 'x') |
+-----------------------+
| abcd                  |
+-----------------------+
```

## References

- [ltrim](ltrim.md)
- [rtrim](rtrim.md)