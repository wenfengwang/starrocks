---
displayed_sidebar: English
---

# 使用横向连接进行列转行操作

列转行操作是ETL处理中的一个常见步骤。Lateral是一个特殊的Join关键词，它能够将一行数据与内部子查询或表函数关联起来。通过结合使用Lateral和`unnest()`函数，你可以把单行数据展开成多行。更多详情，请参考[unnest](../sql-reference/sql-functions/array-functions/unnest.md)函数的文档。

## 限制

* 目前，Lateral Join只能与unnest()函数一起使用，以实现列转行的操作。未来将支持其他表函数和用户定义的表函数（UDTF）。
* 目前，Lateral Join不支持子查询的使用。

## 如何使用横向连接

语法：

```SQL
from table_reference join [lateral] table_reference;
```

示例：

```SQL
SELECT student, score
FROM tests
CROSS JOIN LATERAL UNNEST(scores) AS t (score);

SELECT student, score
FROM tests, UNNEST(scores) AS t (score);
```

这里的第二个语法是第一个语法的简化版，在这个版本中，可以通过使用UNNEST关键词来省略Lateral关键词。UNNEST关键词是一个表函数，用于将数组转换成多行。结合横向连接使用时，它可以实现常见的行展开逻辑。

> **注意**
> 如果你想对多个列执行unnest操作，你必须为每个列指定一个别名，例如，`select v1, t1.unnest as v2, t2.unnest as v3 from lateral_test, unnest(v2) as t1, unnest(v3) as t2;`。

StarRocks支持在**BITMAP**、**STRING**、**ARRAY**和**Column**之间进行类型转换。
![在Lateral Join中的一些类型转换操作](../assets/lateral_join_type_conversion.png)

## 使用示例

结合unnest()函数，你可以实现以下列转行的特性：

### 将一个字符串展开成多行

1. 创建一个表并向该表插入数据。

   ```SQL
   CREATE TABLE lateral_test2 (
       `v1` bigint(20) NULL COMMENT "",
       `v2` string NULL COMMENT ""
   )
   DUPLICATE KEY(v1)
   DISTRIBUTED BY HASH(`v1`)
   PROPERTIES (
       "replication_num" = "3",
       "storage_format" = "DEFAULT"
   );
   
   INSERT INTO lateral_test2 VALUES (1, "1,2,3"), (2, "1,3");
   ```

2. 在展开前查询数据。

   ```Plain
   select * from lateral_test2;
   
   +------+-------+
   | v1   | v2    |
   +------+-------+
   |    1 | 1,2,3 |
   |    2 | 1,3   |
   +------+-------+
   ```

3. 将v2列展开成多行。

   ```Plain
   -- Perform unnest on a single column.
   
   select v1,unnest from lateral_test2, unnest(split(v2, ","));
   
   +------+--------+
   | v1   | unnest |
   +------+--------+
   |    1 | 1      |
   |    1 | 2      |
   |    1 | 3      |
   |    2 | 1      |
   |    2 | 3      |
   +------+--------+
   
   -- Perform unnest on multiple columns. You must specify an alias for each operation.
   
   select v1, t1.unnest as v2, t2.unnest as v3 from lateral_test2, unnest(split(v2, ",")) t1, unnest(split(v3, ",")) t2;
   
   +------+------+------+
   | v1   | v2   | v3   |
   +------+------+------+
   |    1 | 1    | 1    |
   |    1 | 1    | 2    |
   |    1 | 2    | 1    |
   |    1 | 2    | 2    |
   |    1 | 3    | 1    |
   |    1 | 3    | 2    |
   |    2 | 1    | 1    |
   |    2 | 1    | 3    |
   |    2 | 3    | 1    |
   |    2 | 3    | 3    |
   +------+------+------+
   ```

### 将一个数组展开成多行

**从v2.5版本开始，unnest()函数可以处理多个不同类型和长度的数组。**更多详情，请参考[unnest()函数的文档](../sql-reference/sql-functions/array-functions/unnest.md)。

1. 创建一个表并向该表插入数据。

   ```SQL
   CREATE TABLE lateral_test (
       `v1` bigint(20) NULL COMMENT "",
       `v2` ARRAY NULL COMMENT ""
   ) 
   DUPLICATE KEY(v1)
   DISTRIBUTED BY HASH(`v1`)
   PROPERTIES (
       "replication_num" = "3",
       "storage_format" = "DEFAULT"
   );
   
   INSERT INTO lateral_test VALUES (1, [1,2]), (2, [1, null, 3]), (3, null);
   ```

2. 在展开前查询数据。

   ```Plain
   select * from lateral_test;
   
   +------+------------+
   | v1   | v2         |
   +------+------------+
   |    1 | [1,2]      |
   |    2 | [1,null,3] |
   |    3 | NULL       |
   +------+------------+
   ```

3. 将v2列展开成多行。

   ```Plain
   select v1,v2,unnest from lateral_test , unnest(v2) ;
   
   +------+------------+--------+
   | v1   | v2         | unnest |
   +------+------------+--------+
   |    1 | [1,2]      |      1 |
   |    1 | [1,2]      |      2 |
   |    2 | [1,null,3] |      1 |
   |    2 | [1,null,3] |   NULL |
   |    2 | [1,null,3] |      3 |
   +------+------------+--------+
   ```

### 展开Bitmap数据

1. 创建一个表并向该表插入数据。

   ```SQL
   CREATE TABLE lateral_test3 (
   `v1` bigint(20) NULL COMMENT "",
   `v2` Bitmap BITMAP_UNION COMMENT ""
   )
   AGGREGATE KEY(v1)
   DISTRIBUTED BY HASH(`v1`);
   
   INSERT INTO lateral_test3 VALUES (1, bitmap_from_string('1, 2')), (2, to_bitmap(3));
   ```

2. 在展开前查询数据。

   ```Plain
   select v1, bitmap_to_string(v2) from lateral_test3;
   
   +------+------------------------+
   | v1   | bitmap_to_string(`v2`) |
   +------+------------------------+
   |    1 | 1,2                    |
   |    2 | 3                      |
   +------+------------------------+
   
   ```

3. 插入一条新数据。

   ```Plain
   insert into lateral_test3 values (1, to_bitmap(3));
   
   select v1, bitmap_to_string(v2) from lateral_test3;
   
   +------+------------------------+
   | v1   | bitmap_to_string(`v2`) |
   +------+------------------------+
   |    1 | 1,2,3                  |
   |    2 | 3                      |
   +------+------------------------+
   ```

4. 将v2列中的数据展开成多行。

   ```Plain
   select v1,unnest from lateral_test3 , unnest(bitmap_to_array(v2));
   
   +------+--------+
   | v1   | unnest |
   +------+--------+
   |    1 |      1 |
   |    1 |      2 |
   |    1 |      3 |
   |    2 |      3 |
   +------+--------+
   ```
