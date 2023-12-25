---
displayed_sidebar: Chinese
---

# weekofyear

## 機能

指定された日時がその年の何週目かを計算します。

## 構文

```Haskell
INT WEEKOFYEAR(DATETIME date)
```

## 引数説明

引数は DATE または DATETIME 型です。

## 戻り値の説明

INT 型の値を返します。

## 例

```Plain Text
select weekofyear('2008-02-20 00:00:00');
+-----------------------------------+
| weekofyear('2008-02-20 00:00:00') |
+-----------------------------------+
|                                 8 |
+-----------------------------------+
```
