---
displayed_sidebar: "Japanese"
---

# dayofmonth

## 説明

日付の日部分を取得し、1から31までの値を返します。

`date`パラメータはDATEまたはDATETIMEタイプでなければなりません。

## 構文

```Haskell
INT DAYOFMONTH(DATETIME date)
```

## 例

```Plain Text
MySQL > select dayofmonth('1987-01-31');
+-----------------------------------+
| dayofmonth('1987-01-31 00:00:00') |
+-----------------------------------+
|                                31 |
+-----------------------------------+
```

## キーワード

DAYOFMONTH