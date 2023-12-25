---
displayed_sidebar: Chinese
---

# bitmap_union

## 機能

一連のbitmap値を入力し、これらのbitmap値の和集合を求めます。

## 文法

```Haskell
BITMAP_UNION(value)
```

## パラメータ説明

`value`: サポートされるデータタイプは BITMAP です。

## 戻り値の説明

戻り値のデータタイプは BITMAP です。

## 例

```sql
select page_id, bitmap_union(user_id)
from table
group by page_id;
```

bitmap_count 関数と組み合わせて、ウェブページの UV データを求めることができます。

```sql
select page_id, bitmap_count(bitmap_union(user_id))
from table
group by page_id;
```

`user_id` フィールドが INT の場合、上記のクエリは以下のステートメントと同等です:

```sql
select page_id, count(distinct user_id)
from table
group by page_id;
```
