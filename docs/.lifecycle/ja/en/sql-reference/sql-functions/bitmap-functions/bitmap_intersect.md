---
displayed_sidebar: English
---

# bitmap_intersect

## 説明

グループ化後のビットマップの交差部分を計算するために使用される集約関数です。ユーザー維持率を計算するなど、一般的な使用シナリオがあります。

## 構文

```Haskell
BITMAP BITMAP_INTERSECT(BITMAP value)
```

ビットマップ値のセットを入力し、このビットマップ値のセットの交差部分を見つけ、結果を返します。

## 例

テーブル構造

```yml
KeysType: AGG_KEY
Columns: tag varchar, date datetime, user_id bitmap bitmap_union
```

```SQL
-- 今日と昨日の異なるタグの下でのユーザー維持を計算します。
select tag, bitmap_intersect(user_id)
from (
    select tag, date, bitmap_union(user_id) as user_id
    from table
    where date in ('2020-05-18', '2020-05-19')
    group by tag, date) as a
group by tag;
```

bitmap_to_string関数を使用して、交差する具体的なデータを取得します。

```SQL
-- 今日と昨日の異なるタグの下で維持されたユーザーを見つけます。
select tag, bitmap_to_string(bitmap_intersect(user_id))
from (
    select tag, date, bitmap_union(user_id) as user_id
    from table where date in ('2020-05-18', '2020-05-19')
    group by tag, date) as a
group by tag;
```

## キーワード

BITMAP_INTERSECT, BITMAP
