---
displayed_sidebar: English
---

# 地图

## 描述

MAP 是一种复杂的数据类型，用于存储一组键值对，例如 `{a:1, b:2, c:3}`。映射中的键必须是唯一的。嵌套映射最多可以包含 14 个嵌套级别。

MAP 数据类型从 v3.1 版本开始得到支持。在 v3.1 版本中，您可以在创建 StarRocks 表时定义 MAP 列，将 MAP 数据加载到该表中，并查询 MAP 数据。

从 v2.5 版本开始，StarRocks 支持从数据湖中查询复杂数据类型 MAP 和 STRUCT。您可以使用 StarRocks 提供的外部目录查询 Apache Hive™、Apache Hudi 和 Apache Iceberg 的 MAP 和 STRUCT 数据。您只能查询 ORC 和 Parquet 文件中的数据。有关如何使用外部目录查询外部数据源的详细信息，请参阅[目录概述](../../../data_source/catalog/catalog_overview.md)以及相关的主题。

## 语法

```Haskell
MAP<key_type,value_type>
```

- `key_type`：键的数据类型。键必须是 StarRocks 支持的原始类型，如数字、字符串或日期。不能是 HLL、JSON、ARRAY、MAP、BITMAP 或 STRUCT 类型。
- `value_type`：值的数据类型。值可以是任何受支持的类型。

键和值本身可为空。

## 在 StarRocks 中定义 MAP 列

您可以在创建表时定义 MAP 列，并将 MAP 数据加载到该列中。

```SQL
-- 定义一维映射。
CREATE TABLE t0(
  c0 INT,
  c1 MAP<INT,INT>
)
DUPLICATE KEY(c0);

-- 定义嵌套映射。
CREATE TABLE t1(
  c0 INT,
  c1 MAP<DATE, MAP<VARCHAR(10), INT>>
)
DUPLICATE KEY(c0);

-- 定义 NOT NULL 映射。
CREATE TABLE t2(
  c0 INT,
  c1 MAP<INT,DATETIME> NOT NULL
)
DUPLICATE KEY(c0);
```

具有 MAP 类型的列有以下限制：

- 不能用作表中的键列。只能用作值列。
- 不能用作表中的分区键列（PARTITION BY 后面的列）。
- 不能用作表中的分桶列（DISTRIBUTED BY 后面的列）。

## 在 SQL 中构造映射

可以使用以下两种语法在 SQL 中构造 Map：

- `map{key_expr:value_expr, ...}`：映射元素用逗号（`,`）分隔，键和值用冒号（`:`）分隔，例如 `map{a:1, b:2, c:3}`。

- `map(key_expr, value_expr ...)`：键和值的表达式必须成对，例如 `map(a,1,b,2,c,3)`。

StarRocks 可以从所有输入的键和值中推断出键和值的数据类型。

```SQL
select map{1:1, 2:2, 3:3} as numbers;
select map(1,1,2,2,3,3) as numbers; -- 返回 {1:1,2:2,3:3}。
select map{1:"apple", 2:"orange", 3:"pear"} as fruit;
select map(1, "apple", 2, "orange", 3, "pear") as fruit; -- 返回 {1:"apple",2:"orange",3:"pear"}。
select map{true:map{3.13:"abc"}, false:map{}} as nest;
select map(true, map(3.13, "abc"), false, map{}) as nest; -- 返回 {1:{3.13:"abc"},0:{}}。
```

如果键或值的类型不同，StarRocks 会自动推断出适当的类型（超类型）。

```SQL
select map{1:2.2, 1.2:21} as floats_floats; -- 返回 {1.0:2.2,1.2:21.0}。
select map{12:"a", "100":1, NULL:NULL} as string_string; -- 返回 {"12":"a","100":"1",null:null}。
```

您还可以在构造映射时使用 `<>` 定义数据类型。输入的键或值必须能够转换为指定的类型。

```SQL
select map<FLOAT,INT>{1:2}; -- 返回 {1:2}。
select map<INT,INT>{"12": "100"}; -- 返回 {12:100}。
```

键和值可为空。

```SQL
select map{1:NULL};
```

构造空映射。

```SQL
select map{} as empty_map;
select map() as empty_map; -- 返回 {}。
```

## 将 MAP 数据加载到 StarRocks 中

您可以使用 [INSERT INTO](../../../loading/InsertInto.md) 和 [ORC/Parquet 加载](../data-manipulation/BROKER_LOAD.md) 两种方法将地图数据加载到 StarRocks 中。

请注意，StarRocks 在加载 MAP 数据时会移除每个映射的重复键。

### INSERT INTO

```SQL
  CREATE TABLE t0(
    c0 INT,
    c1 MAP<INT,INT>
  )
  DUPLICATE KEY(c0);

  INSERT INTO t0 VALUES(1, map{1:2,3:NULL});
```

### 从 ORC 和 Parquet 文件加载 MAP 数据

StarRocks 中的 MAP 数据类型对应 ORC 或 Parquet 格式的映射结构。不需要额外的规格。您可以按照[ORC/Parquet 加载](../data-manipulation/BROKER_LOAD.md)中的说明从 ORC 或 Parquet 文件加载 MAP 数据。

## 访问 MAP 数据

示例 1：从表 `t0` 中查询 MAP 列 `c1`。

```Plain Text
mysql> select c1 from t0;
+--------------+
| c1           |
+--------------+
| {1:2,3:null} |
+--------------+
```

示例 2：使用 `[ ]` 运算符按键从映射中检索值，或使用函数 `element_at(any_map, any_key)`。

以下示例查询与键 `1` 对应的值。

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

如果映射中不存在该键，将返回 `NULL`。

以下示例查询键为 `2` 的值，该键不存在。

```Plain Text
mysql> select map{1:2,3:NULL}[2];
+-----------------------+
| map(1, 2, 3, NULL)[2] |
+-----------------------+
|                  NULL |
+-----------------------+
```

示例 3：递归查询多维映射。

以下示例首先查询键为 `1` 的值，即 `map{2:1}`，然后递归查询 `map{2:1}` 中键为 `2` 的值。

```Plain Text
mysql> select map{1:map{2:1},3:NULL}[1][2];

+----------------------------------+
| map(1, map(2, 1), 3, NULL)[1][2] |
+----------------------------------+
|                                1 |
+----------------------------------+
```

## 引用

- [Map 函数](../../sql-functions/map-functions/map_values.md)
- [element_at](../../sql-functions/array-functions/element_at.md)
