---
displayed_sidebar: Chinese
---

# lower

## 機能

引数に含まれるすべての文字列を小文字に変換します。

## 文法

```Haskell
lower(str)
```

## 引数説明

`str`: 対応するデータ型は VARCHAR です。

## 文法説明

戻り値のデータ型は VARCHAR です。

## 例

```Plain Text
MySQL > SELECT lower("AbC123");
+-----------------+
| lower('AbC123') |
+-----------------+
| abc123          |
+-----------------+
```
