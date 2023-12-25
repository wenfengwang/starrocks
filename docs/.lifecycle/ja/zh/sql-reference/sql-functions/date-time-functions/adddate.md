---
displayed_sidebar: Chinese
---

# adddate、days_add

## 機能

特定の時間間隔を日付に追加します。

## 構文

```Haskell
DATETIME ADDDATE(DATETIME|DATE date, INTERVAL expr type)
```

## パラメータ説明

* `date`：有効な日付表現でなければなりません。DATETIME または DATE 型が可能です。

* `expr`：追加する時間間隔で、サポートされるデータ型は INT です。

* `type`：時間間隔の単位で、YEAR、MONTH、DAY、HOUR、MINUTE、または SECOND が可能です。

## 戻り値の説明

DATETIME 型の値を返します。入力値が空または形式が正しくない場合は、NULL を返します。

## 例

```Plain Text
select adddate('2010-11-30 23:59:59', INTERVAL 2 DAY);
+-------------------------------------------------+
| adddate('2010-11-30 23:59:59', INTERVAL 2 DAY) |
+-------------------------------------------------+
| 2010-12-02 23:59:59                             |
+-------------------------------------------------+

select adddate('2010-11-30', INTERVAL 2 DAY);
+----------------------------------------+
| adddate('2010-11-30', INTERVAL 2 DAY)  |
+----------------------------------------+
| 2010-12-02 00:00:00                    |
+----------------------------------------+
1 row in set (0.01 sec)

```
