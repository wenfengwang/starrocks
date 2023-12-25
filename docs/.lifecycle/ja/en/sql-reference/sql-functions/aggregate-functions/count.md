---
displayed_sidebar: English
---


# count

## 説明

指定された式によって特定される行の合計数を返します。

この関数には3つのバリエーションがあります：

- `COUNT(*)` は、NULL値が含まれているかどうかに関わらず、テーブル内の全ての行をカウントします。

- `COUNT(expr)` は、特定の列でNULLでない値を持つ行の数をカウントします。

- `COUNT(DISTINCT expr)` は、列内の異なるNULLでない値の数をカウントします。

`COUNT(DISTINCT expr)` は正確なカウントディスティンクトに使用されます。より高いカウントディスティンクトパフォーマンスが必要な場合は、[Use bitmap for exact count distinct](../../../using_starrocks/Using_bitmap.md)を参照してください。

StarRocks 2.4以降、1つのステートメントで複数のCOUNT(DISTINCT)を使用できます。

## 構文

~~~Haskell
COUNT(expr)
COUNT(DISTINCT expr [,expr,...])
~~~

## パラメータ

`expr`: `count()`が実行される基となる列または式です。`expr`が列名である場合、その列は任意のデータ型であることができます。

## 戻り値

数値を返します。行が見つからない場合は0が返されます。この関数はNULL値を無視します。

## 例

`test`という名前のテーブルがあるとします。`id`によって各注文の国、カテゴリ、およびサプライヤーをクエリします。

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

例1: `test`テーブルの行数をカウントします。

~~~Plain
    select count(*) from test;
    +----------+
    | count(*) |
    +----------+
    |        7 |
    +----------+
~~~

例2: `id`列の値の数をカウントします。

~~~Plain
    select count(id) from test;
    +-----------+
    | count(id) |
    +-----------+
    |         7 |
    +-----------+
~~~

例3: NULL値を無視して、`category`列の値の数をカウントします。

~~~Plain
select count(category) from test;
  +-----------------+
  | count(category) |
  +-----------------+
  |               6 |
  +-----------------+
~~~

例4: `category`列の異なる値の数をカウントします。

~~~Plain
select count(distinct category) from test;
+-------------------------+
| count(DISTINCT category) |
+-------------------------+
|                         4 |
+-------------------------+
~~~

例5: `category`と`supplier`によって形成される組み合わせの数をカウントします。

~~~Plain
select count(distinct category, supplier) from test;
+------------------------------------+
| count(DISTINCT category, supplier) |
+------------------------------------+
|                                    5 |
+------------------------------------+
~~~

出力では、`id` 1004の組み合わせが`id` 1002の組み合わせと重複しています。これらは一度だけカウントされます。`id` 1007の組み合わせはNULL値を持ち、カウントされません。

例6: 1つのステートメントで複数のCOUNT(DISTINCT)を使用します。

~~~Plain
select count(distinct country, category), count(distinct country,supplier) from test;
+-----------------------------------+-----------------------------------+
| count(DISTINCT country, category) | count(DISTINCT country, supplier) |
+-----------------------------------+-----------------------------------+
|                                 6 |                                 7 |
+-----------------------------------+-----------------------------------+
~~~
