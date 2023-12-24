---
displayed_sidebar: English
---

# bitmap_union

## 描述

计算分组后一组数值的位图并集。常见的使用场景包括计算 PV 和 UV。

## 语法

```Haskell
BITMAP BITMAP_UNION(BITMAP value)
```

## 例子

```sql
select page_id, bitmap_union(user_id)
from table
group by page_id;
```

使用此函数与 bitmap_count() 结合可获取网页的 UV。

```sql
select page_id, bitmap_count(bitmap_union(user_id))
from table
group by page_id;
```

如果 `user_id` 是整数，上述查询语句等同于以下内容：

```sql
select page_id, count(distinct user_id)
from table
group by page_id;
```

## 关键词

BITMAP_UNION、位图
