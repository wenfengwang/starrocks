---
displayed_sidebar: Chinese
---

# length

## 機能

文字列の **バイト** 長を返します。

## 文法

```Haskell
length(str)
```

## パラメータ説明

`str`: 対応するデータ型は VARCHAR です。

## 戻り値の説明

戻り値のデータ型は INT です。

## 例

```Plain Text
MySQL > select length("abc");
+---------------+
| length('abc') |
+---------------+
|             3 |
+---------------+

MySQL > select length("中国");
+------------------+
| length('中国')   |
+------------------+
|                6 |
+------------------+
```
