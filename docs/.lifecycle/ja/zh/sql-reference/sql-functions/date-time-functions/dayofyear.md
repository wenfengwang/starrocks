---
displayed_sidebar: Chinese
---

# dayofyear

## 機能

指定された日付がその年の何日目かを計算します。

引数は DATE または DATETIME 型です。

## 文法

```Haskell
INT DAYOFYEAR(DATETIME date)
```

## 例

```Plain Text
select dayofyear('2007-02-03 00:00:00');
+----------------------------------+
| dayofyear('2007-02-03 00:00:00') |
+----------------------------------+
|                               34 |
+----------------------------------+
```
