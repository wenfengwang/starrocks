---
displayed_sidebar: English
---

# 位图

以下是一个简单的例子，用以阐释在Bitmap中一些聚合函数的使用方法。要了解更多关于函数的详细定义或其他位图函数，请参考位图函数文档。

## 创建表格

在创建表格时，需要指定聚合模型。数据类型为bitmap，聚合函数使用的是bitmap_union。

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

注意：在处理大量数据时，建议创建一个与频繁使用的bitmap_union相匹配的汇总表。

```SQL
ALTER TABLE pv_bitmap ADD ROLLUP pv (page, user_id);
```

## 数据加载

TO_BITMAP(expr)：将0至18446744073709551615范围内的无符号bigint转换为位图。

BITMAP_EMPTY()：生成一个空的位图列，用作插入或输入数据时的默认值填充。

BITMAP_HASH(expr)：通过哈希运算将任意类型的列转换为位图。

### 流式加载

使用Stream Load导入数据时，可以如下将数据转换为Bitmap字段：

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

### 插入数据

使用Insert Into导入数据时，需要根据源表中的列类型选择相应的模式。

* 源表中id2的列类型是bitmap。

```SQL
insert into bitmap_table1
select id, id2 from bitmap_table2;
```

* 目标表中id2的列类型是bitmap。

```SQL
insert into bitmap_table1 (id, id2)
values (1001, to_bitmap(1000))
, (1001, to_bitmap(2000));
```

* 源表中id2的列类型是bitmap，是通过bit_map_union()函数聚合得到的结果。

```SQL
insert into bitmap_table1
select id, bitmap_union(id2) from bitmap_table2 group by id;
```

* 源表中id2的列类型是INT，通过to_bitmap()函数生成bitmap类型。

```SQL
insert into bitmap_table1
select id, to_bitmap(id2) from table;
```

* 源表中id2的列类型是STRING，通过bitmap_hash()函数生成bitmap类型。

```SQL
insert into bitmap_table1
select id, bitmap_hash(id2) from table;
```

## 数据查询

### 语法

BITMAP_UNION(expr)：计算输入位图的并集，并返回新位图。

BITMAP_UNION_COUNT(expr)：计算输入位图的并集，并返回其基数，这等同于BITMAP_COUNT(BITMAP_UNION(expr))。推荐优先使用BITMAP_UNION_COUNT函数，因为其性能比BITMAP_COUNT(BITMAP_UNION(expr))更好。

BITMAP_UNION_INT(expr)：计算TINYINT、SMALLINT和INT类型列中不同值的数量，返回的值与COUNT(DISTINCT expr)相同。

INTERSECT_COUNT(bitmap_column_to_count, filter_column, filter_values...)：计算满足filter_column条件的多个位图交集的基数。bitmap_column_to_count是位图类型的列，filter_column是不同维度的列，filter_values是维度值的列表。

BITMAP_INTERSECT(expr)：计算这组位图值的交集并返回一个新的位图。

### 示例

以下SQL以pv_bitmap表为例进行说明：

计算user_id的去重值：

```SQL
select bitmap_union_count(user_id)
from pv_bitmap;

select bitmap_count(bitmap_union(user_id))
from pv_bitmap;
```

计算id的去重值：

```SQL
select bitmap_union_int(id)
from pv_bitmap;
```

计算user_id的留存率：

```SQL
select intersect_count(user_id, page, 'game') as game_uv,
    intersect_count(user_id, page, 'shopping') as shopping_uv,
    intersect_count(user_id, page, 'game', 'shopping') as retention -- Number of users that access both the 'game' and 'shopping' pages
from pv_bitmap
where page in ('game', 'shopping');
```

## 关键字

BITMAP、BITMAP_COUNT、BITMAP_EMPTY、BITMAP_UNION、BITMAP_UNION_INT、TO_BITMAP、BITMAP_UNION_COUNT、INTERSECT_COUNT、BITMAP_INTERSECT。
