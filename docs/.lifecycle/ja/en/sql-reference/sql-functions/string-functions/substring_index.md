---
displayed_sidebar: English
---

# substring_index

## 説明

`count` 回の区切り文字の出現に先行または後続する部分文字列を抽出します。

- `count` が正の場合、カウントは文字列の先頭から開始され、この関数は `count` 番目の区切り文字の前にある部分文字列を返します。例えば、`select substring_index('https://www.starrocks.io', '.', 2);` は2番目の区切り文字 `.` の前の部分文字列（`https://www.starrocks`）を返します。

- `count` が負の場合、カウントは文字列の末尾から開始され、この関数は `count` 番目の区切り文字に続く部分文字列を返します。例えば、`select substring_index('https://www.starrocks.io', '.', -2);` は2番目の区切り文字 `.` の後の部分文字列（`starrocks.io`）を返します。

いずれかの入力パラメータが NULL の場合は、NULL が返されます。

この関数は v3.2 からサポートされています。

## 構文

```Haskell
VARCHAR substring_index(VARCHAR str, VARCHAR delimiter, INT count)
```

## パラメータ

- `str`: 必須、分割する文字列。
- `delimiter`: 必須、文字列を分割するために使用される区切り文字。
- `count`: 必須、区切り文字の位置。値は 0 にすることはできません。そうでなければ、NULL が返されます。値が文字列内の区切り文字の実際の数よりも大きい場合、文字列全体が返されます。

## 戻り値

VARCHAR 値を返します。

## 例

```Plain Text
-- 2番目の "." 区切り文字の前にある部分文字列を返します。
mysql> select substring_index('https://www.starrocks.io', '.', 2);
+-----------------------------------------------------+
| substring_index('https://www.starrocks.io', '.', 2) |
+-----------------------------------------------------+
| https://www.starrocks                               |
+-----------------------------------------------------+

-- count が負です。
mysql> select substring_index('https://www.starrocks.io', '.', -2);
+------------------------------------------------------+
| substring_index('https://www.starrocks.io', '.', -2) |
+------------------------------------------------------+
| starrocks.io                                         |
+------------------------------------------------------+

mysql> select substring_index("hello world", " ", 1);
+----------------------------------------+
| substring_index("hello world", " ", 1) |
+----------------------------------------+
| hello                                  |
+----------------------------------------+

mysql> select substring_index("hello world", " ", -1);
+-----------------------------------------+
| substring_index("hello world", " ", -1) |
+-----------------------------------------+
| world                                   |
+-----------------------------------------+

-- count が 0 で、NULL が返されます。
mysql> select substring_index("hello world", " ", 0);
+----------------------------------------+
| substring_index("hello world", " ", 0) |
+----------------------------------------+
| NULL                                   |
+----------------------------------------+

-- count が文字列内のスペースの数よりも大きい場合、文字列全体が返されます。
mysql> select substring_index("hello world", " ", 2);
+----------------------------------------+
| substring_index("hello world", " ", 2) |
+----------------------------------------+
| hello world                            |
+----------------------------------------+

-- count が文字列内のスペースの数よりも大きい場合、文字列全体が返されます。
mysql> select substring_index("hello world", " ", -2);
+-----------------------------------------+
| substring_index("hello world", " ", -2) |
+-----------------------------------------+
| hello world                             |
+-----------------------------------------+
```

## キーワード

substring_index
