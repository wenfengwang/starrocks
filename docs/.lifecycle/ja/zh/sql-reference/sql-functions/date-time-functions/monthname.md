---
displayed_sidebar: Chinese
---

# monthname

## 機能

指定された日付に対応する月の名前を返します。引数は DATE または DATETIME 型です。

日付が存在しない場合は、NULL を返します。

## 文法

```Haskell
VARCHAR MONTHNAME(DATETIME|DATE date)
```

## 例

```Plain Text
select monthname('2008-02-03 00:00:00');
+----------------------------------+
| monthname('2008-02-03 00:00:00') |
+----------------------------------+
| February                         |
+----------------------------------+

select monthname('2008-02-03');
+-------------------------+
| monthname('2008-02-03') |
+-------------------------+
| February                |
+-------------------------+
```
