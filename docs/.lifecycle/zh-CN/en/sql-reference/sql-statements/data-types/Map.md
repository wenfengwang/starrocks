---
displayed_sidebar: "Chinese"
---

# MAP

## Description

MAP是一种复杂数据类型，它存储一组键值对，例如`{a:1, b:2, c:3}`。 MAP中的键必须是唯一的。嵌套的MAP最多可以包含14层嵌套。

MAP数据类型从v3.1开始受支持。在v3.1中，您可以在创建StarRocks表时定义MAP列，将MAP数据加载到该表中，并查询MAP数据。

从v2.5开始，StarRocks支持从数据湖中查询复杂数据类型MAP和STRUCT。您可以使用StarRocks提供的外部目录来查询来自Apache Hive™，Apache Hudi和Apache Iceberg的MAP和STRUCT数据。您只能查询ORC和Parquet文件中的数据。有关如何使用外部目录来查询外部数据源的更多信息，请参见[目录概览](../../../data_source/catalog/catalog_overview.md)以及相关的所需目录类型的主题。

## 语法

```Haskell
MAP<key_type,value_type>
```

- `key_type`：键的数据类型。键必须是StarRocks支持的原始类型，例如数字、字符串或日期。它不能是HLL、JSON、ARRAY、MAP、BITMAP或STRUCT类型。
- `value_type`：值的数据类型。值可以是任何受支持的类型。

键和值都是**本地可为空**。

## 在StarRocks中定义MAP列

您可以在创建表并将MAP数据加载到该列时定义MAP列。

```SQL
-- 定义一维map。
CREATE TABLE t0(
  c0 INT,
  c1 MAP<INT,INT>
)
DUPLICATE KEY(c0);

-- 定义一个嵌套map。
CREATE TABLE t1(
  c0 INT,
  c1 MAP<DATE, MAP<VARCHAR(10), INT>>
)
DUPLICATE KEY(c0);

-- 定义一个非空map。
CREATE TABLE t2(
  c0 INT,
  c1 MAP<INT,DATETIME> NOT NULL
)
DUPLICATE KEY(c0);
```

具有MAP类型的列有以下限制：

- 不能用作表中的键列。它们只能用作值列。
- 不能用作表中的分区键列（在PARTITION BY后面的列）。
- 不能用作表中的桶列（在DISTRIBUTED BY后面的列）。

## 在SQL中构造MAP

可以使用以下两种语法在SQL中构造Map：

- `map{key_expr:value_expr, ...}`：地图元素由逗号（`,`）分隔，键和值由冒号（`:`）分隔，例如`map{a:1, b:2, c:3}`。

- `map(key_expr, value_expr ...)`：键和值的表达式必须成对出现，例如`map(a,1,b,2,c,3)`。

StarRocks可以从所有输入的键和值中推导出键和值的数据类型。

```SQL
select map{1:1, 2:2, 3:3} as numbers;
select map(1,1,2,2,3,3) as numbers; -- 返回 {1:1,2:2,3:3}。
select map{1:"apple", 2:"orange", 3:"pear"} as fruit;
select map(1, "apple", 2, "orange", 3, "pear") as fruit; -- 返回 {1:"apple",2:"orange",3:"pear"}。
select map{true:map{3.13:"abc"}, false:map{}} as nest;
select map(true, map(3.13, "abc"), false, map{}) as nest; -- 返回 {1:{3.13:"abc"},0:{}}。
```

如果键或值具有不同的类型，StarRocks会自动推导出适当的类型（超类型）。

```SQL
select map{1:2.2, 1.2:21} as floats_floats; -- 返回 {1.0:2.2,1.2:21.0}。
select map{12:"a", "100":1, NULL:NULL} as string_string; -- 返回 {"12":"a","100":"1",null:null}。
```

构造地图时还可以使用`<>`来定义数据类型。输入键或值必须能够转换为指定的类型。

```SQL
select map<FLOAT,INT>{1:2}; -- 返回 {1:2}。
select map<INT,INT>{"12": "100"}; -- 返回 {12:100}。
```

键和值可以为空。

```SQL
select map{1:NULL};
```

构造空地图。

```SQL
select map{} as empty_map;
select map() as empty_map; -- 返回 {}。
```

## 将MAP数据加载到StarRocks

您可以使用两种方法将地图数据加载到StarRocks中：[INSERT INTO](../../../loading/InsertInto.md) 和[ORC/Parquet加载](../data-manipulation/BROKER_LOAD.md)。

请注意，StarRocks在加载MAP数据时会删除每个地图的重复键。

### INSERT INTO

```SQL
  CREATE TABLE t0(
    c0 INT,
    c1 MAP<INT,INT>
  )
  DUPLICATE KEY(c0);

  INSERT INTO t0 VALUES(1, map{1:2,3:NULL});
```

### 从ORC和Parquet文件加载MAP数据

StarRocks中的MAP数据类型对应于ORC或Parquet格式中的map结构。不需要其他规范。您可以按照[ORC/Parquet加载](../data-manipulation/BROKER_LOAD.md)中的说明从ORC或Parquet文件中加载MAP数据。

## 访问MAP数据

示例1：从表`t0`中查询MAP列`c1`。

```Plain Text
mysql> select c1 from t0;
+--------------+
| c1           |
+--------------+
| {1:2,3:null} |
+--------------+
```

示例2：使用`[ ]`运算符按键检索地图中的值，或使用`element_at(any_map, any_key)`函数。

以下示例查询与键`1`对应的值。

```Plain Text
mysql> select map{1:2,3:NULL}[1];
+-----------------------+
| map(1, 2, 3, NULL)[1] |
+-----------------------+
|                     2 |
+-----------------------+

mysql> select element_at(map{1:2,3:NULL},1);
+--------------------+
| map{1:2,3:NULL}[1] |
+--------------------+
|                  2 |
+--------------------+
```

如果键在地图中不存在，则返回`NULL`。

以下示例查询不存在的键2对应的值。

```Plain Text
mysql> select map{1:2,3:NULL}[2];
+-----------------------+
| map(1, 2, 3, NULL)[2] |
+-----------------------+
|                  NULL |
+-----------------------+
```

示例3：**递归**查询多维地图。

以下示例首先查询与键`1`对应的值，即`map{2:1}`，然后递归查询`map{2:1}`中键`2`对应的值。

```Plain Text
mysql> select map{1:map{2:1},3:NULL}[1][2];

+----------------------------------+
| map(1, map(2, 1), 3, NULL)[1][2] |
+----------------------------------+
|                                1 |
+----------------------------------+
```

## 引用

- [Map函数](../../sql-functions/map-functions/map_values.md)
- [element_at](../../sql-functions/array-functions/element_at.md)