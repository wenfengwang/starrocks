---
displayed_sidebar: English
---

# 翻訳

## 説明

文字列内の指定された文字を置換します。これは、文字列(`source`)を入力として受け取り、`source`内の`from_string`の文字を`to_string`に置き換えることで機能します。

この関数はv3.2からサポートされています。

## 構文

```Haskell
TRANSLATE(source, from_string, to_string)
```

## パラメーター

- `source`: `VARCAHR`型をサポートします。翻訳するソース文字列です。`source`内の文字が`from_string`に見つからない場合、その文字は結果文字列にそのまま含まれます。

- `from_string`: `VARCAHR`型をサポートします。`from_string`の各文字は、`to_string`の対応する文字に置き換えられるか、対応する文字がない場合（つまり、`to_string`が`from_string`よりも文字数が少ない場合）、その文字は結果の文字列から除外されます。例2と3を参照してください。`from_string`に文字が複数回出現する場合、最初に出現した文字のみが有効です。例5を参照してください。

- `to_string`: `VARCAHR`型をサポートします。文字の置換に使用される文字列です。`to_string`に`from_string`の引数よりも多くの文字が指定されている場合、余分な文字は無視されます。例4を参照してください。

## 戻り値

`VARCHAR`型の値を返します。

結果が`NULL`になるシナリオ:

- 入力パラメータがいずれも`NULL`です。

- 翻訳後の結果文字列の長さが`VARCHAR`の最大長（1048576）を超える場合。

## 例

```plaintext
-- 'ab'をソース文字列で'12'に置換します。
mysql > select translate('abcabc', 'ab', '12') as test;
+--------+
| test   |
+--------+
| 12c12c |
+--------+

-- 'mf1'をソース文字列で'to'に置換します。'to'は'mf1'より文字数が少なく、'1'は結果文字列から除外されます。
mysql > select translate('s1m1a1rrfcks','mf1','to') as test;
+-----------+
| test      |
+-----------+
| starrocks |
+-----------+

-- 'ab'をソース文字列で'1'に置換します。'1'は'ab'より文字数が少なく、'b'は結果文字列から除外されます。
mysql > select translate('abcabc', 'ab', '1') as test;
+------+
| test |
+------+
| 1c1c |
+------+

-- 'ab'をソース文字列で'123'に置換します。'123'は'ab'より文字数が多く、'3'は無視されます。
mysql > select translate('abcabc', 'ab', '123') as test;
+--------+
| test   |
+--------+
| 12c12c |
+--------+

-- 'aba'をソース文字列で'123'に置換します。'a'が2回出現しますが、最初の'a'のみが置換されます。
mysql > select translate('abcabc', 'aba', '123') as test;
+--------+
| test   |
+--------+
| 12c12c |
+--------+

-- この関数をrepeat()とconcat()と共に使用します。結果文字列がVARCHARの最大長を超えるため、NULLが返されます。
mysql > select translate(concat('bcde', repeat('a', 1024*1024-3)), 'a', 'z') as test;
+--------+
| test   |
+--------+
| NULL   |
+--------+

-- この関数をlength()、repeat()、concat()と共に使用して、結果文字列の長さを計算します。
mysql > select length(translate(concat('bcd', repeat('a', 1024*1024-3)), 'a', 'z')) as test;
+---------+
| test    |
+---------+
| 1048576 |
+---------+
```

## 関連項目

- [concat](./concat.md)
- [length](./length.md)
- [repeat](./repeat.md)
