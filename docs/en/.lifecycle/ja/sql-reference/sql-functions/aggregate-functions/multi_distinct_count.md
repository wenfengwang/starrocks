---
displayed_sidebar: "Japanese"
---

# multi_distinct_count

## 説明

`expr`の総行数を返します。これはcount(distinct expr)と同等です。

## 構文

```Haskell
multi_distinct_count(expr)
```

## パラメータ

`expr`: `multi_distinct_count()`を実行する基準となる列または式です。`expr`が列名の場合、列は任意のデータ型であることができます。

## 戻り値

数値を返します。行が見つからない場合は0が返されます。この関数はNULL値を無視します。

## 例

`test`というテーブルがあるとします。`id`で各注文のカテゴリとサプライヤーをクエリします。

~~~Plain
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
~~~

例1: `category`列の重複値の数をカウントします。

~~~Plain
select multi_distinct_count(category) from test;
+--------------------------------+
| multi_distinct_count(category) |
+--------------------------------+
|                              4 |
+--------------------------------+
~~~

例2: `supplier`列の重複値の数をカウントします。

~~~Plain
select multi_distinct_count(supplier) from test;
+--------------------------------+
| multi_distinct_count(supplier) |
+--------------------------------+
|                              5 |
+--------------------------------+
~~~
