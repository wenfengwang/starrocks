---
displayed_sidebar: English
---

# min_by

## 描述

返回与最小值 `y` 关联的值 `x`。

例如，`SELECT min_by(subject, exam_result) FROM exam;` 是为了返回具有最低考试分数的科目。

此功能从 v2.5 版本开始支持。

## 语法

```Haskell
min_by(x,y)
```

## 参数

- `x`：任意类型的表达式。
- `y`：可排序的类型的表达式。

## 返回值

返回与 `x` 相同类型的值。

## 使用说明

- `y` 必须是可排序的类型。如果使用了不可排序的 `y` 类型，比如 `bitmap` 或 `hll`，将返回错误。
- 如果 `y` 包含空值，则忽略对应空值的行。
- 如果有多个 `x` 具有相同的最小 `y` 值，则此函数返回遇到的第一个 `x` 值。

## 例子

1. 创建一个名为 `exam` 的表。

    ```SQL
    CREATE TABLE exam (
        subject_id INT,
        subject STRING,
        exam_result INT
    ) DISTRIBUTED BY HASH(`subject_id`);
    ```

2. 向该表中插入数值，并从该表中查询数据。

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

3. 获取分数最低的科目。
   返回分数最低的科目 `english`，分数为 `70`。

    ```Plain
    SELECT min_by(subject, exam_result) FROM exam;
    +------------------------------+
    | min_by(subject, exam_result) |
    +------------------------------+
    | english                      |
    +------------------------------+
    1 row in set (0.01 sec)
    ```
