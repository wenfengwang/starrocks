---
displayed_sidebar: "Chinese"
---

# array_contains_all

## 描述

检查`arr1`是否包含`arr2`的所有元素，也就是说，`arr2`是否是`arr1`的子集。如果是，则返回1。如果不是，则返回0。

## 语法

~~~Haskell
BOOLEAN array_contains_all(arr1, arr2)
~~~

## 参数

`arr`：要比较的两个数组。该语法检查`arr2`是否是`arr1`的子集。

两个数组中元素的数据类型必须相同。有关StarRocks支持的数组元素数据类型，请参见[ARRAY](../../../sql-reference/sql-statements/data-types/Array.md)。

## 返回值

返回BOOLEAN类型的值。

如果`arr2`是`arr1`的子集，则返回1。否则返回0。

如果两个数组中有任何一个是NULL，则返回NULL。

## 使用说明

- 如果一个数组包含`null`元素，则将`null`处理为值。

- 空数组是任何数组的子集。

- 两个数组中的元素可以具有不同的顺序。

## 示例

1. 创建名为`t1`的表并向该表插入数据。

    ~~~SQL
    CREATE TABLE t1 (
        c0 INT,
        c1 ARRAY<INT>,
        c2 ARRAY<INT>
    ) ENGINE=OLAP
    DUPLICATE KEY(c0)
    DISTRIBUTED BY HASH(c0);

    INSERT INTO t1 VALUES
        (1,[1,2,3],[1,2]),
        (2,[1,2,3],[1,4]),
        (3,NULL,[1]),
        (4,[1,2,null],NULL),
        (5,[1,2,null],[null]),
        (6,[2,3],[]);
    ~~~

2. 从该表中查询数据。

    ~~~Plain
    SELECT * FROM t1 ORDER BY c0;
    +------+------------+----------+
    | c0   | c1         | c2       |
    +------+------------+----------+
    |    1 | [1,2,3]    | [1,2]    |
    |    2 | [1,2,3]    | [1,4]    |
    |    3 | NULL       | [1]      |
    |    4 | [1,2,null] | NULL     |
    |    5 | [1,2,null] | [null]   |
    |    6 | [2,3]      | []       |
    +------+------------+----------+
    ~~~

3. 检查每行的`c2`是否是对应行的`c1`的子集。

    ~~~Plaintext
    SELECT c0, c1, c2, array_contains_all(c1, c2) FROM t1 ORDER BY c0;
    +------+------------+----------+----------------------------+
    | c0   | c1         | c2       | array_contains_all(c1, c2) |
    +------+------------+----------+----------------------------+
    |    1 | [1,2,3]    | [1,2]    |                          1 |
    |    2 | [1,2,3]    | [1,4]    |                          0 |
    |    3 | NULL       | [1]      |                       NULL |
    |    4 | [1,2,null] | NULL     |                       NULL |
    |    5 | [1,2,null] | [null]   |                          1 |
    |    6 | [2,3]      | []       |                          1 |
    +------+------------+----------+----------------------------+
    ~~~

在输出中：

对于第1行，`c2`是`c1`的子集，返回1。

对于第2行，`c2`不是`c1`的子集，返回0。

对于第3行，`c1`是NULL，返回NULL。

对于第4行，`c2`是NULL，返回NULL。

对于第5行，两个数组包含`null`，将`null`处理为普通值，返回1。

对于第6行，`c2`是空数组，被视为`c1`的子集。因此，返回1。
