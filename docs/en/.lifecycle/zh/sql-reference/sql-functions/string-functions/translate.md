---
displayed_sidebar: English
---

# 翻译

## 描述

替换字符串中的指定字符。它的工作原理是接受一个字符串 (`source`) 作为输入，并用 `to_string` 中的字符替换 `source` 中的 `from_string` 字符。

此函数从 v3.2 版本开始支持。

## 语法

```Haskell
TRANSLATE(source, from_string, to_string)
```

## 参数

- `source`：支持 `VARCHAR` 类型。要进行翻译的源字符串。如果在 `source` 中找不到 `from_string` 中的字符，则该字符将简单地包含在结果字符串中。

- `from_string`：支持 `VARCHAR` 类型。`from_string` 中的每个字符要么被其在 `to_string` 中对应的字符替换，要么如果没有对应的字符（即如果 `to_string` 的字符数少于 `from_string` 的字符数），则该字符将从结果字符串中排除。参见示例 2 和 3。如果一个字符在 `from_string` 中出现多次，则只有其第一次出现有效。参见示例 5。

- `to_string`：支持 `VARCHAR` 类型。用于替换字符的字符串。如果 `to_string` 中指定的字符数多于 `from_string` 中指定的字符数，则将忽略 `to_string` 中的额外字符。参见示例 4。

## 返回值

返回 `VARCHAR` 类型的值。

结果为 `NULL` 的情况：

- 任一输入参数为 `NULL`。

- 翻译后的结果字符串长度超过 `VARCHAR` 的最大长度（1048576）。

## 例子

```plaintext
-- 用 '12' 替换源字符串中的 'ab'。
mysql > select translate('abcabc', 'ab', '12') as test;
+--------+
| test   |
+--------+
| 12c12c |
+--------+

-- 用 'to' 替换源字符串中的 'mf1'。'to' 的字符数少于 'mf1'，因此结果字符串中排除了 '1'。
mysql > select translate('s1m1a1rrfcks','mf1','to') as test;
+-----------+
| test      |
+-----------+
| starrocks |
+-----------+

-- 用 '1' 替换源字符串中的 'ab'。'1' 的字符数少于 'ab'，因此结果字符串中排除了 'b'。
mysql > select translate('abcabc', 'ab', '1') as test;
+------+
| test |
+------+
| 1c1c |
+------+

-- 用 '123' 替换源字符串中的 'ab'。'123' 的字符数多于 'ab'，因此忽略了 '3'。
mysql > select translate('abcabc', 'ab', '123') as test;
+--------+
| test   |
+--------+
| 12c12c |
+--------+

-- 用 '123' 替换源字符串中的 'aba'。'a' 出现了两次，只有第一次出现的 'a' 被替换。
mysql > select translate('abcabc', 'aba', '123') as test;
+--------+
| test   |
+--------+
| 12c12c |
+--------+

-- 将此函数与 repeat() 和 concat() 结合使用。结果字符串超过了 `VARCHAR` 的最大长度，因此返回 NULL。
mysql > select translate(concat('bcde', repeat('a', 1024*1024-3)), 'a', 'z') as test;
+--------+
| test   |
+--------+
| NULL   |
+--------+

-- 将此函数与 length()、repeat() 和 concat() 结合使用，以计算结果字符串的长度。
mysql > select length(translate(concat('bcd', repeat('a', 1024*1024-3)), 'a', 'z')) as test;
+---------+
| test    |
+---------+
| 1048576 |
+---------+
```

## 另请参阅

- [concat](./concat.md)
- [length](./length.md)
- [repeat](./repeat.md)
