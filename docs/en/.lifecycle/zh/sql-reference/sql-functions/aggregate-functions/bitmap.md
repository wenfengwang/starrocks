---
displayed_sidebar: English
---

# 位图

下面通过一个简单的例子来说明Bitmap中几个聚合函数的用法。有关详细的函数定义或更多Bitmap函数，请参阅bitmap-functions。

## 创建表

创建表时需要聚合模型。数据类型为bitmap，聚合函数为bitmap_union。

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

注意：当数据量较大时，最好创建一个与高频bitmap_union相对应的rollup表。

```SQL
ALTER TABLE pv_bitmap ADD ROLLUP pv (page, user_id);
```

## 数据加载

`TO_BITMAP(expr)`: 将0 ~ 18446744073709551615无符号bigint转换为bitmap

`BITMAP_EMPTY()`: 生成空bitmap列，用于插入或输入时填充的默认值

`BITMAP_HASH(expr)`: 通过哈希将任何类型的列转换为bitmap

### Stream Load

使用Stream Load输入数据时，可以将数据转换为Bitmap字段，如下所示：

```bash
cat data | curl --location-trusted -u user:passwd -T - \
    -H "columns: dt,page,user_id, user_id=to_bitmap(user_id)" \
    http://host:8410/api/test/testDb/_stream_load
```

```bash
cat data | curl --location-trusted -u user:passwd -T - \
    -H "columns: dt,page,user_id, user_id=bitmap_hash(user_id)" \
    http://host:8410/api/test/testDb/_stream_load
```

```bash
cat data | curl --location-trusted -u user:passwd -T - \
    -H "columns: dt,page,user_id, user_id=bitmap_empty()" \
    http://host:8410/api/test/testDb/_stream_load
```

### Insert Into

使用Insert Into输入数据时，需要根据源表的列类型选择相应的模式。

* 源表中id2的列类型是bitmap

```SQL
insert into bitmap_table1
select id, id2 from bitmap_table2;
```

* 目标表中id2的列类型为bitmap

```SQL
insert into bitmap_table1 (id, id2)
values (1001, to_bitmap(1000))
, (1001, to_bitmap(2000));
```

* 源表中id2的列类型是bitmap，且是使用bitmap_union()聚合的结果。

```SQL
insert into bitmap_table1
select id, bitmap_union(id2) from bitmap_table2 group by id;
```

* 源表中id2的列类型为INT，bitmap类型由to_bitmap()生成。

```SQL
insert into bitmap_table1
select id, to_bitmap(id2) from table;
```

* 源表中id2的列类型为STRING，bitmap类型由bitmap_hash()生成。

```SQL
insert into bitmap_table1
select id, bitmap_hash(id2) from table;
```

## 数据查询

### 语法

`BITMAP_UNION(expr)`: 计算输入Bitmaps的并集，并返回新的Bitmap。

`BITMAP_UNION_COUNT(expr)`: 计算输入Bitmaps的并集，并返回其基数，相当于BITMAP_COUNT(BITMAP_UNION(expr))。建议首先使用BITMAP_UNION_COUNT函数，其性能优于BITMAP_COUNT(BITMAP_UNION(expr))。

`BITMAP_UNION_INT(expr)`: 计算TINYINT、SMALLINT和INT类型列中不同值的数量，返回值与COUNT(DISTINCT expr)相同。

`INTERSECT_COUNT(bitmap_column_to_count, filter_column, filter_values ...)`: 计算满足filter_column条件的多个bitmaps的交集的基数。bitmap_column_to_count是bitmap类型的列，filter_column是不同维度的列，filter_values是维度值的列表。

`BITMAP_INTERSECT(expr)`: 计算这组bitmap值的交集并返回一个新的bitmap。

### 示例

以下SQL以上面的`pv_bitmap`表为例：

计算`user_id`的去重值：

```SQL
select bitmap_union_count(user_id)
from pv_bitmap;

select bitmap_count(bitmap_union(user_id))
from pv_bitmap;
```

计算`id`的去重值：

```SQL
select bitmap_union_int(id)
from pv_bitmap;
```

计算`user_id`的留存率：

```SQL
select intersect_count(user_id, page, 'game') as game_uv,
    intersect_count(user_id, page, 'shopping') as shopping_uv,
    intersect_count(user_id, page, 'game', 'shopping') as retention -- 访问'game'和'shopping'页面的用户数量
from pv_bitmap
where page in ('game', 'shopping');
```

## 关键词

BITMAP, BITMAP_COUNT, BITMAP_EMPTY, BITMAP_UNION, BITMAP_UNION_INT, TO_BITMAP, BITMAP_UNION_COUNT, INTERSECT_COUNT, BITMAP_INTERSECT