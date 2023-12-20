---
displayed_sidebar: English
---

# 余弦相似度

## 描述

通过计算两个向量之间角度的余弦来衡量它们的相似度。这个角度是由向量的方向形成的，而忽略它们的大小差异。

相似度的范围是 -1 到 1。向量之间的角度越小，余弦相似度越大。

- 如果两个向量方向相同，它们的夹角为 0 度，余弦相似度为 1。
- 如果向量垂直，它们的夹角为 90 度，余弦相似度为 0。
- 如果向量方向相反，它们的夹角为 180 度，余弦相似度为 -1。

`cosine_similarity` 常用于评估文本和视频的相似性。

此函数在测量余弦相似度之前会对向量进行标准化。如果输入向量已经标准化，您可以使用 [cosine_similarity_norm](./cos_similarity_norm.md)。

## 语法

```Haskell
cosine_similarity(a, b)
```

## 参数

`a` 和 `b` 是要比较的向量。它们必须具有相同的维度。支持的数据类型是 `Array<float>`。两个数组必须具有相同数量的元素。否则，会返回错误。

## 返回值

返回一个范围在 [-1, 1] 内的 FLOAT 值。如果任何输入参数为空或无效，将报告错误。

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
   SELECT id, data, cosine_similarity([0.1, 0.2, 0.3], data) AS dist
   FROM t1_similarity 
   ORDER BY dist DESC;
   
   +------+---------------+-----------+
   | id   | data          | dist      |
   +------+---------------+-----------+
   |    1 | [0.1,0.2,0.3] | 0.9999999 |
   |    2 | [0.2,0.1,0.3] | 0.9285713 |
   |    3 | [0.3,0.2,0.1] | 0.7142856 |
   +------+---------------+-----------+
   ```