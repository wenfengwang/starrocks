---
displayed_sidebar: Chinese
---

# char

## 機能

入力されたASCII値に対応する文字を返します。

## 文法

```Haskell
char(x);
```

## 引数説明

`x`: サポートされるデータ型はINTです。

## 戻り値の説明

戻り値のデータ型はVARCHARです。

## 例

```Plain Text
mysql> SELECT CHAR(77);
+----------+
| char(77) |
+----------+
| M        |
+----------+
1行がセットされました (0.00秒)
```
