---
displayed_sidebar: English
---

# cos_similarity_norm

## 描述

通过计算两个归一化向量之间的夹角余弦值来衡量它们的相似性。这个夹角是由向量的方向形成的，而它们的大小差异被忽略。该函数假定输入向量已经过归一化。如果需要在计算余弦相似度之前对向量进行归一化，请使用 [cosine_similarity](./cos_similarity.md)。

相似度的取值范围在 -1 到 1 之间。向量之间的夹角越小，表示余弦相似度越大。

- 如果两个向量的方向相同，则它们的夹角为 0 度，余弦相似度为 1。
- 垂直向量的夹角为 90 度，余弦相似度为 0。
- 相反向量的夹角为 180 度，余弦相似度为 -1。

## 语法

```Haskell
cosine_similarity_norm(a, b)
```

## 参数

`a` 和 `b` 是要比较的向量。它们必须具有相同的维度。支持的数据类型是 `Array<float>`。这两个数组必须具有相同数量的元素，否则将返回错误。

## 返回值

返回一个范围在 [-1, 1] 之间的浮点数。如果任何输入参数为 null 或无效，则会报错。

## 例子

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
