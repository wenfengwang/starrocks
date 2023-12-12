---
displayed_sidebar: "Japanese"
---

# count（カウント）

## 説明

式で指定された行の合計数を返します。

この関数には、3つのバリエーションがあります。

- `COUNT(*)`は、NULL値を含んでいるかどうかに関係なく、テーブル内のすべての行をカウントします。

- `COUNT(expr)`は、特定の列内のNULLでない値の行数を数えます。

- `COUNT(DISTINCT expr)`は、列内の異なるNULLでない値の数を数えます。

`COUNT(DISTINCT expr)` は、正確な重複カウントに使用されます。より高速な重複カウントパフォーマンスが必要な場合は、[ビットマップを使用した正確な重複カウント](../../../using_starrocks/Using_bitmap.md)を参照してください。

StarRocks 2.4以降、1つのステートメント内で複数の`COUNT(DISTINCT)`を使用できます。

## 構文

~~~Haskell
COUNT(expr)
COUNT(DISTINCT expr [,expr,...])`
~~~

## パラメータ

`expr`: `count()` が実行される列または式。`expr` が列名の場合、その列は任意のデータ型であることができます。

## 戻り値

数値を返します。行が見つからない場合、0が返されます。この関数はNULL値を無視します。

## 例

テーブル名が `test` の場合、`id` で注文の国、カテゴリ、およびサプライヤーをクエリーします。

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

例1：テーブル `test` の行数をカウントします。

~~~Plain
    select count(*) from test;
    +----------+
    | count(*) |
    +----------+
    |        7 |
    +----------+
~~~

例2：`id` 列内の値の数をカウントします。

~~~Plain
    select count(id) from test;
    +-----------+
    | count(id) |
    +-----------+
    |         7 |
    +-----------+
~~~

例3：`category` 列内の値の数をカウントしますが、NULL値は無視します。

~~~Plain
select count(category) from test;
  +-----------------+
  | count(category) |
  +-----------------+
  |         6       |
  +-----------------+
~~~

例4：`category` 列の異なる値の数をカウントします。

~~~Plain
select count(distinct category) from test;
+-------------------------+
| count(DISTINCT category) |
+-------------------------+
|                       4 |
+-------------------------+
~~~

例5：`category` と `supplier` によって形成される組み合わせの数をカウントします。

~~~Plain
select count(distinct category, supplier) from test;
+------------------------------------+
| count(DISTINCT category, supplier) |
+------------------------------------+
|                                  5 |
+------------------------------------+
~~~

出力では、`id` 1004の組み合わせが`id` 1002の組み合わせと重複しています。それらは1度だけ数えられます。`id` 1007の組み合わせはNULL値を持っているため、数えられません。

例6：1つのステートメント内で複数の `COUNT(DISTINCT)` を使用します。

~~~Plain
select count(distinct country, category), count(distinct country,supplier) from test;
+-----------------------------------+-----------------------------------+
| count(DISTINCT country, category) | count(DISTINCT country, supplier) |
+-----------------------------------+-----------------------------------+
|                                 6 |                                 7 |
+-----------------------------------+-----------------------------------+
~~~