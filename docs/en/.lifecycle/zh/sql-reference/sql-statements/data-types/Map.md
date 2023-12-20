---
displayed_sidebar: English
---

# MAP

## 描述

MAP 是一种复杂的数据类型，它存储一组键值对，例如 `{a:1, b:2, c:3}`。MAP 中的键必须是唯一的。嵌套 MAP 最多可以包含 14 层嵌套。

从 v3.1 版本开始支持 MAP 数据类型。在 v3.1 版本中，您可以在创建 StarRocks 表时定义 MAP 列，将 MAP 数据加载到该表中，并查询 MAP 数据。

从 v2.5 版本开始，StarRocks 支持从数据湖查询复杂数据类型 MAP 和 STRUCT。您可以使用 StarRocks 提供的外部目录来查询 Apache Hive™、Apache Hudi 和 Apache Iceberg 中的 MAP 和 STRUCT 数据。您只能查询 ORC 和 Parquet 文件中的数据。有关如何使用外部目录查询外部数据源的更多信息，请参阅[目录概述](../../../data_source/catalog/catalog_overview.md)和相关的目录类型主题。

## 语法

```Haskell
MAP<key_type,value_type>
```

- `key_type`：键的数据类型。键必须是 StarRocks 支持的原始类型，例如数字、字符串或日期。它不能是 HLL、JSON、ARRAY、MAP、BITMAP 或 STRUCT 类型。
- `value_type`：值的数据类型。值可以是任何支持的类型。

键和值**本身可以为空**。

## 在 StarRocks 中定义 MAP 列

您可以在创建表时定义 MAP 列，并将 MAP 数据加载到该列中。

```SQL
-- 定义一维 MAP。
CREATE TABLE t0(
  c0 INT,
  c1 MAP<INT,INT>
)
DUPLICATE KEY(c0);

-- 定义嵌套 MAP。
CREATE TABLE t1(
  c0 INT,
  c1 MAP<DATE, MAP<VARCHAR(10), INT>>
)
DUPLICATE KEY(c0);

-- 定义 NOT NULL MAP。
CREATE TABLE t2(
  c0 INT,
  c1 MAP<INT,DATETIME> NOT NULL
)
DUPLICATE KEY(c0);
```

带有 MAP 类型的列有以下限制：

- 不能用作表中的键列。它们只能用作值列。
- 不能用作表中的分区键列（PARTITION BY 后的列）。
- 不能用作表中的分桶列（DISTRIBUTED BY 后的列）。

## 在 SQL 中构造 MAP

可以使用以下两种语法在 SQL 中构造 MAP：

- `map{key_expr:value_expr, ...}`：MAP 元素之间用逗号（`,`）分隔，键和值之间用冒号（`:`）分隔，例如 `map{a:1, b:2, c:3}`。

- `map(key_expr, value_expr ...)`：键和值的表达式必须成对出现，例如 `map(a,1,b,2,c,3)`。

StarRocks 可以从所有输入的键和值中推导出键和值的数据类型。

```SQL
select map{1:1, 2:2, 3:3} as numbers;
select map(1,1,2,2,3,3) as numbers; -- 返回 {1:1,2:2,3:3}。
select map{1:"apple", 2:"orange", 3:"pear"} as fruit;
select map(1, "apple", 2, "orange", 3, "pear") as fruit; -- 返回 {1:"apple",2:"orange",3:"pear"}。
select map{true:map{3.13:"abc"}, false:map{}} as nest;
select map(true, map(3.13, "abc"), false, map{}) as nest; -- 返回 {true:{3.13:"abc"},false:{}}。
```

如果键或值具有不同的类型，StarRocks 会自动推导出合适的类型（超类型）。

```SQL
select map{1:2.2, 1.2:21} as floats_floats; -- 返回 {1.0:2.2,1.2:21.0}。
select map{12:"a", "100":1, NULL:NULL} as string_string; -- 返回 {"12":"a","100":"1",null:null}。
```

您还可以在构造 MAP 时使用 `<>` 来定义数据类型。输入的键或值必须能够转换为指定的类型。

```SQL
select map<FLOAT,INT>{1:2}; -- 返回 {1:2}。
select map<INT,INT>{"12": "100"}; -- 返回 {12:100}。
```

键和值可以为空。

```SQL
select map{1:NULL};
```

构造空 MAP。

```SQL
select map{} as empty_map;
select map() as empty_map; -- 返回 {}。
```

## 将 MAP 数据加载到 StarRocks 中

您可以使用两种方法将 MAP 数据加载到 StarRocks 中：[INSERT INTO](../../../loading/InsertInto.md) 和 [ORC/Parquet 加载](../data-manipulation/BROKER_LOAD.md)。

请注意，StarRocks 在加载 MAP 数据时会移除每个 MAP 中的重复键。

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

StarRocks 中的 MAP 数据类型对应于 ORC 或 Parquet 格式中的 map 结构。无需额外的规范。您可以按照 [ORC/Parquet 加载](../data-manipulation/BROKER_LOAD.md) 中的指南从 ORC 或 Parquet 文件加载 MAP 数据。

## 访问 MAP 数据

示例 1：从表 `t0` 中查询 MAP 列 `c1`。

```Plain
mysql> select c1 from t0;
+--------------+
| c1           |
+--------------+
| {1:2,3:null} |
+--------------+
```

示例 2：使用 `[]` 运算符按键从 MAP 中检索值，或使用 `element_at(any_map, any_key)` 函数。

以下示例查询键 `1` 对应的值。

```Plain
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

如果 MAP 中不存在该键，则返回 `NULL`。

以下示例查询键 `2` 对应的值，该键不存在。

```Plain
mysql> select map{1:2,3:NULL}[2];
+-----------------------+
| map(1, 2, 3, NULL)[2] |
+-----------------------+
|                  NULL |
+-----------------------+
```

示例 3：**递归**查询多维 MAP。

以下示例首先查询键 `1` 对应的值，即 `map{2:1}`，然后递归查询 `map{2:1}` 中键 `2` 对应的值。

```Plain
mysql> select map{1:map{2:1},3:NULL}[1][2];

+----------------------------------+
| map(1, map(2, 1), 3, NULL)[1][2] |
+----------------------------------+
|                                1 |
+----------------------------------+
```

## 参考资料

- [Map 函数](../../sql-functions/map-functions/map_values.md)
- [element_at](../../sql-functions/array-functions/element_at.md)