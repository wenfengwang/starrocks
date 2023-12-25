---
displayed_sidebar: Chinese
---

# cardinality

## 機能

配列内の要素数を計算し、戻り値の型は INT です。入力パラメータが NULL の場合、戻り値も NULL です。配列内の NULL 要素も長さに含まれます。例えば `[1,2,3,null]` は 4 つの要素として計算されます。

この関数はバージョン 3.0 からサポートされています。別名は [array_length()](array_length.md) です。

## 文法

```Haskell
INT cardinality(any_array)
```

## パラメータ説明

`any_array`: ARRAY 式、必須。

## 戻り値の説明

INT 型の要素数を返します。

## 例

```plain text
mysql> select cardinality([1,2,3]);
+-----------------------+
|  cardinality([1,2,3]) |
+-----------------------+
|                     3 |
+-----------------------+
1 row in set (0.00 sec)

mysql> select cardinality([1,2,3,null]);
+------------------------------+
| cardinality([1, 2, 3, NULL]) |
+------------------------------+
|                            4 |
+------------------------------+

mysql> select cardinality([[1,2], [3,4]]);
+-----------------------------+
|  cardinality([[1,2],[3,4]]) |
+-----------------------------+
|                           2 |
+-----------------------------+
1 row in set (0.01 sec)
```

## キーワード

CARDINALITY, ARRAY_LENGTH, ARRAY
