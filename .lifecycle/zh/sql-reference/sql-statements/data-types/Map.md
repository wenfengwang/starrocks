---
displayed_sidebar: English
---

# 映射

## 描述

MAP 是一种复杂的数据类型，用于存储键值对集合，例如 {a:1, b:2, c:3}。映射中的键必须唯一。嵌套映射最多可以包含 14 层嵌套。

MAP 数据类型从 v3.1 版本开始得到支持。在 v3.1 版本中，您可以在创建 StarRocks 表时定义 MAP 类型的列，将 MAP 数据加载进表中，并进行 MAP 数据的查询。

从 v2.5 版本开始，StarRocks支持查询复杂数据类型**MAP**和**STRUCT**。您可以利用StarRocks提供的外部目录功能，从Apache Hive™、Apache Hudi和Apache Iceberg中查询**MAP**和**STRUCT**数据。目前仅支持查询**ORC**和**Parquet**文件格式的数据。有关如何使用外部目录来查询外部数据源的更多信息，请参见[目录概览](../../../data_source/catalog/catalog_overview.md)及相关目录类型的主题。

## 语法

```Haskell
MAP<key_type,value_type>
```

- key_type：键的数据类型。键必须是 StarRocks 支持的基本类型，如数字、字符串或日期类型。不能是 HLL、JSON、ARRAY、MAP、BITMAP 或 STRUCT 类型。
- value_type：值的数据类型。值可以是任何 StarRocks 支持的类型。

键和值**默认允许**为空。

## 在 StarRocks 中定义 MAP 类型的列

您可以在创建表时定义 MAP 类型的列，并将 MAP 数据加载到此列中。

```SQL
-- Define a one-dimensional map.
CREATE TABLE t0(
  c0 INT,
  c1 MAP<INT,INT>
)
DUPLICATE KEY(c0);

-- Define a nested map.
CREATE TABLE t1(
  c0 INT,
  c1 MAP<DATE, MAP<VARCHAR(10), INT>>
)
DUPLICATE KEY(c0);

-- Define a NOT NULL map.
CREATE TABLE t2(
  c0 INT,
  c1 MAP<INT,DATETIME> NOT NULL
)
DUPLICATE KEY(c0);
```

MAP 类型的列有以下限制：

- 不能用作表的主键列。它们只能作为值列使用。
- 不能用作表的分区键列（在 PARTITION BY 语句之后的列）。
- 不能用作表的分桶键列（在 DISTRIBUTED BY 语句之后的列）。

## 在 SQL 中构造映射

可以使用以下两种语法在 SQL 中构造映射：

- map{key_expr:value_expr, ...}：映射元素间用逗号（,）分隔，键与值之间用冒号（:）分隔，例如 map{a:1, b:2, c:3}。

- map(key_expr, value_expr, ...)：键与值的表达式必须成对出现，例如 map(a,1,b,2,c,3)。

StarRocks 能够根据所有输入的键和值推导出键和值的数据类型。

```SQL
select map{1:1, 2:2, 3:3} as numbers;
select map(1,1,2,2,3,3) as numbers; -- Return {1:1,2:2,3:3}.
select map{1:"apple", 2:"orange", 3:"pear"} as fruit;
select map(1, "apple", 2, "orange", 3, "pear") as fruit; -- Return {1:"apple",2:"orange",3:"pear"}.
select map{true:map{3.13:"abc"}, false:map{}} as nest;
select map(true, map(3.13, "abc"), false, map{}) as nest; -- Return {1:{3.13:"abc"},0:{}}.
```

如果键或值具有不同的类型，StarRocks 会自动推导出一个合适的超类型。

```SQL
select map{1:2.2, 1.2:21} as floats_floats; -- Return {1.0:2.2,1.2:21.0}.
select map{12:"a", "100":1, NULL:NULL} as string_string; -- Return {"12":"a","100":"1",null:null}.
```

您也可以在构造映射时使用尖括号 <> 来定义数据类型。输入的键或值必须能够转换为指定的类型。

```SQL
select map<FLOAT,INT>{1:2}; -- Return {1:2}.
select map<INT,INT>{"12": "100"}; -- Return {12:100}.
```

键和值允许为空。

```SQL
select map{1:NULL};
```

构造空映射。

```SQL
select map{} as empty_map;
select map() as empty_map; -- Return {}.
```

## 将 MAP 数据加载到 StarRocks

您可以通过两种方法将 MAP 数据加载到 StarRocks：[INSERT INTO](../../../loading/InsertInto.md)，和[ORC/Parquet loading](../data-manipulation/BROKER_LOAD.md)。

请注意，在加载 MAP 数据时，StarRocks 会去除每个映射中的重复键。

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

StarRocks 中的 MAP 数据类型与 ORC 或 Parquet 格式中的映射结构相对应。无需额外的说明。您可以按照[ORC/Parquet loading](../data-manipulation/BROKER_LOAD.md)的指南，从 ORC 或 Parquet 文件中加载 MAP 数据。

## 访问 MAP 数据

示例 1：从表 t0 中查询 MAP 类型的列 c1。

```Plain
mysql> select c1 from t0;
+--------------+
| c1           |
+--------------+
| {1:2,3:null} |
+--------------+
```

示例 2：使用 [ ] 运算符通过键从映射中检索值，或使用 element_at(any_map, any_key) 函数。

以下示例查询与键 1 对应的值。

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

如果映射中不存在该键，则返回 NULL。

以下示例查询与键 2 对应的值，但该键不存在。

```Plain
mysql> select map{1:2,3:NULL}[2];
+-----------------------+
| map(1, 2, 3, NULL)[2] |
+-----------------------+
|                  NULL |
+-----------------------+
```

示例 3：查询多维地图**递归**。

以下示例首先查询与键 1 对应的值，即 map{2:1}，然后递归查询在 map{2:1} 中与键 2 对应的值。

```Plain
mysql> select map{1:map{2:1},3:NULL}[1][2];

+----------------------------------+
| map(1, map(2, 1), 3, NULL)[1][2] |
+----------------------------------+
|                                1 |
+----------------------------------+
```

## 参考资料

- [映射函数](../../sql-functions/map-functions/map_values.md)
- [element_at](../../sql-functions/array-functions/element_at.md)
