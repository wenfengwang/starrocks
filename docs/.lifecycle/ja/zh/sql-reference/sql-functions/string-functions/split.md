---
displayed_sidebar: Chinese
---

# split

## 機能

分割符号に基づいて文字列を分割し、分割された全ての文字列を ARRAY 形式で返します。

## 文法

```Haskell
split(content, delimiter)
```

## パラメータ説明

`content`: 対応するデータ型は VARCHAR です。

`delimiter`: 対応するデータ型は VARCHAR です。

## 戻り値の説明

戻り値のデータ型は ARRAY です。

## 例

```Plain Text
mysql> select split("a,b,c",",");
+---------------------+
| split('a,b,c', ',') |
+---------------------+
| ["a","b","c"]       |
+---------------------+
mysql> select split("a,b,c",",b,");
+-----------------------+
| split('a,b,c', ',b,') |
+-----------------------+
| ["a","c"]             |
+-----------------------+
mysql> select split("abc","");
+------------------+
| split('abc', '') |
+------------------+
| ["a","b","c"]    |
+------------------+
```
