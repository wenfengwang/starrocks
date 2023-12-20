---
displayed_sidebar: English
---

# bitmap_intersect

## 描述

聚合函数，用于计算分组后的bitmap交集。常见的使用场景包括计算用户留存率。

## 语法

```Haskell
BITMAP BITMAP_INTERSECT(BITMAP value)
```

输入一组bitmap值，找出这些bitmap值的交集，并返回结果。

## 示例

表结构

```yml
KeysType: AGG_KEY
Columns: tag varchar, date datetime, user_id bitmap bitmap_union
```

```SQL
-- 计算今天和昨天不同标签下的用户留存。
select tag, bitmap_intersect(user_id)
from (
    select tag, date, bitmap_union(user_id) as user_id
    from table
    where date in ('2020-05-18', '2020-05-19')
    group by tag, date) as a
group by tag;
```

结合bitmap_to_string函数使用，以获得交集的具体数据。

```SQL
-- 查找今天和昨天不同标签下保留的用户。
select tag, bitmap_to_string(bitmap_intersect(user_id))
from (
    select tag, date, bitmap_union(user_id) as user_id
    from table where date in ('2020-05-18', '2020-05-19')
    group by tag, date) as a
group by tag;
```

## 关键词

BITMAP_INTERSECT，BITMAP