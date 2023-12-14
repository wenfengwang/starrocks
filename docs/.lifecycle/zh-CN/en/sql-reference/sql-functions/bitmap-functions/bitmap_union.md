---
displayed_sidebar: "Chinese"
---

# bitmap_union

## 描述

计算分组后值集合的位图并集。常见的使用场景包括计算PV和UV。

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

使用此函数与bitmap_count()一起获取网页的UV。

```sql
select page_id, bitmap_count(bitmap_union(user_id))
from table
group by page_id;
```

如果`user_id`是整数，则上述查询语句等同于以下内容：

```sql
select page_id, count(distinct user_id)
from table
group by page_id;
```

## 关键字

BITMAP_UNION, BITMAP