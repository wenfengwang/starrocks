---
displayed_sidebar: Chinese
---


# カウント

## 機能

条件を満たす行数を返します。

この関数には3つの形式があります：

- COUNT(*) はテーブルの全行数を返します。

- COUNT(expr) はある列の非NULL値の行数を返します。

- COUNT(DISTINCT expr) はある列の重複を除いた非NULL値の行数を返します。

COUNT DISTINCT は正確な重複排除に使用されますが、より良い重複排除性能が必要な場合は、[Bitmapを使用して正確な重複排除を実現する](../../../using_starrocks/Using_bitmap.md)を参照してください。

**バージョン2.4から、StarRocksは一つのクエリで複数のCOUNT(DISTINCT)を使用することをサポートしています。**

## 语法

```Haskell
COUNT(expr)
COUNT(DISTINCT expr [,expr,...])
```

## パラメータ説明

`expr`: 条件式。`expr`が列名の場合、列の値は任意のタイプをサポートします。

## 戻り値説明

戻り値は数値型です。一致する行がない場合は0を返します。NULL値はカウントに含まれません。

## 例

`test`というテーブルがあり、注文`id`ごとに国、商品カテゴリ、サプライヤー番号を表示します。

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

例1：テーブルにいくつの行があるかを確認します。

```Plain
select count(*) from test;
+----------+
| count(*) |
+----------+
|        7 |
+----------+
```

例2：注文`id`の数を確認します。

```Plain
select count(id) from test;
+-----------+
| count(id) |
+-----------+
|         7 |
+-----------+
```

例3：`category`の数を確認します。合計する際にNULL値は無視します。

```Plain
select count(category) from test;
  +-----------------+
  | count(category) |
  +-----------------+
  |               6 |
  +-----------------+
```

例4：DISTINCTを使用して重複を排除し、`category`の数を確認します。

```Plain
select count(distinct category) from test;
+-------------------------+
| count(DISTINCT category) |
+-------------------------+
|                       4 |
+-------------------------+
```

例5：商品カテゴリ(`category`)とサプライヤー(`supplier`)の異なる組み合わせがいくつあるかを確認します。

```Plain
select count(distinct category, supplier) from test;
+------------------------------------+
| count(DISTINCT category, supplier) |
+------------------------------------+
|                                  5 |
+------------------------------------+
```

上記の結果では、`id`が1004の組み合わせは`id`が1002の組み合わせと重複しているため、1回だけカウントされます。`id`が1007の組み合わせにはNULL値が含まれているため、カウントされません。

例6：一つのクエリで複数のCOUNT(DISTINCT)を使用します。

```Plain
select count(distinct country, category), count(distinct country,supplier) from test;
+-----------------------------------+-----------------------------------+
| count(DISTINCT country, category) | count(DISTINCT country, supplier) |
+-----------------------------------+-----------------------------------+
|                                 6 |                                 7 |
+-----------------------------------+-----------------------------------+
```
