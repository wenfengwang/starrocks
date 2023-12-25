---
displayed_sidebar: Chinese
---

# char_length

## 機能

文字列の長さを返します。
マルチバイト文字に対しては、**文字** の数を返します。現在は UTF-8 エンコーディングのみをサポートしており、この関数には `character_length` という別名もあります。

## 文法

```Haskell
char_length(str)
```

## パラメータ説明

`str`: 対応するデータ型は VARCHAR です。

## 戻り値の説明

戻り値のデータ型は INT です。

## 例

```Plain Text
MySQL > select char_length("abc");
+--------------------+
| char_length('abc') |
+--------------------+
|                  3 |
+--------------------+

MySQL > select char_length("中国");
+----------------------+
| char_length('中国')  |
+----------------------+
|                    2 |
+----------------------+
```
