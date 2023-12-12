---
displayed_sidebar: "Japanese"
---

# bitmap_union

## 説明

グループ化した値のビットマップ和を計算します。一般的な使用シナリオには、PVとUVの計算が含まれます。

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

この関数は、`bitmap_count()`と組み合わせてウェブページのUVを取得するために使用します。

```sql
select page_id, bitmap_count(bitmap_union(user_id))
from table
group by page_id;
```

`user_id`が整数である場合、上記のクエリ文は次のものと同等です。

```sql
select page_id, count(distinct user_id)
from table
group by page_id;
```

## キーワード

BITMAP_UNION, BITMAP