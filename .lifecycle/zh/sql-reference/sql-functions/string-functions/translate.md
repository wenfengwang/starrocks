---
displayed_sidebar: English
---

# 翻译

## 描述

此函数用于替换字符串中指定的字符。它通过接收一个字符串（源字符串）作为输入，然后将源字符串中的 from_string 字符替换成 to_string 字符。

该功能从 v3.2 版本开始支持。

## 语法

```Haskell
TRANSLATE(source, from_string, to_string)
```

## 参数

- source：支持 VARCHAR 类型。要进行字符替换的源字符串。如果源字符串中的字符在 from_string 中找不到，则该字符会直接保留在结果字符串中。

- from_string：支持 VARCHAR 类型。from_string 中的每个字符将被 to_string 中相应的字符替换，如果没有相应的字符（即 to_string 中的字符少于 from_string），则该字符不会出现在结果字符串中。参见示例 2 和示例 3。如果 from_string 中的字符多次出现，只有第一次出现会被替换。参见示例 5。

- to_string：支持 VARCHAR 类型。用于替换字符的字符串。如果 to_string 中的字符数量多于 from_string 参数中的字符，那么 to_string 中多余的字符将被忽略。参见示例 4。

## 返回值

返回 VARCHAR 类型的值。

结果为 NULL 的情况：

- 任何输入参数为 NULL。

- 翻译后的结果字符串长度超出 VARCHAR 类型的最大长度（1048576）。

## 示例

```plaintext
-- Replace 'ab' in the source string with '12'.
mysql > select translate('abcabc', 'ab', '12') as test;
+--------+
| test   |
+--------+
| 12c12c |
+--------+

-- Replace 'mf1' in the source string with 'to'. 'to' has less characters than 'mf1' and '1' is excluded from the result string.
mysql > select translate('s1m1a1rrfcks','mf1','to') as test;
+-----------+
| test      |
+-----------+
| starrocks |
+-----------+

-- Replace 'ab' in the source string with '1'. '1' has less characters than 'ab' and 'b' is excluded from the result string.
mysql > select translate('abcabc', 'ab', '1') as test;
+------+
| test |
+------+
| 1c1c |
+------+

-- Replace 'ab' in the source string with '123'. '123' has more characters than 'ab' and '3' is ignored.
mysql > select translate('abcabc', 'ab', '123') as test;
+--------+
| test   |
+--------+
| 12c12c |
+--------+

-- Replace 'aba' in the source string with '123'. 'a' appears twice and only the first occurrence of 'a' is replaced.
mysql > select translate('abcabc', 'aba', '123') as test;
+--------+
| test   |
+--------+
| 12c12c |
+--------+

-- Use this function with repeat() and concat(). The result string exceeds the maximum length of VARCHAR and NULL is returned.
mysql > select translate(concat('bcde', repeat('a', 1024*1024-3)), 'a', 'z') as test;
+--------+
| test   |
+--------+
| NULL   |
+--------+

-- Use this function with length(), repeat(), and concat() to calculate the length of the result string.
mysql > select length(translate(concat('bcd', repeat('a', 1024*1024-3)), 'a', 'z')) as test;
+---------+
| test    |
+---------+
| 1048576 |
+---------+
```

## 另请参阅

- [concat（连接）](./concat.md)
- [长度](./length.md)
- [repeat](./repeat.md)（重复）
