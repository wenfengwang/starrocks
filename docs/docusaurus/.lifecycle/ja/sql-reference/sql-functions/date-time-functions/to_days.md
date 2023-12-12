---
displayed_sidebar: "Japanese"
---

# to_days

## Description

0000-01-01との日数を返します。

`date`パラメータはDATEかDATETIME型でなければなりません。

## Syntax

```Haskell
INT TO_DAYS(DATETIME date)
```

## Examples

```Plain Text
MySQL > select to_days('2007-10-07');
+-----------------------+
| to_days('2007-10-07') |
+-----------------------+
|                733321 |
+-----------------------+
```

## keyword

TO_DAYS,TO,DAYS