---
displayed_sidebar: "Japanese"
---

# year

## 説明

日付の年部分を返し、1000から9999までの値を返します。

`date` パラメータは DATE もしくは DATETIME 型である必要があります。

## 構文

```Haskell
INT YEAR(DATETIME date)
```

## 例

```Plain Text
MySQL > select year('1987-01-01');
+-----------------------------+
| year('1987-01-01 00:00:00') |
+-----------------------------+
|                        1987 |
+-----------------------------+
```

## キーワード

YEAR
