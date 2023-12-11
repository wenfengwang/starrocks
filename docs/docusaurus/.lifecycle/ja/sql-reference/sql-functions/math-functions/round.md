---
displayed_sidebar: "Japanese"
---

# round, dround

## Description

指定された桁数に数値を丸めます。`n`が指定されていない場合、`x`は最も近い整数に丸められます。`n`が指定されている場合、`x`は`n`桁まで丸められます。`n`が負の場合、`x`は小数点の左側に丸められます。オーバーフローが発生した場合、エラーが返されます。

## Syntax

```Haskell
ROUND(x [,n]);
```

## Parameters

`x`: DOUBLEおよびDECIMAL128データ型をサポートしています。

`n`: INTデータ型をサポートしています。このパラメータはオプションです。

## Return value

`x`のみが指定されている場合、戻り値は以下のデータ型です：

["DECIMAL128"] -> "DECIMAL128"

["DOUBLE"] -> "BIGINT"

`x`および`n`の両方が指定されている場合、戻り値は以下のデータ型です：

["DECIMAL128", "INT"] -> "DECIMAL128"

["DOUBLE", "INT"] -> "DOUBLE"

## Examples

```Plain
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