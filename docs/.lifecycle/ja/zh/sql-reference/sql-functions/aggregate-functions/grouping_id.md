---
displayed_sidebar: Chinese
---


# grouping_id

## 機能

同じグループ基準の集計結果を区別するために使用されます。

## 文法

```Haskell
GROUPING_ID(expr)
```

## パラメータ説明

`expr`: 条件式で、その結果はBIGINT型である必要があります。

## 戻り値の説明

BIGINT型の値を返します。

## 例

```Plain
MySQL > SELECT COL1, GROUPING_ID(COL2) AS 'GroupingID' FROM tbl GROUP BY ROLLUP (COL1, COL2);
+------+------------+
| COL1 | GroupingID |
+------+------------+
| NULL |          1 |
| 2.20 |          1 |
| 2.20 |          0 |
| 1.10 |          1 |
| 1.10 |          0 |
+------+------------+
```
