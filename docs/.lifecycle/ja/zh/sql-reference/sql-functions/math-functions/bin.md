---
displayed_sidebar: Chinese
---

# bin

## 機能

引数`x`を二進数に変換します。

## 文法

```Haskell
BIN(x);
```

## 引数説明

`x`: サポートされるデータ型はBIGINTです。

## 戻り値の説明

戻り値のデータ型はVARCHARです。

## 例

```Plain Text
mysql>select bin(3);
+--------+
| bin(3) |
+--------+
| 11     |
+--------+
1行がセットされました (0.02秒)
```
