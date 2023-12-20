---
displayed_sidebar: English
---

# 最大值依据

## 描述

返回与 y 的最大值相关联的 x 的值。

例如，SELECT max_by(subject, exam_result) FROM exam; 用于返回考试成绩最高的科目。

该函数从 v2.5 版本开始支持。

## 语法

```Haskell
max_by(x,y)
```

## 参数

- x：任意类型的表达式。
- y：可以进行排序的类型的表达式。

## 返回值

返回一个与 x 类型相同的值。

## 使用说明

- y 必须是可以排序的类型。如果使用了不可排序的 y 类型，如 bitmap 或 hll，则会返回错误。
- 如果 y 包含 null 值，则忽略对应 null 值的行。
- 如果有多个 x 的值具有相同的最大 y 值，该函数返回第一个遇到的 x 的值。

## 示例

1. 创建一张名为 exam 的表。

   ```SQL
   CREATE TABLE exam (
       subject_id INT,
       subject STRING,
       exam_result INT
   ) DISTRIBUTED BY HASH(`subject_id`);
   ```

2. 向这张表中插入值，并从中查询数据。

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

3. 获取最高分的科目。物理和音乐这两门科目都有最高分 95 分，返回首先遇到的科目（物理）。

   ```Plain
   SELECT max_by(subject, exam_result) FROM exam;
   +------------------------------+
   | max_by(subject, exam_result) |
   +------------------------------+
   | physics                      |
   +------------------------------+
   1 row in set (0.01 sec)
   ```
