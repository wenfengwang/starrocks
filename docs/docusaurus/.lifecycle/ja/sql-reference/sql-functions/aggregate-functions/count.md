---
displayed_sidebar: "Japanese"
---

# count（カウント）

## 説明

指定された式によって指定された行の総数を返します。

この関数には3つのバリエーションがあります：

- `COUNT(*)`は、NULL値を含んでいても、テーブル内のすべての行を数えます。

- `COUNT(expr)`は、特定の列にNULL以外の値を持つ行の数を数えます。

- `COUNT(DISTINCT expr)`は、列内の重複しないNULL以外の値の数を数えます。

`COUNT(DISTINCT expr)`は、正確な重複数を数えるために使用されます。より高速な重複数のパフォーマンスが必要な場合は、[正確な重複数のためにビットマップを使用](../../../using_starrocks/Using_bitmap.md)してください。

StarRocks 2.4以降では、1つのステートメントで複数のCOUNT(DISTINCT)を使用できます。

## 構文

~~~Haskell
COUNT(expr)
COUNT(DISTINCT expr [,expr,...])`
~~~

## パラメータ

`expr`: `count()`が実行される基になる列または式。`expr`が列名である場合、その列は任意のデータ型であることができます。

## 戻り値

数値を返します。行が見つからない場合、0が返されます。この関数はNULLの値を無視します。

## 例

`test`という名前のテーブルがあるとします。`id`で各注文の国、カテゴリ、およびサプライヤーをクエリします。

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

例1：テーブル`test`の行数を数えます。

~~~Plain
    select count(*) from test;
    +----------+
    | count(*) |
    +----------+
    |        7 |
    +----------+
~~~

例2：`id`列の値の数を数えます。

~~~Plain
    select count(id) from test;
    +-----------+
    | count(id) |
    +-----------+
    |         7 |
    +-----------+
~~~

例3：NULLの値を無視して`category`列の値の数を数えます。

~~~Plain
select count(category) from test;
  +-----------------+
  | count(category) |
  +-----------------+
  |         6       |
  +-----------------+
~~~

例4：`category`列の重複しない値の数を数えます。

~~~Plain
select count(distinct category) from test;
+-------------------------+
| count(DISTINCT category) |
+-------------------------+
|                       4 |
+-------------------------+
~~~

例5：`category`と`supplier`によって形成される組み合わせの数を数えます。

~~~Plain
select count(distinct category, supplier) from test;
+------------------------------------+
| count(DISTINCT category, supplier) |
+------------------------------------+
|                                  5 |
+------------------------------------+
~~~

出力では、`id`が1004の組み合わせと`id`が1002の組み合わせが重複しています。それらは1度だけ数えられます。`id`が1007の組み合わせにはNULL値が含まれており、数えられません。

例6：1つのステートメントで複数のCOUNT(DISTINCT)を使用します。

~~~Plain
select count(distinct country, category), count(distinct country,supplier) from test;
+-----------------------------------+-----------------------------------+
| count(DISTINCT country, category) | count(DISTINCT country, supplier) |
+-----------------------------------+-----------------------------------+
|                                 6 |                                 7 |
+-----------------------------------+-----------------------------------+
~~~