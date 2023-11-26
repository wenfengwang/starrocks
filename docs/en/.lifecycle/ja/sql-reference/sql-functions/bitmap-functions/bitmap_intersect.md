---
displayed_sidebar: "Japanese"
---

# bitmap_intersect

## 説明

集計関数であり、グループ化後のビットマップの共通部分を計算するために使用されます。ユーザーの維持率など、一般的な使用シナリオに適しています。

## 構文

```Haskell
BITMAP BITMAP_INTERSECT(BITMAP value)
```

ビットマップの集合を入力し、その集合の共通部分を見つけて結果を返します。

## 例

テーブル構造

```yml
KeysType: AGG_KEY
Columns: tag varchar, date datetime, user_id bitmap bitmap_union
```

```SQL
-- 異なるタグのユーザーの維持率を計算します（今日と昨日）。 
select tag, bitmap_intersect(user_id)
from (
    select tag, date, bitmap_union(user_id) user_id
    from table
    where date in ('2020-05-18', '2020-05-19')
    group by tag, date) a
group by tag;
```

bitmap_to_string 関数と組み合わせて、共通部分の特定のデータを取得します。

```SQL
-- 異なるタグのユーザーの維持率を見つけます（今日と昨日）。 
select tag, bitmap_to_string(bitmap_intersect(user_id))
from (
    select tag, date, bitmap_union(user_id) user_id
    from table where date in ('2020-05-18', '2020-05-19')
    group by tag, date) a
group by tag;
```

## キーワード

BITMAP_INTERSECT, BITMAP
