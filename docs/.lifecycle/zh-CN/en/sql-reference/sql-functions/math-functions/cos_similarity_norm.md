---
displayed_sidebar: "Chinese"
---

# cos_similarity_norm

## 描述

通过计算两个标准向量之间的角度的余弦值来测量它们的相似度。向量的方向形成的角度而忽略其大小的差异。此函数假设输入向量已被标准化。如果需要在计算余弦相似性之前标准化向量，请使用[cosine_similarity](./cos_similarity.md)。

相似度在-1到1之间。向量之间的角度越小, 说明余弦相似度越大。

- 如果两个向量具有相同的方向, 它们的角度为0度, 余弦相似度为1。
- 垂直向量的角度为90度, 余弦相似度为0。
- 相反的向量的角度为180度, 余弦相似度为-1。

## 语法

```Haskell
cosine_similarity_norm(a, b)
```

## 参数

`a` 和 `b` 是要比较的向量。它们必须具有相同的维度。支持的数据类型是 `Array<float>`。这两个数组必须具有相同的元素数量, 否则将返回错误。

## 返回值

返回范围在[-1, 1]内的浮点值。如果任何输入参数为null或无效, 将报告错误。

## 示例

1. 创建一个表来存储向量并将数据插入此表中。

    ```SQL
    CREATE TABLE t1_similarity 
    (id int, data array<float>)
    DISTRIBUTED BY HASH(id);

    INSERT INTO t1_similarity VALUES
    (1, array<float>[0.1, 0.2, 0.3]), 
    (2, array<float>[0.2, 0.1, 0.3]), 
    (3, array<float>[0.3, 0.2, 0.1]);
    ```

2. 计算与`data`列中的每行与数组`[0.1, 0.2, 0.3]`的相似性, 并按降序列出结果。

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