---
displayed_sidebar: English
---

# to_days

## 説明

0000-01-01から特定の日付までの日数を返します。

`date` パラメータは DATE 型または DATETIME 型でなければなりません。

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

TO_DAYS,TO,DAYS
