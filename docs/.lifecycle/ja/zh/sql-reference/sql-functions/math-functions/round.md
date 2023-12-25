---
displayed_sidebar: Chinese
---

# round, dround

## 機能

`x` のみが存在する場合、ROUND は `x` を最も近い整数に丸めます。`n` も存在する場合、ROUND は `x` を小数点以下 `n` 桁で丸めます。`n` が負の数であれば、ROUND は小数点を切り捨てて整数化し、中間値は 0 から遠ざかる方向に丸められます。オーバーフローが発生した場合は、エラーが生成されます。

## 文法

```Haskell
ROUND(x [,n]);
```

## パラメータ説明

`x`: 対応するデータ型は DOUBLE または DECIMAL128。

`n`: オプションで、対応するデータ型は INT。

## 戻り値の説明

`x` のみを指定した場合、戻り値のデータ型は以下の通りです：

["DECIMAL128"] -> "DECIMAL128"

["DOUBLE"] -> "BIGINT"

`x` と `n` の両方を指定した場合、戻り値のデータ型は以下の通りです：

["DECIMAL128", "INT"] -> "DECIMAL128"

["DOUBLE", "INT"] -> "DOUBLE"

## 例

```Plain Text
mysql> select round(3.14);
+-------------+
| round(3.14) |
+-------------+
|           3 |
+-------------+
1 row in set (0.00 sec)

mysql> select round(3.14,1);
+----------------+
| round(3.14, 1) |
+----------------+
|            3.1 |
+----------------+
1 row in set (0.00 sec)

mysql> select round(13.14,-1);
+------------------+
| round(13.14, -1) |
+------------------+
|               10 |
+------------------+
1 row in set (0.00 sec)
```
