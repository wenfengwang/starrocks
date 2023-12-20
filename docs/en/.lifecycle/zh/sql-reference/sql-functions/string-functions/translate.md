---
displayed_sidebar: English
---

# 翻译

## 描述

替换字符串中的指定字符。它的工作原理是将字符串（`source`）作为输入，并将`source`中的`from_string`字符替换为`to_string`。

该函数从v3.2版本开始支持。

## 语法

```Haskell
TRANSLATE(source, from_string, to_string)
```

## 参数

- `source`：支持`VARCHAR`类型。要翻译的源字符串。如果`source`中的字符在`from_string`中未找到，则仅将其包含在结果字符串中。

- `from_string`：支持`VARCHAR`类型。`from_string`中的每个字符要么被`to_string`中对应的字符替换，要么如果没有对应的字符（即，如果`to_string`的字符数少于`from_string`，则该字符将从结果字符串中排除）。请参见示例2和示例3。如果某个字符在`from_string`中出现多次，则只有第一次出现才有效。参见示例5。

- `to_string`：支持`VARCHAR`类型。用于替换字符的字符串。如果`to_string`中指定的字符多于`from_string`参数中的字符，则`to_string`中的额外字符将被忽略。参见示例4。

## 返回值

返回`VARCHAR`类型的值。

结果为`NULL`的场景：

- 任何输入参数均为`NULL`。

- 转换后的结果字符串长度超过了`VARCHAR`的最大长度（1048576）。

## 示例

```plaintext
-- 将源字符串中的'ab'替换为'12'。
mysql > select translate('abcabc', 'ab', '12') as test;
+--------+
| test   |
+--------+
| 12c12c |
+--------+

-- 将源字符串中的'mf1'替换为'to'。'to'的字符数少于'mf1'，'1'被排除在结果字符串外。
mysql > select translate('s1m1a1rrfcks','mf1','to') as test;
+-----------+
| test      |
+-----------+
| starrocks |
+-----------+

-- 将源字符串中的'ab'替换为'1'。'1'的字符数少于'ab'，'b'被排除在结果字符串外。
mysql > select translate('abcabc', 'ab', '1') as test;
+------+
| test |
+------+
| 1c1c |
+------+

-- 将源字符串中的'ab'替换为'123'。'123'的字符数多于'ab'，'3'被忽略。
mysql > select translate('abcabc', 'ab', '123') as test;
+--------+
| test   |
+--------+
| 12c12c |
+--------+

-- 将源字符串中的'aba'替换为'123'。'a'出现两次，只有第一次出现的'a'被替换。
mysql > select translate('abcabc', 'aba', '123') as test;
+--------+
| test   |
+--------+
| 12c12c |
+--------+

-- 将此函数与repeat()和concat()一起使用。结果字符串超出VARCHAR的最大长度，返回NULL。
mysql > select translate(concat('bcde', repeat('a', 1024*1024-3)), 'a', 'z') as test;
+--------+
| test   |
+--------+
| NULL   |
+--------+

-- 将此函数与length()、repeat()和concat()一起使用来计算结果字符串的长度。
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