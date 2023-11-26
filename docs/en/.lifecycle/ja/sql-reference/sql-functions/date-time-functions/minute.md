---
displayed_sidebar: "Japanese"
---

# 分

## 説明

指定された日付の分を返します。返される値の範囲は0から59までです。

`date` パラメータは DATE 型または DATETIME 型である必要があります。

## 構文

```Haskell
INT MINUTE(DATETIME|DATE date)
```

## 例

```Plain Text
MySQL > select minute('2018-12-31 23:59:59');
+-----------------------------+
|minute('2018-12-31 23:59:59')|
+-----------------------------+
|                          59 |
+-----------------------------+
```

## キーワード

MINUTE
