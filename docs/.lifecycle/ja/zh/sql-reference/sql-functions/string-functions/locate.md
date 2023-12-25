---
displayed_sidebar: Chinese
---

# locate

## 機能

`substr`が`str`内に現れる位置を返します（1から数え始め、文字単位で計算）。第3引数`pos`が指定されている場合、`pos`インデックスから始まる文字列で`substr`が最初に現れる位置を検索し、見つからない場合は0を返します。

## 文法

```Haskell
locate(substr, str, pos)
```

## 引数説明

`substr`: 対応するデータ型はVARCHARです。

`str`: 対応するデータ型はVARCHARです。

`pos`: オプションの引数で、対応するデータ型はINTです。

## 戻り値の説明

戻り値のデータ型はINTです。

## 例

```Plain Text
MySQL > SELECT LOCATE('bar', 'foobarbar');
+----------------------------+
| locate('bar', 'foobarbar') |
+----------------------------+
|                          4 |
+----------------------------+

MySQL > SELECT LOCATE('xbar', 'foobar');
+--------------------------+
| locate('xbar', 'foobar') |
+--------------------------+
|                        0 |
+--------------------------+

MySQL > SELECT LOCATE('bar', 'foobarbar', 5);
+-------------------------------+
| locate('bar', 'foobarbar', 5) |
+-------------------------------+
|                             7 |
+-------------------------------+
```
