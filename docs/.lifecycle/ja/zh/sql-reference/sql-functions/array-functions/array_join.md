---
displayed_sidebar: Chinese
---

# array_join

## 機能

配列内のすべての要素を結合して一つの文字列を生成します。

## 文法

```Haskell
ARRAY_JOIN(array, sep[, null_replace_str])
```

## パラメータ説明

* `array`：結合する必要がある配列。サポートされているデータ型は ARRAY です。
* `sep`：区切り文字。サポートされているデータ型は VARCHAR です。
* `null_replace_str`：NULL を置き換える文字列。サポートされているデータ型は VARCHAR です。

## 戻り値の説明

戻り値のデータ型は VARCHAR です。

## 注意事項

* `array` は一次元配列のみをサポートします。
* `array` は Decimal 型をサポートしません。
* `sep` パラメータが NULL の場合、戻り値は NULL になります。
* `null_replace_str` パラメータが渡されない場合、NULL は破棄されます。
* `null_replace_str` パラメータが NULL の場合、戻り値は NULL になります。

## 例

データ配列の `NULL` を破棄し、`_` を区切り文字として配列内の要素を結合します。

```Plain Text
mysql> select array_join([1, 3, 5, null], '_');
+-------------------------------+
| array_join([1,3,5,NULL], '_') |
+-------------------------------+
| 1_3_5                         |
+-------------------------------+
```

データ配列の `NULL` を文字列 `NULL` に置き換え、`_` を区切り文字として配列内の要素を結合します。

```Plain Text
mysql> select array_join([1, 3, 5, null], '_', 'NULL');
+---------------------------------------+
| array_join([1,3,5,NULL], '_', 'NULL') |
+---------------------------------------+
| 1_3_5_NULL                            |
+---------------------------------------+
```
