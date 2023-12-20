---
displayed_sidebar: English
---

# min_by 函数说明

## 描述

返回与 y 的最小值相关联的 x 的值。

例如，执行 SELECT min_by(subject, exam_result) FROM exam; 将返回得分最低的科目。

该函数从 v2.5 版本开始支持。

## 语法

```Haskell
min_by(x,y)
```

## 参数

- x：任意类型的表达式。
- y：可以进行排序的类型的表达式。

## 返回值

返回一个与 x 类型相同的值。

## 使用须知

- y 必须是可以排序的类型。如果 y 使用的是不可排序的类型，比如 bitmap 或 hll，将会报错。
- 如果 y 中包含 null 值，那么包含 null 值的行将被忽略。
- 如果有多个 x 的值对应于 y 的最小值，那么这个函数将返回第一个遇到的 x 的值。

## 示例

1. 创建一个名为 exam 的表。

   ```SQL
   CREATE TABLE exam (
       subject_id INT,
       subject STRING,
       exam_result INT
   ) DISTRIBUTED BY HASH(`subject_id`);
   ```

2. 向该表插入数据并查询。

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

3. 获取得分最低的科目。返回得分最低的英语科目，其分数为70分。

   ```Plain
   SELECT min_by(subject, exam_result) FROM exam;
   +------------------------------+
   | min_by(subject, exam_result) |
   +------------------------------+
   | english                      |
   +------------------------------+
   1 row in set (0.01 sec)
   ```
