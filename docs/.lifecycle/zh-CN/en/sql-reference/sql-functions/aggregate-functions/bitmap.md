---
displayed_sidebar: "Chinese"
---

# 位图

这是一个简单的示例，用于说明在位图中使用多个聚合函数的用法。有关详细的函数定义或更多位图函数，请参见位图函数。

## 创建表

在创建表时需要聚合模型。数据类型是位图，聚合函数是bitmap_union。

```SQL
CREATE TABLE `pv_bitmap` (
  `dt` int(11) NULL COMMENT "",
  `page` varchar(10) NULL COMMENT "",
  `user_id` bitmap BITMAP_UNION NULL COMMENT ""
) ENGINE=OLAP
AGGREGATE KEY(`dt`, `page`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`dt`);
```

注意：在有大量数据的情况下，最好创建一个与bitmap_union对应的滚动表。

```SQL
ALTER TABLE pv_bitmap ADD ROLLUP pv (page, user_id);
```

## 数据加载

`TO_BITMAP (expr)`: 将0 ~ 18446744073709551615的无符号大整数转换为位图

`BITMAP_EMPTY ()`: 生成空的位图列，用于插入或输入时要填充的默认值

`BITMAP_HASH (expr)`: 通过哈希将任何类型的列转换为位图

### 流加载

在使用流加载输入数据时，可以将数据转换为位图字段，如下所示：

``` bash
cat data | curl --location-trusted -u user:passwd -T - \
    -H "columns: dt,page,user_id, user_id=to_bitmap(user_id)" \
    http://host:8410/api/test/testDb/_stream_load
```

``` bash
cat data | curl --location-trusted -u user:passwd -T - \
    -H "columns: dt,page,user_id, user_id=bitmap_hash(user_id)" \
    http://host:8410/api/test/testDb/_stream_load
```

``` bash
cat data | curl --location-trusted -u user:passwd -T - \
    -H "columns: dt,page,user_id, user_id=bitmap_empty()" \
    http://host:8410/api/test/testDb/_stream_load
```

### 插入数据

在使用Insert Into输入数据时，根据源表中列的类型选择相应的模式。

* 源表中id2的列类型为位图

```SQL
insert into bitmap_table1
select id, id2 from bitmap_table2;
```

* 目标表中id2的列类型为位图

```SQL
insert into bitmap_table1 (id, id2)
values (1001, to_bitmap(1000))
, (1001, to_bitmap(2000));
```

* 源表中id2的列类型为位图，并且是使用bit_map_union()进行聚合的结果。

```SQL
insert into bitmap_table1
select id, bitmap_union(id2) from bitmap_table2 group by id;
```

* 源表中id2的列类型为INT，位图类型由to_bitmap()生成。

```SQL
insert into bitmap_table1
select id, to_bitmap(id2) from table;
```

* 源表中id2的列类型为STRING，位图类型由bitmap_hash()生成。

```SQL
insert into bitmap_table1
select id, bitmap_hash(id2) from table;
```

## 数据查询

### 语法

``BITMAP_UNION (expr)`: 计算输入位图的并集，并返回新的位图。

`BITMAP_UNION_COUNT (expr)`: 计算输入位图的并集，并返回其基数，等同于BITMAP_COUNT (BITMAP_UNION (expr))。建议先使用BITMAP_UNION_COUNT函数，因为其性能优于BITMAP_COUNT (BITMAP_UNION (expr))。

`BITMAP_UNION_INT (expr)`: 计算类型为TINYINT、SMALLINT和INT的列中不同值的数量，返回与COUNT (DISTINCT expr) 相同的值。

`INTERSECT_COUNT (bitmap_column_to_count, filter_column, filter_values ...)`: 计算满足filter_column条件的多个位图的交集的基数。bitmap_column_to_count是位图类型的列，filter_column是变化维度的列，filter_values是维度值的列表。

`BITMAP_INTERSECT(expr)`: 计算该组位图值的交集，并返回一个新的位图。

### 示例

以下SQL以上述`pv_bitmap`表为例：

计算'user_id'的去重值：

```SQL
select bitmap_union_count(user_id)
from pv_bitmap;

select bitmap_count(bitmap_union(user_id))
from pv_bitmap;
```

计算'id'的去重值：

```SQL
select bitmap_union_int(id)
from pv_bitmap;
```

计算'user_id'的留存：

```SQL
select intersect_count(user_id, page, 'game') as game_uv,
    intersect_count(user_id, page, 'shopping') as shopping_uv,
    intersect_count(user_id, page, 'game', 'shopping') as retention -- 访问'game'和'shopping'页面的用户数
from pv_bitmap
where page in ('game', 'shopping');
```

## 关键词

BITMAP,BITMAP_COUNT,BITMAP_EMPTY,BITMAP_UNION,BITMAP_UNION_INT,TO_BITMAP,BITMAP_UNION_COUNT,INTERSECT_COUNT,BITMAP_INTERSECT