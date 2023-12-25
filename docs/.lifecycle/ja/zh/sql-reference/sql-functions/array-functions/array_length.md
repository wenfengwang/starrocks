---
displayed_sidebar: Chinese
---

# array_length

## 機能

配列内の要素数を計算し、戻り値の型は INT です。入力パラメータが NULL の場合、戻り値も NULL です。配列内の NULL 要素も長さに含まれます。

この関数の別名は [cardinality](cardinality.md) です。

## 文法

```Haskell
INT array_length(any_array)
```

## パラメータ説明

`any_array`: ARRAY 式、必須。

## 戻り値の説明

INT 型の値を返します。

## 例

```plain text
mysql> select array_length([1,2,3]);
+-----------------------+
| array_length([1,2,3]) |
+-----------------------+
|                     3 |
+-----------------------+
1 行 in set (0.00 秒)

mysql> select array_length([1,2,3,null]);
+-------------------------------+
| array_length([1, 2, 3, NULL]) |
+-------------------------------+
|                             4 |
+-------------------------------+

mysql> select array_length([[1,2], [3,4]]);
+-----------------------------+
| array_length([[1,2],[3,4]]) |
+-----------------------------+
|                           2 |
+-----------------------------+
1 行 in set (0.01 秒)
```

## キーワード

ARRAY_LENGTH, ARRAY, CARDINALITY
