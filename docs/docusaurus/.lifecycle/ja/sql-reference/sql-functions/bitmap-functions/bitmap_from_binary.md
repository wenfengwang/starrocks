```yaml
---
displayed_sidebar: "Japanese"
---

# bitmap_from_binary

## 説明

特定の形式でバイナリ文字列をビットマップに変換します。

この機能はStarRocksにビットマップデータをロードするために使用できます。

この機能はv3.0からサポートされています。

## 構文

```Haskell
BITMAP bitmap_from_binary(VARBINARY str)
```

## パラメータ

`str`：サポートされているデータ型はVARBINARYです。

## 戻り値

BITMAP型の値を返します。

## 例

例1： 他のビットマップ機能とともにこの関数を使用します。

```Plain
mysql> select bitmap_to_string(bitmap_from_binary(bitmap_to_binary(bitmap_from_string("0,1,2,3"))));
+---------------------------------------------------------------------------------------+
| bitmap_to_string(bitmap_from_binary(bitmap_to_binary(bitmap_from_string('0,1,2,3')))) |
+---------------------------------------------------------------------------------------+
| 0,1,2,3                                                                               |
+---------------------------------------------------------------------------------------+
1 row in set (0.01 sec)
```
```