---
displayed_sidebar: "Japanese"
---

# hour

## 説明

与えられた日付の時を返します。返り値の範囲は0から23です。

`date`パラメータはDATEまたはDATETIME型である必要があります。

## 構文

```Haskell
INT HOUR(DATETIME|DATE date)
```

## 例

```Plain Text
MySQL > select hour('2018-12-31 23:59:59');
+-----------------------------+
| hour('2018-12-31 23:59:59') |
+-----------------------------+
|                          23 |
+-----------------------------+
```

## キーワード

HOUR