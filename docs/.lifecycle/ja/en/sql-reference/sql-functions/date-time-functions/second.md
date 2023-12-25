---
displayed_sidebar: English
---

# second

## 説明

指定された日付の秒部分を返します。戻り値の範囲は 0 から 59 です。

`date` パラメータは DATE 型または DATETIME 型である必要があります。

## 構文

```Haskell
INT SECOND(DATETIME date)
```

## 例

```Plain Text
MySQL > select second('2018-12-31 23:59:59');
+-----------------------------+
|second('2018-12-31 23:59:59')|
+-----------------------------+
|                          59 |
+-----------------------------+
```

## キーワード

SECOND
