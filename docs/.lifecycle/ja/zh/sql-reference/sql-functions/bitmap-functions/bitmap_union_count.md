---
displayed_sidebar: Chinese
---

# bitmap_union_count

## 機能

入力されたビットマップ値の集合の和集合を計算し、その和集合の基数を返します。この関数はバージョン2.3からサポートされています。

## 文法

```Haskell
BITMAP_UNION_COUNT(value)
```

## パラメータ説明

- `value` ：入力されたビットマップ値の集合で、サポートされるデータ型は BITMAP です。

## 戻り値の説明

戻り値のデータ型は BIGINT です。

## 例

この関数を使用してウェブページの UV データを計算します。`user_id` フィールドの型が INT と仮定すると、以下の2つのクエリは等価です。

```sql
mysql> select * from test
+---------+---------+
| page_id | user_id |
+---------+---------+
|       1 |       1 |
|       1 |       2 |
|       2 |       1 |
+---------+---------+

mysql> select page_id,count(distinct user_id) from test group by page_id;
+---------+-------------------------+
| page_id | count(DISTINCT user_id) |
+---------+-------------------------+
|       1 |                       2 |
|       2 |                       1 |
+---------+-------------------------+

mysql> select page_id,bitmap_union_count(to_bitmap(user_id)) from test group by page_id;
+---------+----------------------------------------+
| page_id | bitmap_union_count(to_bitmap(user_id)) |
+---------+----------------------------------------+
|       1 |                                      2 |
|       2 |                                      1 |
+---------+----------------------------------------+

```
