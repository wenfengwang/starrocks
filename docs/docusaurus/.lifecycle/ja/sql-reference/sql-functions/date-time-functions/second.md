---
displayed_sidebar: "Japanese"
---

# second

## 説明

指定された日付の2番目の部分を返します。返り値の範囲は0から59までです。

`date`パラメータはDATEまたはDATETIMEタイプである必要があります。

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