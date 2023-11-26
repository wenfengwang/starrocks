---
displayed_sidebar: "Japanese"
---

# to_days

## 説明

日付と0000-01-01の間の日数を返します。

`date`パラメータはDATEまたはDATETIME型である必要があります。

## 構文

```Haskell
INT TO_DAYS(DATETIME date)
```

## 例

```Plain Text
MySQL > select to_days('2007-10-07');
+-----------------------+
| to_days('2007-10-07') |
+-----------------------+
|                733321 |
+-----------------------+
```

## キーワード

TO_DAYS, TO, DAYS
