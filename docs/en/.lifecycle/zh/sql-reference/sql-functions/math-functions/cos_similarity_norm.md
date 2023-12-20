---
displayed_sidebar: English
---

# cos_similarity_norm

## 描述

通过计算两个已归一化向量之间角度的余弦值来衡量它们的相似性。角度是由向量的方向决定的，而忽略它们的大小差异。此函数假设输入向量已经归一化。如果你需要在计算余弦相似度之前对向量进行归一化，请使用 [cosine_similarity](./cos_similarity.md)。

相似度介于 -1 和 1 之间。向量之间角度越小，余弦相似度越大。

- 如果两个向量方向相同，则它们之间的角度为 0 度，余弦相似度为 1。
- 垂直向量的角度为 90 度，余弦相似度为 0。
- 相反方向的向量角度为 180 度，余弦相似度为 -1。

## 语法

```Haskell
cosine_similarity_norm(a, b)
```

## 参数

`a` 和 `b` 是要比较的向量。它们必须具有相同的维度。支持的数据类型是 `Array<float>`。两个数组必须具有相同数量的元素。否则，将返回错误。

## 返回值

返回一个范围在 [-1, 1] 内的 FLOAT 值。如果任何输入参数为空或无效，则报告错误。

## 示例

1. 创建一个表来存储向量，并向该表中插入数据。

   ```SQL
   CREATE TABLE t1_similarity 
   (id int, data array<float>)
   DISTRIBUTED BY HASH(id);
   
   INSERT INTO t1_similarity VALUES
   (1, array<float>[0.1, 0.2, 0.3]), 
   (2, array<float>[0.2, 0.1, 0.3]), 
   (3, array<float>[0.3, 0.2, 0.1]);
   ```

2. 计算 `data` 列中每一行与数组 `[0.1, 0.2, 0.3]` 的相似度，并按降序列出结果。

   ```Plain
   SELECT id, data, cosine_similarity_norm([0.1, 0.2, 0.3], data) as dist
   FROM t1_similarity 
   ORDER BY dist DESC;
   
   +------+---------------+------------+
   | id   | data          | dist       |
   +------+---------------+------------+
   |    1 | [0.1,0.2,0.3] | 0.14000002 |
   |    2 | [0.2,0.1,0.3] | 0.13000001 |
   |    3 | [0.3,0.2,0.1] | 0.10000001 |
   +------+---------------+------------+
   ```