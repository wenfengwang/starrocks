---
displayed_sidebar: English
---

# min_by

## 描述

返回与 `y` 的最小值关联的 `x` 值。

例如，`SELECT min_by(subject, exam_result) FROM exam;` 用于返回考试成绩最低的科目。

该函数从 v2.5 版本开始支持。

## 语法

```Haskell
min_by(x,y)
```

## 参数

- `x`：任何类型的表达式。
- `y`：可排序类型的表达式。

## 返回值

返回一个与 `x` 类型相同的值。

## 使用说明

- `y` 必须是可排序的类型。如果使用的 `y` 类型不可排序，如 `bitmap` 或 `hll`，将返回错误。
- 如果 `y` 包含空值，则忽略对应空值的行。
- 如果有多个 `x` 的值与 `y` 的最小值相同，则此函数返回首次遇到的 `x` 的值。

## 示例

1. 创建一个表 `exam`。

   ```SQL
   CREATE TABLE exam (
       subject_id INT,
       subject STRING,
       exam_result INT
   ) DISTRIBUTED BY HASH(`subject_id`);
   ```

2. 向该表插入值并查询数据。

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

3. 获取成绩最低的科目。
返回成绩最低的科目 `english`，分数为 `70`。

   ```Plain
   SELECT min_by(subject, exam_result) FROM exam;
   +------------------------------+
   | min_by(subject, exam_result) |
   +------------------------------+
   | english                      |
   +------------------------------+
   1 row in set (0.01 sec)
   ```