---
displayed_sidebar: Chinese
---

# bitmap_from_binary

## 機能

特定のフォーマットのVarbinary型の文字列をBitmapに変換します。StarRocksへのBitmapデータのインポートに使用できます。

この関数はバージョン3.0からサポートされています。

## 文法

```Haskell
BITMAP bitmap_from_binary(VARBINARY str)
```

## パラメータ説明

`str`：サポートされるデータタイプはVARBINARYです。

## 戻り値の説明

BITMAPタイプのデータを返します。

## 例

例1: この関数は他のBitmap関数と組み合わせて使用します。

   ```Plain
   mysql> select bitmap_to_string(bitmap_from_binary(bitmap_to_binary(bitmap_from_string("0,1,2,3"))));
   +---------------------------------------------------------------------------------------+
   | bitmap_to_string(bitmap_from_binary(bitmap_to_binary(bitmap_from_string('0,1,2,3')))) |
   +---------------------------------------------------------------------------------------+
   | 0,1,2,3                                                                               |
   +---------------------------------------------------------------------------------------+
   1行がセットされました (0.01秒)
   ```
