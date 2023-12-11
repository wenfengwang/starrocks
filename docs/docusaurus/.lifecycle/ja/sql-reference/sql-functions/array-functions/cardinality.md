---
displayed_sidebar: "Japanese"
---

# カーディナリティ

## 説明

配列内の要素数を返します。結果の型はINTです。入力パラメータがNULLの場合、結果もNULLです。NULL要素も長さに含まれます。

これは、[array_length()](array_length.md) のエイリアスです。

この機能はv3.0以降でサポートされています。

## 構文

```Haskell
INT cardinality(any_array)
```

## パラメータ

`any_array`: 要素数を取得したいARRAY値です。

## 戻り値

INT値を返します。

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