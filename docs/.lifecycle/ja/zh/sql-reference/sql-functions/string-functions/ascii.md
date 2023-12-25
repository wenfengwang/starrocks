---
displayed_sidebar: Chinese
---

# ascii

## 機能

文字列の最初の文字に対応するASCIIコードを返します。

## 文法

```Haskell
ascii(str)
```

## 引数説明

`str`: サポートされるデータ型はVARCHARです。

## 戻り値の説明

戻り値のデータ型はINTです。

## 例

```Plain Text
MySQL > select ascii('1');
+------------+
| ascii('1') |
+------------+
|         49 |
+------------+

MySQL > select ascii('234');
+--------------+
| ascii('234') |
+--------------+
|           50 |
+--------------+
```
