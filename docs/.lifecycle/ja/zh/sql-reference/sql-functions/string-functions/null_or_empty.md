---
displayed_sidebar: Chinese
---

# null_or_empty

## 機能

文字列が空文字またはNULLの場合はtrueを返し、そうでない場合はfalseを返します。

## 文法

```Haskell
NULL_OR_EMPTY(str)
```

## パラメータ説明

`str`: 対応するデータ型はVARCHARです。

## 戻り値の説明

戻り値のデータ型はBOOLEANです。

## 例

```Plain Text
MySQL > select null_or_empty(null);
+---------------------+
| null_or_empty(NULL) |
+---------------------+
|                   1 |
+---------------------+

MySQL > select null_or_empty("");
+-------------------+
| null_or_empty('') |
+-------------------+
|                 1 |
+-------------------+

MySQL > select null_or_empty("a");
+--------------------+
| null_or_empty('a') |
+--------------------+
|                  0 |
+--------------------+
```
