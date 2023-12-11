---
displayed_sidebar: "Japanese"
---

# bitmap_union

## 説明

グループ化した後の値のセットのビットマップの和を計算します。一般的な使用シナリオには、PVとUVの計算が含まれます。

## 構文

```Haskell
BITMAP BITMAP_UNION(BITMAP value)
```

## 例

```sql
select page_id, bitmap_union(user_id)
from table
group by page_id;
```

この関数をbitmap_count()と共に使用して、ウェブページのUVを取得します。

```sql
select page_id, bitmap_count(bitmap_union(user_id))
from table
group by page_id;
```

`user_id`が整数である場合、上記のクエリ文は次のと等価です。

```sql
select page_id, count(distinct user_id)
from table
group by page_id;
```

## キーワード

BITMAP_UNION, BITMAP