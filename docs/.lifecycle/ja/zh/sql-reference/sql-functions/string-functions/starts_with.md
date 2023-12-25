---
displayed_sidebar: Chinese
---

# starts_with

## 機能

文字列が指定された接頭辞で始まる場合は 1 を返し、そうでない場合は 0 を返します。任意の引数が NULL の場合は NULL を返します。

## 文法

```Haskell
starts_with(str, prefix)
```

## 引数説明

`str`: 対応するデータ型は VARCHAR です。

`prefix`: 対応するデータ型は VARCHAR です。

## 戻り値の説明

戻り値のデータ型は BOOLEAN です。

## 例

```Plain Text
mysql> select starts_with("hello world","hello");
+-------------------------------------+
|starts_with('hello world', 'hello')  |
+-------------------------------------+
| 1                                   |
+-------------------------------------+

mysql> select starts_with("hello world","world");
+-------------------------------------+
|starts_with('hello world', 'world')  |
+-------------------------------------+
| 0                                   |
+-------------------------------------+
```
