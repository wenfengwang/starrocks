---
displayed_sidebar: Chinese
---

# multi_distinct_count

## 機能

`expr` から重複する値を除外した後の行数を返します。COUNT(DISTINCT expr) と同等の機能です。

## 文法

```Haskell
multi_distinct_count(expr)
```

## パラメータ説明

`expr`: 条件式。`expr` が列名の場合、列の値は任意のタイプをサポートします。

## 戻り値の説明

戻り値は数値型です。一致する行がない場合は 0 を返します。NULL 値は統計に含まれません。

## 例

`test` テーブルがあり、注文 `id` に従って、各注文の国、商品カテゴリ、サプライヤー番号を表示するとします。

```Plain
select * from test order by id;
+------+----------+----------+------------+
| id   | country  | category | supplier   |
+------+----------+----------+------------+
| 1001 | US       | A        | supplier_1 |
| 1002 | Thailand | A        | supplier_2 |
| 1003 | Turkey   | B        | supplier_3 |
| 1004 | US       | A        | supplier_2 |
| 1005 | China    | C        | supplier_4 |
| 1006 | Japan    | D        | supplier_3 |
| 1007 | Japan    | NULL     | supplier_5 |
+------+----------+----------+------------+
```

例1：multi_distinct_count を使用して重複を除外し、`category` の数を確認します。

```Plain
select multi_distinct_count(category) from test;
+--------------------------------+
| multi_distinct_count(category) |
+--------------------------------+
|                              4 |
+--------------------------------+
```

例2：multi_distinct_count を使用して重複を除外し、サプライヤー `supplier` の数を確認します。

```Plain
select multi_distinct_count(supplier) from test;
+--------------------------------+
| multi_distinct_count(supplier) |
+--------------------------------+
|                              5 |
+--------------------------------+
```
