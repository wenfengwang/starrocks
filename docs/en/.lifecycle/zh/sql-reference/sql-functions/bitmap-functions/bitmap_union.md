---
displayed_sidebar: English
---

# bitmap_union

## 描述

在分组后计算一组值的bitmap并集。常见的使用场景包括计算PV和UV。

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

使用此函数结合bitmap_count()来获取网页的UV。

```sql
select page_id, bitmap_count(bitmap_union(user_id))
from table
group by page_id;
```

如果`user_id`是整型的，那么上面的查询语句等同于下面的语句：

```sql
select page_id, count(distinct user_id)
from table
group by page_id;
```

## 关键字

BITMAP_UNION, BITMAP