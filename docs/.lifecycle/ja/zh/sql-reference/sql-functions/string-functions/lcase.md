---
displayed_sidebar: Chinese
---

# lcase

## 機能

この関数は `lower` と同じで、文字列を小文字に変換します。

## 文法

```Haskell
lcase(str)
```

## パラメータ説明

`str`: 対応するデータ型は VARCHAR です。

## 戻り値の説明

戻り値のデータ型は VARCHAR です。

## 例

```Plain Text
mysql> SELECT lcase("AbC123");
+-----------------+
|lcase('AbC123')  |
+-----------------+
|abc123           |
+-----------------+
```
