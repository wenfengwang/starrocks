---
displayed_sidebar: English
---

# 位图交集

## 描述

这是一个聚合函数，用于在分组后计算位图的交集。它常用于一些场景，比如计算用户的留存率。

## 语法

```Haskell
BITMAP BITMAP_INTERSECT(BITMAP value)
```

输入一系列位图值，找出这些位图值的交集，并返回结果。

## 示例

表的结构

```yml
KeysType: AGG_KEY
Columns: tag varchar, date datetime, user_id bitmap bitmap_union
```

```SQL
-- Calculate users retention under different tags today and yesterday. 
select tag, bitmap_intersect(user_id)
from (
    select tag, date, bitmap_union(user_id) user_id
    from table
    where date in ('2020-05-18', '2020-05-19')
    group by tag, date) a
group by tag;
```

可以与bitmap_to_string函数一起使用，以获得交集的详细数据。

```SQL
--Find out users retained under different tags today and yesterday. 
select tag, bitmap_to_string(bitmap_intersect(user_id))
from (
    select tag, date, bitmap_union(user_id) user_id
    from table where date in ('2020-05-18', '2020-05-19')
    group by tag, date) a
group by tag;
```

## 关键字

BITMAP_INTERSECT，位图
