---
displayed_sidebar: English
---

# 使用 Lateral Join 进行列到行转换

在 ETL 处理中，列到行转换是一种常见操作。Lateral 是一个特殊的 Join 关键字，可以将行与内部子查询或表函数关联起来。通过将 Lateral 与 unnest() 结合使用，可以将一行扩展为多行。更多信息，请参见 [unnest](../sql-reference/sql-functions/array-functions/unnest.md)。

## 限制

* 目前，Lateral Join 仅与 unnest() 一起用于实现列到行转换。其他表函数和 UDTF 将在以后支持。
* 目前，Lateral Join 不支持子查询。

## 使用 Lateral Join

语法：

~~~SQL
from table_reference join [lateral] table_reference;
~~~

示例：

~~~SQL
SELECT student, score
FROM tests
CROSS JOIN LATERAL UNNEST(scores) AS t (score);

SELECT student, score
FROM tests, UNNEST(scores) AS t (score);
~~~

这里的第二种语法是第一种语法的简化版本，可以使用 UNNEST 关键字省略 Lateral 关键字。UNNEST 关键字是一个表函数，用于将数组转换为多行。与 Lateral Join 一起，它可以实现常见的行扩展逻辑。

> **注意**
>
> 如果要对多列执行 unnest，必须为每一列指定一个别名，例如，`select v1, t1.unnest as v2, t2.unnest as v3 from lateral_test, unnest(v2) t1, unnest(v3) t2;`。

StarRocks 支持 BITMAP、STRING、ARRAY 和 Column 之间的类型转换。
![Lateral Join 中的一些类型转换](../assets/lateral_join_type_conversion.png)

## 使用示例

结合 unnest()，您可以实现以下列到行转换的功能：

### 将字符串展开为多行

1. 创建一个表并向其中插入数据。

    ~~~SQL
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
    ~~~

2. 展开扩容前的数据。

    ~~~Plain Text
    select * from lateral_test2;

    +------+-------+
    | v1   | v2    |
    +------+-------+
    |    1 | 1,2,3 |
    |    2 | 1,3   |
    +------+-------+
    ~~~

3. 将 `v2` 展开为多行。

    ~~~Plain Text
    -- 对单列执行 unnest。

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

    -- 对多列执行 unnest。必须为每个操作指定一个别名。

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
    ~~~

### 将数组展开为多行

 **从 v2.5 开始，unnest() 可以接受多个不同类型和长度的数组。** 更多信息，请参见 [unnest()](../sql-reference/sql-functions/array-functions/unnest.md)。

1. 创建一个表并向其中插入数据。

    ~~~SQL
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
    ~~~

2. 展开扩容前的数据。

    ~~~Plain Text
    select * from lateral_test;

    +------+------------+
    | v1   | v2         |
    +------+------------+
    |    1 | [1,2]      |
    |    2 | [1,null,3] |
    |    3 | NULL       |
    +------+------------+
    ~~~

3. 将 `v2` 展开为多行。

    ~~~Plain Text
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
    ~~~

### 展开位图数据

1. 创建一个表并向其中插入数据。

    ~~~SQL
    CREATE TABLE lateral_test3 (
    `v1` bigint(20) NULL COMMENT "",
    `v2` Bitmap BITMAP_UNION COMMENT ""
    )
    AGGREGATE KEY(v1)
    DISTRIBUTED BY HASH(`v1`);

    INSERT INTO lateral_test3 VALUES (1, bitmap_from_string('1, 2')), (2, to_bitmap(3));
    ~~~

2. 展开扩容前的数据。

    ~~~Plain Text
    select v1, bitmap_to_string(v2) from lateral_test3;

    +------+------------------------+
    | v1   | bitmap_to_string(`v2`) |
    +------+------------------------+
    |    1 | 1,2                    |
    |    2 | 3                      |
    +------+------------------------+

3. 插入新行。

    ~~~Plain Text
    insert into lateral_test3 values (1, to_bitmap(3));

    select v1, bitmap_to_string(v2) from lateral_test3;

    +------+------------------------+
    | v1   | bitmap_to_string(`v2`) |
    +------+------------------------+
    |    1 | 1,2,3                  |
    |    2 | 3                      |
    +------+------------------------+
    ~~~

4. 将 `v2` 中的数据展开为多行。

    ~~~Plain Text
    select v1,unnest from lateral_test3 , unnest(bitmap_to_array(v2));

    +------+--------+
    | v1   | unnest |
    +------+--------+
    |    1 |      1 |
    |    1 |      2 |
    |    1 |      3 |
    |    2 |      3 |
    +------+--------+
    ~~~
