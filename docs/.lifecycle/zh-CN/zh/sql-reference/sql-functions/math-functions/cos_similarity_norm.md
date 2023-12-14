---
displayed_sidebar: "汉语"
---

# cosine_similarity_norm

## 功能

计算两个归一化向量的余弦夹角，以评估向量之间的相似度。此函数假定向量已经进行过归一化处理。如果在计算之前需要对向量进行归一化，请使用 [cosine_similarity](./cos_similarity.md) 函数。

相似度的取值范围从 -1 到 1。如果两个向量的夹角为 0°，即两个向量的方向完全相同时，相似度为 1；如果夹角为 180°，即两个向量方向完全相反时，相似度为 -1；如果夹角为 90°，即两向量方向正交时，相似度为 0。

值越接近 1 表示两个向量的方向越相似；越接近 -1 表示方向越相反；越接近 0 表示两向量近似正交。

余弦相似度常用于测量文本、视频等的相似性。

## 语法

```Haskell
cosine_similarity_norm(a, b)
```

## 参数说明

`a` 和 `b` 是进行比较的两个向量，必须具有相同的维度。它们的值必须是 `Array<float>` 类型，即数组中的元素只能是 FLOAT 类型。注意，两个数组必须包含相同数量的元素，否则函数会返回错误。

## 返回值说明

返回 FLOAT 类型的结果，其取值范围是 [-1, 1]。如果输入参数为 NULL 或者参数类型不正确，函数将返回错误。

## 示例

1. 创建一个表来存储向量数据，并向表中插入数据。

    ```SQL
      CREATE TABLE t1_similarity 
      (id int, data array<float>)
      DISTRIBUTED BY HASH(id);
      
      INSERT INTO t1_similarity VALUES
      (1, array<float>[0.1, 0.2, 0.3]), 
      (2, array<float>[0.2, 0.1, 0.3]), 
      (3, array<float>[0.3, 0.2, 0.1]);
    ```

2. 使用 `cosine_similarity_norm` 函数计算 `data` 列中每个数组与数组 `[0.1, 0.2, 0.3]` 之间的相似度，并按降序排列结果。

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