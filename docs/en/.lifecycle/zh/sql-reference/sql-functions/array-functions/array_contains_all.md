---
displayed_sidebar: English
---

# array_contains_all

## 描述

检查 `arr1` 是否包含 `arr2` 的所有元素，即 `arr2` 是否是 `arr1` 的子集。如果是，则返回 1。如果不是，则返回 0。

## 语法

```Haskell
BOOLEAN array_contains_all(arr1, arr2)
```

## 参数

`arr1` 和 `arr2`：要比较的两个数组。此语法检查 `arr2` 是否是 `arr1` 的子集。

两个数组中元素的数据类型必须相同。对于 StarRocks 支持的数组元素的数据类型，请参见 [ARRAY](../../../sql-reference/sql-statements/data-types/Array.md)。

## 返回值

返回 BOOLEAN 类型的值。

如果 `arr2` 是 `arr1` 的子集，则返回 1。否则，返回 0。

如果两个数组中有任何一个为 NULL，则返回 NULL。

## 使用说明

- 如果数组包含 `null` 元素，则将 `null` 作为值处理。

- 空数组是任何数组的子集。

- 两个数组中的元素可以有不同的顺序。

## 示例

1. 创建一个名为 `t1` 的表并向该表中插入数据。

   ```SQL
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
   ```

2. 从该表中查询数据。

   ```Plain
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
   ```

3. 检查 `c2` 的每一行是否是对应 `c1` 行的子集。

   ```Plaintext
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
   ```

在输出中：

对于第 1 行，`c2` 是 `c1` 的子集，并且返回 1。

对于第 2 行，`c2` 不是 `c1` 的子集，因此返回 0。

对于第 3 行，`c1` 为 NULL，并且返回 NULL。

对于第 4 行，`c2` 为 NULL，并且返回 NULL。

对于第 5 行，两个数组都包含 `null`，并且将 `null` 作为普通值处理，返回 1。

对于第 6 行，`c2` 是一个空数组，被视为 `c1` 的子集。因此，返回 1。