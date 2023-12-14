---
displayed_sidebar: "Chinese"
---

# cos_similarity

## 描述

通过计算两个向量之间的夹角的余弦值来衡量它们的相似度。夹角是由向量的方向形成的，而它们的大小差异被忽略。

相似度介于-1和1之间。向量之间的夹角越小，余弦相似度就越大。

- 如果两个向量的方向相同，它们的夹角为0度，余弦相似度为1。
- 垂直向量的夹角为90度，余弦相似度为0。
- 对向量的夹角为180度，余弦相似度为-1。

`cosine_similarity`通常用于评估文本和视频的相似度。

在测量余弦相似度之前，此函数会对向量进行归一化处理。如果输入向量已经被归一化，可以使用[cos_similarity_norm](./cos_similarity_norm.md)。

## 语法

```Haskell
cosine_similarity(a, b)
```

## 参数

`a` 和 `b` 是要比较的向量。它们必须具有相同的维度。支持的数据类型是 `Array<float>`。这两个数组必须具有相同数量的元素。否则，将返回错误。

## 返回值

返回一个范围在[-1, 1]内的浮点数。如果任何输入参数为null或无效，则会报告错误。

## 示例

1. 创建一个表来存储向量并插入数据到该表。

    ```SQL
    CREATE TABLE t1_similarity 
    (id int, data array<float>)
    DISTRIBUTED BY HASH(id);

    INSERT INTO t1_similarity VALUES
    (1, array<float>[0.1, 0.2, 0.3]), 
    (2, array<float>[0.2, 0.1, 0.3]), 
    (3, array<float>[0.3, 0.2, 0.1]);
    ```

2. 计算`data`列中每一行与数组`[0.1, 0.2, 0.3]`的相似度，并按降序列出结果。

    ```Plain
    SELECT id, data, cosine_similarity([0.1, 0.2, 0.3], data) as dist
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