---
displayed_sidebar: Chinese
---

# bitmap_intersect

## 機能

複数の bitmap 値を入力し、それらの交差部分を求めて返します。

## 文法

```Haskell
BITMAP_INTERSECT(value)
```

## パラメータ説明

`value`: 対応するデータ型は BITMAP です。

## 戻り値の説明

戻り値のデータ型は BITMAP です。

## 例

テーブル構造

```yml
KeysType: AGG_KEY
Columns: tag varchar, date datetime, user_id bitmap bitmap_union
```

```SQL
-- 今日と昨日の異なる tag におけるユーザーのリテンションを求める。
select tag, bitmap_intersect(user_id)
from (
    select tag, date, bitmap_union(user_id) as user_id
    from table
    where date in ('2020-05-18', '2020-05-19')
    group by tag, date) as a
group by tag;
```

bitmap_to_string 関数と組み合わせることで、交差部分の具体的なデータを取得できます。

```SQL
-- 今日と昨日の異なる tag におけるリテンションユーザーが誰であるかを求める。
select tag, bitmap_to_string(bitmap_intersect(user_id))
from (
    select tag, date, bitmap_union(user_id) as user_id
    from table where date in ('2020-05-18', '2020-05-19')
    group by tag, date) as a
group by tag;
```
