---
displayed_sidebar: English
---

# bitmap_from_binary

## 説明

特定のフォーマットのバイナリ文字列をビットマップに変換します。

この関数は、ビットマップデータをStarRocksにロードするために使用できます。

この関数はv3.0からサポートされています。

## 構文

```Haskell
BITMAP bitmap_from_binary(VARBINARY str)
```

## パラメーター

`str`: サポートされるデータ型はVARBINARYです。

## 戻り値

BITMAP型の値を返します。

## 例

例1: この関数を他のBitmap関数と組み合わせて使用します。

```Plain
mysql> select bitmap_to_string(bitmap_from_binary(bitmap_to_binary(bitmap_from_string("0,1,2,3"))));
+---------------------------------------------------------------------------------------+
| bitmap_to_string(bitmap_from_binary(bitmap_to_binary(bitmap_from_string('0,1,2,3')))) |
+---------------------------------------------------------------------------------------+
| 0,1,2,3                                                                               |
+---------------------------------------------------------------------------------------+
1 row in set (0.01 sec)
```
