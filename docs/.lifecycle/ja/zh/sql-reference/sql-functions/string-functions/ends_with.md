---
displayed_sidebar: Chinese
---

# ends_with

## 機能

文字列が指定された接尾辞で終わる場合は true を返し、そうでない場合は false を返します。いずれかの引数が NULL の場合は NULL を返します。

## 文法

```Haskell
ENDS_WITH(str, suffix)
```

## 引数説明

`str`: 対応するデータ型は VARCHAR です。

`suffix`: 対応するデータ型は VARCHAR です。

## 戻り値の説明

戻り値のデータ型は BOOLEAN です。

## 例

```Plain Text
MySQL > select ends_with("Hello starrocks", "starrocks");
+-----------------------------------+
| ends_with('Hello starrocks', 'starrocks') |
+-----------------------------------+
|                                 1 |
+-----------------------------------+

MySQL > select ends_with("Hello starrocks", "Hello");
+-----------------------------------+
| ends_with('Hello starrocks', 'Hello') |
+-----------------------------------+
|                                 0 |
+-----------------------------------+
```
