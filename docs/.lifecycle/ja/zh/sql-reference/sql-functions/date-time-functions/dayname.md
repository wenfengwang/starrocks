---
displayed_sidebar: Chinese
---

# dayname

## 機能

指定された日付に対応する曜日名を返します。

引数は DATE または DATETIME 型です。

## 文法

```Haskell
VARCHAR DAYNAME(DATETIME|DATE date)
```

## 例

```Plain Text
select dayname('2007-02-03 00:00:00');
+--------------------------------+
| dayname('2007-02-03 00:00:00') |
+--------------------------------+
| Saturday                       |
+--------------------------------+
```
