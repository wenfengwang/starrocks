---
displayed_sidebar: "Japanese"
---

# second

## 説明

指定された日付の秒を返します。 返される値の範囲は0から59です。

`date`パラメータはDATEまたはDATETIME型でなければなりません。

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