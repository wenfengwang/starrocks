---
displayed_sidebar: "Japanese"
---

# bitmap_intersect

## 説明

グループ化後のビットマップの交差を計算するために使用される集約関数。ユーザーの保持率のような一般的な使用シナリオがあります。

## 構文

```Haskell
BITMAP BITMAP_INTERSECT(BITMAP value)
```

ビットマップ値のセットを入力し、そのビットマップ値のセットの交わりを見つけて結果を返します。

## 例

テーブル構造

```yml
KeysType: AGG_KEY
Columns: tag varchar, date datetime, user_id bitmap bitmap_union
```

```SQL
-- 異なるタグのユーザーの保持を計算する。 
select tag, bitmap_intersect(user_id)
from (
    select tag, date, bitmap_union(user_id) user_id
    from table
    where date in ('2020-05-18', '2020-05-19')
    group by tag, date) a
group by tag;
```

bitmap_to_string関数を使用して、交差部分の特定のデータを取得する方法。

```SQL
-- 異なるタグのユーザーの保持を知る。 
select tag, bitmap_to_string(bitmap_intersect(user_id))
from (
    select tag, date, bitmap_union(user_id) user_id
    from table where date in ('2020-05-18', '2020-05-19')
    group by tag, date) a
group by tag;
```

## キーワード

BITMAP_INTERSECT, BITMAP