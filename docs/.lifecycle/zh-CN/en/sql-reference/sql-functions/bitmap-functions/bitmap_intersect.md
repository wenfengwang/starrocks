---
displayed_sidebar: "Chinese"
---

# bitmap_intersect

## 描述

聚合函数，用于在分组后计算位图的交集。常见的使用场景，比如计算用户的留存率。

## 语法

```Haskell
BITMAP BITMAP_INTERSECT(BITMAP value)
```

输入一组位图值，找到这组位图值的交集，并返回结果。

## 示例

表结构

```yml
KeysType: AGG_KEY
Columns: tag varchar, date datetime, user_id bitmap bitmap_union
```

```SQL
-- 计算不同标签下今天和昨天的用户留存情况。
select tag, bitmap_intersect(user_id)
from (
    select tag, date, bitmap_union(user_id) user_id
    from table
    where date in ('2020-05-18', '2020-05-19')
    group by tag, date) a
group by tag;
```

与 `bitmap_to_string` 函数一起使用，获取交集的具体数据。

```SQL
--找出不同标签下今天和昨天的留存用户。
select tag, bitmap_to_string(bitmap_intersect(user_id))
from (
    select tag, date, bitmap_union(user_id) user_id
    from table where date in ('2020-05-18', '2020-05-19')
    group by tag, date) a
group by tag;
```

## 关键词

BITMAP_INTERSECT, BITMAP