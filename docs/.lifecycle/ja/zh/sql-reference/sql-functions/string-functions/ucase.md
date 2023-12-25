---
displayed_sidebar: Chinese
---

# ucase

## 機能

この関数は `upper` と同じで、文字列を大文字に変換します。

## 文法

```Haskell
ucase(str)
```

## パラメータ説明

`str`: 対応するデータ型は VARCHAR です。

## 戻り値の説明

戻り値のデータ型は VARCHAR です。

## 例

```Plain Text
mysql> SELECT ucase("AbC123");
+-----------------+
|ucase('AbC123')  |
+-----------------+
|ABC123           |
+-----------------+
```
