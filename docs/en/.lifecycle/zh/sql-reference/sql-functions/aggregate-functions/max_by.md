---
displayed_sidebar: English
---

# max_by

## 描述

返回与最大值的`y`关联的`x`的值。

例如，`SELECT max_by(subject, exam_result) FROM exam;` 是为了返回具有最高考试分数的科目。

此功能从v2.5开始支持。

## 语法

```Haskell
max_by(x,y)
```

## 参数

- `x`：任何类型的表达式。
- `y`：可以排序的类型的表达式。

## 返回值

返回与`x`相同类型的值。

## 使用说明

- `y`必须是可排序的类型。如果使用了不可排序的`y`类型，例如`bitmap`或`hll`，则会返回错误。
- 如果`y`包含null值，则会忽略对应null值的行。
- 如果有多个`x`的值具有相同的最大`y`值，则此函数会返回遇到的第一个`x`的值。

## 例子

1. 创建一个`exam`表。

    ```SQL
    CREATE TABLE exam (
        subject_id INT,
        subject STRING,
        exam_result INT
    ) DISTRIBUTED BY HASH(`subject_id`);
    ```

2. 向该表中插入值并从该表中查询数据。

    ```SQL
    insert into exam values
    (1,'math',90),
    (2,'english',70),
    (3,'physics',95),
    (4,'chemistry',85),
    (5,'music',95),
    (6,'biology',null);

    select * from exam order by subject_id;
    +------------+-----------+-------------+
    | subject_id | subject   | exam_result |
    +------------+-----------+-------------+
    |          1 | math      |          90 |
    |          2 | english   |          70 |
    |          3 | physics   |          95 |
    |          4 | chemistry |          85 |
    |          5 | music     |          95 |
    |          6 | biology   |        null |
    +------------+-----------+-------------+
    6 rows in set (0.03 sec)
    ```

3. 获取最高分的科目。
   两个科目`physics`和`music`具有相同的最高分`95`，并返回遇到的第一个科目（`physics`）。

    ```Plain
    SELECT max_by(subject, exam_result) FROM exam;
    +------------------------------+
    | max_by(subject, exam_result) |
    +------------------------------+
    | physics                      |
    +------------------------------+
    1 row in set (0.01 sec)
    ```
