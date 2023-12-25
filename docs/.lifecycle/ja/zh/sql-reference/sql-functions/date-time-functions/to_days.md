---
displayed_sidebar: Chinese
---

# to_days

## 機能

指定された日付が `0000-01-01` から何日経過したかを返します。

引数は DATE または DATETIME 型でなければなりません。

## 文法

```Haskell
INT TO_DAYS(DATETIME date)
```

## 例

```Plain Text
select to_days('2007-10-07');
+-----------------------+
| to_days('2007-10-07') |
+-----------------------+
|                733321 |
+-----------------------+
```
