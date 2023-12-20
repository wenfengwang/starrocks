---
displayed_sidebar: English
---

# array_sortby

## 描述

根据另一个数组或由 lambda 表达式转换得到的数组中元素的升序对数组中的元素进行排序。有关更多信息，请参阅 [Lambda 表达式](../Lambda_expression.md)。该函数从 v2.5 版本开始支持。

两个数组中的元素类似于键值对。例如，b = [7,5,6] 是 a = [3,1,4] 的排序键。根据键值对关系，两个数组中的元素具有以下一一对应关系。

|**数组**|**元素 1**|**元素 2**|**元素 3**|
|---|---|---|---|
|a|3|1|4|
|b|7|5|6|

数组 `b` 升序排序后，变为 [5,6,7]。数组 `a` 相应地变为 [1,4,3]。

|**数组**|**元素 1**|**元素 2**|**元素 3**|
|---|---|---|---|
|a|1|4|3|
|b|5|6|7|

## 语法

```Haskell
array_sortby(array0, array1)
array_sortby(<lambda function>, array0 [, array1...])
```

- `array_sortby(array0, array1)`

  根据 `array1` 的升序对 `array0` 进行排序。

- `array_sortby(<lambda function>, array0 [, array1...])`

  根据 lambda 函数返回的数组对 `array0` 进行排序。

## 参数

- `array0`：要排序的数组。它必须是数组、数组表达式或 `null`。数组中的元素必须是可排序的。
- `array1`：用于对 `array0` 进行排序的排序数组。它必须是数组、数组表达式或 `null`。
- `<lambda function>`：用于生成排序数组的 lambda 表达式。

## 返回值

返回一个数组。

## 使用说明

- 此函数只能按升序对数组元素进行排序。
- `NULL` 值放置在返回的数组的开头。
- 如果要按降序对数组元素进行排序，请使用 [reverse](./reverse.md) 函数。
- 如果排序数组（`array1`）为 `null`，则 `array0` 中的数据保持不变。
- 返回数组的元素与 `array0` 的元素具有相同的数据类型。`NULL` 值的属性也是相同的。
- 两个数组必须具有相同数量的元素。否则，将返回错误。

## 示例

下表用于演示如何使用该函数。

```SQL
CREATE TABLE `test_array` (
  `c1` int(11) NULL COMMENT "",
  `c2` ARRAY<int(11)> NULL COMMENT "",
  `c3` ARRAY<int(11)> NULL COMMENT ""
) ENGINE=OLAP
DUPLICATE KEY(`c1`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`c1`)
PROPERTIES (
"replication_num" = "3",
"storage_format" = "DEFAULT",
"enable_persistent_index" = "false",
"compression" = "LZ4"
);

insert into test_array values
(1,[4,3,5],[82,1,4]),
(2,null,[23]),
(3,[4,2],[6,5]),
(4,null,null),
(5,[],[]),
(6,NULL,[]),
(7,[],null),
(8,[null,null],[3,6]),
(9,[432,21,23],[5,4,null]);

select * from test_array order by c1;
+------+-------------+------------+
| c1   | c2          | c3         |
+------+-------------+------------+
|    1 | [4,3,5]     | [82,1,4]   |
|    2 | NULL        | [23]       |
|    3 | [4,2]       | [6,5]      |
|    4 | NULL        | NULL       |
|    5 | []          | []         |
|    6 | NULL        | []         |
|    7 | []          | NULL       |
|    8 | [null,null] | [3,6]      |
|    9 | [432,21,23] | [5,4,null] |
+------+-------------+------------+
9 rows in set (0.00 sec)
```

示例 1：根据 `c2` 对 `c3` 进行排序。此示例还提供了 `array_sort()` 的结果以供比较。

```Plaintext
select c1, c3, c2, array_sort(c2), array_sortby(c3,c2)
from test_array order by c1;
+------+------------+-------------+----------------+----------------------+
| c1   | c3         | c2          | array_sort(c2) | array_sortby(c3, c2) |
+------+------------+-------------+----------------+----------------------+
|    1 | [82,1,4]   | [4,3,5]     | [3,4,5]        | [1,82,4]             |
|    2 | [23]       | NULL        | NULL           | [23]                 |
|    3 | [6,5]      | [4,2]       | [2,4]          | [5,6]                |
|    4 | NULL       | NULL        | NULL           | NULL                 |
|    5 | []         | []          | []             | []                   |
|    6 | []         | NULL        | NULL           | []                   |
|    7 | NULL       | []          | []             | NULL                 |
|    8 | [3,6]      | [null,null] | [null,null]    | [3,6]                |
|    9 | [5,4,null] | [432,21,23] | [21,23,432]    | [4,null,5]           |
+------+------------+-------------+----------------+----------------------+
```

示例 2：根据由 lambda 表达式生成的 `c2` 对数组 `c3` 进行排序。此示例与示例 1 等效。它还提供了 `array_sort()` 的结果以供比较。

```Plaintext
select
    c1,
    c3,
    c2,
    array_sort(c2) as sorted_c2_asc,
    array_sortby((x,y) -> y, c3, c2) as sorted_c3_by_c2
from test_array order by c1;
+------+------------+-------------+---------------+-----------------+
| c1   | c3         | c2          | sorted_c2_asc | sorted_c3_by_c2 |
+------+------------+-------------+---------------+-----------------+
|    1 | [82,1,4]   | [4,3,5]     | [3,4,5]       | [82,1,4]        |
|    2 | [23]       | NULL        | NULL          | [23]            |
|    3 | [6,5]      | [4,2]       | [2,4]         | [5,6]           |
|    4 | NULL       | NULL        | NULL          | NULL            |
|    5 | []         | []          | []            | []              |
|    6 | []         | NULL        | NULL          | []              |
|    7 | NULL       | []          | []            | NULL            |
|    8 | [3,6]      | [null,null] | [null,null]   | [3,6]           |
|    9 | [5,4,null] | [432,21,23] | [21,23,432]   | [4,null,5]      |
+------+------------+-------------+---------------+-----------------+
```

示例 3：根据 `c2+c3` 的升序对数组 `c3` 进行排序。

```Plain
select
    c3,
    c2,
    array_map((x,y)-> x+y, c3, c2) as sum,
    array_sort(array_map((x,y)-> x+y, c3, c2)) as sorted_sum,
    array_sortby((x,y) -> x+y, c3, c2) as sorted_c3_by_sum
from test_array where c1=1;
+----------+---------+----------+------------+------------------+
| c3       | c2      | sum      | sorted_sum | sorted_c3_by_sum |
+----------+---------+----------+------------+------------------+
| [82,1,4] | [4,3,5] | [86,4,9] | [4,9,86]   | [1,4,82]         |
+----------+---------+----------+------------+------------------+
```

## 参考资料

[array_sort](array_sort.md)