---
displayed_sidebar: English
---

# 位图合并

## 描述

在分组后，计算一系列值的位图合并。常见应用场景包括计算页面浏览量（PV）和独立访客数（UV）。

## 语法

```Haskell
BITMAP BITMAP_UNION(BITMAP value)
```

## 示例

```sql
select page_id, bitmap_union(user_id)
from table
group by page_id;
```

将这个函数和 bitmap_count() 一起使用，可以得到网页的独立访客数（UV）。

```sql
select page_id, bitmap_count(bitmap_union(user_id))
from table
group by page_id;
```

如果 user_id 是一个整数类型，那么上面的查询语句相当于以下语句：

```sql
select page_id, count(distinct user_id)
from table
group by page_id;
```

## 关键词

BITMAP_UNION，位图
