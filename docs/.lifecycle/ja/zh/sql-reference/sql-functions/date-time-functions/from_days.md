---
displayed_sidebar: Chinese
---

# from_days

## 機能

`0000-01-01` からの日数を計算して、どの日かを求めます。

## 文法

```Haskell
DATE FROM_DAYS(INT N)
```

## 例

```Plain Text
select from_days(730669);
+-------------------+
| from_days(730669) |
+-------------------+
| 2000-07-03        |
+-------------------+
```
