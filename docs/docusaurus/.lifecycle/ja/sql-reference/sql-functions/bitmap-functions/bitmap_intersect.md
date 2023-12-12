---
displayed_sidebar: "Japanese"
---

# bitmap_intersect

## 説明

集約関数。グループ化後のビットマップの積集合を計算するために使用されます。ユーザーの継続率など、一般的な使用シナリオがあります。

## 構文

```Haskell
BITMAP BITMAP_INTERSECT(BITMAP value)
```

ビットマップの値の集合を入力し、この集合のビットマップの積集合を見つけて結果を返します。

## 例

テーブル構造

```yml
KeysType: AGG_KEY
Columns: tag varchar, date datetime, user_id bitmap bitmap_union
```

```SQL
-- 異なるタグで今日と昨日のユーザーの継続を計算します。
select tag, bitmap_intersect(user_id)
from (
    select tag, date, bitmap_union(user_id) user_id
    from table
    where date in ('2020-05-18', '2020-05-19')
    group by tag, date) a
group by tag;
```

bitmap_to_string 関数と組み合わせて、積集合の特定のデータを取得します。

```SQL
-- 異なるタグで今日と昨日のユーザーの継続を把握します。
select tag, bitmap_to_string(bitmap_intersect(user_id))
from (
    select tag, date, bitmap_union(user_id) user_id
    from table where date in ('2020-05-18', '2020-05-19')
    group by tag, date) a
group by tag;
```

## キーワード

BITMAP_INTERSECT, BITMAP