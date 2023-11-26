---
displayed_sidebar: "Japanese"
---

# bitmap_union_count

## 説明

ビットマップ値の集合の和を返し、その和の要素数を返します。この関数はv2.3からサポートされています。

## 構文

```Haskell
BIGINT bitmap_union_count(BITMAP value)
```

### パラメーター

`value`: ビットマップ値の集合です。サポートされているデータ型はBITMAPです。

## 戻り値

BIGINT型の値を返します。

## 例

ウェブページのユニークビュー（UV）を計算します。`user_id`がINT型の場合、後の2つのクエリは同等です。

```Plaintext
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
