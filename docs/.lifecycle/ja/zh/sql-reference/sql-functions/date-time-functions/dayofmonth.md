---
displayed_sidebar: Chinese
---

# dayofmonth

## 機能

日付から日にちの情報を取得し、返り値の範囲は1〜31です。

引数は DATE または DATETIME 型です。

## 文法

```Haskell
INT DAYOFMONTH(DATETIME date)
```

## 例

```Plain Text
select dayofmonth('1987-01-31');
+-----------------------------------+
| dayofmonth('1987-01-31 00:00:00') |
+-----------------------------------+
|                                31 |
+-----------------------------------+
```
