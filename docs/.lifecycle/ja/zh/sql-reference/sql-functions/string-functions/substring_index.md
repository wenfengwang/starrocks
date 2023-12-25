---
displayed_sidebar: Chinese
---

# substring_index

## 機能

指定された文字列から、`count` 番目の区切り文字の前または後の文字列を切り取ります。

- `count` が正の数の場合、文字列の左側から数えて `count` 番目の区切り文字の前の文字列を切り取ります。例えば、`select substring_index('https://www.starrocks.io', '.', 2);` は左から数えて2番目の `.` の前の文字列を切り取り、`https://www.starrocks` を返します。

- `count` が負の数の場合、文字列の右側から数えて `count` 番目の区切り文字の後の文字列を切り取ります。例えば、`select substring_index('https://www.starrocks.io', '.', -2);` は右から数えて2番目の `.` の後の文字列を切り取り、`starrocks.io` を返します。

いずれかの入力パラメータが NULL の場合、NULL を返します。

この関数はバージョン 3.2 からサポートされています。

## 文法

```Haskell
VARCHAR substring_index(VARCHAR str, VARCHAR delimiter, INT count)
```

## パラメータ説明

- `str`：必須。切り取り対象の文字列。
- `delimiter`：必須。使用する区切り文字。
- `count`：必須。区切り文字の位置。0 ではない必要があります。0 の場合は NULL を返します。このパラメータの値が `delimiter` の実際の出現回数よりも大きい場合、文字列全体を返します。

## 戻り値の説明

VARCHAR 文字列を返します。

## 例

```Plain Text
-- 左から数えて2番目の `.` の前の文字列を切り取ります。
mysql> select substring_index('https://www.starrocks.io', '.', 2);
+-----------------------------------------------------+
| substring_index('https://www.starrocks.io', '.', 2) |
+-----------------------------------------------------+
| https://www.starrocks                               |
+-----------------------------------------------------+

-- count が負の数で、右から数えて2番目の `.` の後の文字列を切り取ります。
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

-- count が 0 の場合、NULL を返します。
mysql> select substring_index("hello world", " ", 0);
+----------------------------------------+
| substring_index("hello world", " ", 0) |
+----------------------------------------+
| NULL                                   |
+----------------------------------------+

-- count が `delimiter` の実際の出現回数よりも大きい場合、文字列全体を返します。
mysql> select substring_index("hello world", " ", 2);
+----------------------------------------+
| substring_index("hello world", " ", 2) |
+----------------------------------------+
| hello world                            |
+----------------------------------------+

-- count が `delimiter` の実際の出現回数よりも大きい場合、文字列全体を返します。
mysql> select substring_index("hello world", " ", -2);
+-----------------------------------------+
| substring_index("hello world", " ", -2) |
+-----------------------------------------+
| hello world                             |
+-----------------------------------------+
```

## キーワード

substring_index
