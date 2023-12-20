---
displayed_sidebar: English
---

# 数组排序

## 描述

根据另一个数组的元素升序，或者根据由 lambda 表达式转换而成的数组对目标数组中的元素进行排序。更多有关 lambda 表达式的信息，请参见[相关文档](../Lambda_expression.md)。该函数自 v2.5 版本起提供支持。

两个数组中的元素类似于键值对。例如，b = [7,5,6] 作为排序键对应于 a = [3,1,4]。根据键值对的关系，两个数组中的元素具有以下的一对一映射：

|数组|元素 1|元素 2|元素 3|
|---|---|---|---|
|a|3|1|4|
|b|7|5|6|

当数组 b 按升序排序后，变为 [5,6,7]，相应地数组 a 变为 [1,4,3]。

|数组|元素 1|元素 2|元素 3|
|---|---|---|---|
|a|1|4|3|
|b|5|6|7|

## 语法

```Haskell
array_sortby(array0, array1)
array_sortby(<lambda function>, array0 [, array1...])
```

- array_sortby(array0, array1)

  根据 array1 的升序对 array0 进行排序。

- array_sortby(<lambda 函数>, array0 [, array1...])

  根据 lambda 函数返回的数组对 array0 进行排序。

## 参数

- array0：你想要排序的数组。它必须是一个数组、数组表达式或者 null。数组中的元素必须可以被排序。
- array1：用来对 array0 进行排序的数组。它必须是一个数组、数组表达式或者 null。
- lambda 函数：用来生成排序数组的 lambda 表达式。

## 返回值

返回排序后的数组。

## 使用说明

- 此函数只能按照升序对数组元素进行排序。
- NULL 值会被放置在返回数组的开头。
- 如果你想要按照降序对数组元素进行排序，请使用[reverse](./reverse.md)函数。
- 如果用于排序的数组（array1）为 null，那么 array0 中的数据将保持不变。
- 返回数组的元素将和 array0 中的元素具有相同的数据类型。NULL 值的属性也保持一致。
- 两个数组必须具有相同的元素数量。否则，将返回一个错误。

## 示例

以下表格用于演示如何使用这个函数。

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

例子 1：根据 c2 对 c3 进行排序。此示例还提供了 array_sort() 函数的比较结果。

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

例子 2：基于由 lambda 表达式生成的 c2 对数组 c3 进行排序。此示例与例子 1 相当。它同样提供了 array_sort() 函数的比较结果。

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

例子 3：根据 c2 与 c3 之和的升序对数组 c3 进行排序。

```Plain
select
    c3,
    c2,
    array_map((x,y)-> x+y,c3,c2) as sum,
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

[array_sort](array_sort.md)函数
