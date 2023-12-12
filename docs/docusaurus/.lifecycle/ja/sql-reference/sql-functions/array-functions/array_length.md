---
displayed_sidebar: "Japanese"
---

# array_length（配列の長さ）

## 説明

配列内の要素の数を返します。結果の型はINTです。入力パラメータがNULLの場合、結果もNULLになります。Null要素も長さに含まれます。

[cardinality()](cardinality.md)というエイリアスがあります。

## 構文

```Haskell
INT array_length(any_array)
```

## パラメータ

`any_array`: 要素の数を取得したいARRAYの値。

## 返り値

INT値を返します。

## 例

```plain text
mysql> select array_length([1,2,3]);
+-----------------------+
| array_length([1,2,3]) |
+-----------------------+
|                     3 |
+-----------------------+
1 行が返されました (0.00 秒)

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
1 行が返されました (0.01 秒)
```

## キーワード

ARRAY_LENGTH, ARRAY, CARDINALITY