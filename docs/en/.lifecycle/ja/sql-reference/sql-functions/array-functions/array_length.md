---
displayed_sidebar: "Japanese"
---

# array_length

## 説明

配列の要素数を返します。結果の型はINTです。入力パラメータがNULLの場合、結果もNULLです。NULLの要素も長さに含まれます。

[cardinality()](cardinality.md)という別名があります。

## 構文

```Haskell
INT array_length(any_array)
```

## パラメータ

`any_array`: 要素数を取得したいARRAYの値。

## 返り値

INTの値を返します。

## 例

```plain text
mysql> select array_length([1,2,3]);
+-----------------------+
| array_length([1,2,3]) |
+-----------------------+
|                     3 |
+-----------------------+
1 row in set (0.00 sec)

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
1 row in set (0.01 sec)
```

## キーワード

ARRAY_LENGTH, ARRAY, CARDINALITY
