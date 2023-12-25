---
displayed_sidebar: Chinese
---

# to_date

## 機能

DATETIME 型の値から日付部分を返します。

## 文法

```Haskell
DATE TO_DATE(DATETIME datetime)
```

## 例

```Plain Text
select to_date("2020-02-02 00:00:00");
+--------------------------------+
| to_date('2020-02-02 00:00:00') |
+--------------------------------+
| 2020-02-02                     |
+--------------------------------+
```
