---
displayed_sidebar: Chinese
---

# append_trailing_char_if_absent

## 機能

もし `str` 文字列が空でなく、末尾に `trailing_char` 文字が含まれていない場合、`trailing_char` 文字を末尾に追加します。

## 文法

```Haskell
append_trailing_char_if_absent(str, trailing_char)
```

## パラメータ説明

`str`: 指定された文字列で、サポートされるデータ型は VARCHAR です。

`trailing_char`: 指定された文字で、サポートされるデータ型は VARCHAR です。`trailing_char` は1文字のみを含むことができ、複数の文字が含まれている場合は NULL を返します。

## 戻り値の説明

戻り値のデータ型は VARCHAR です。

## 例

```Plain Text
MySQL [test]> select append_trailing_char_if_absent('a','c');
+------------------------------------------+
|append_trailing_char_if_absent('a', 'c')  |
+------------------------------------------+
| ac                                       |
+------------------------------------------+
1 row in set (0.02 sec)

MySQL [test]> select append_trailing_char_if_absent('ac','c');
+-------------------------------------------+
|append_trailing_char_if_absent('ac', 'c')  |
+-------------------------------------------+
| ac                                        |
+-------------------------------------------+
1 row in set (0.00 sec)
```
