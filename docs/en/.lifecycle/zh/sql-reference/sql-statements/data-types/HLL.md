---
displayed_sidebar: English
---

# HLL（HyperLogLog）

## 描述

HLL 用于[近似计数不同](../../../using_starrocks/Using_HLL.md)。

HLL 支持基于 HyperLogLog 算法的程序开发。它用于存储 HyperLogLog 计算过程的中间结果。它只能用作表的值列类型。HLL 通过聚合来减少数据量，从而加快查询过程。估计结果可能有 1% 的偏差。

HLL 列是根据导入的数据或来自其他列的数据生成的。导入数据时，[hll_hash](../../sql-functions/aggregate-functions/hll_hash.md) 函数指定将使用哪一列生成 HLL 列。HLL 通常用于替换 COUNT DISTINCT 并使用 rollup 快速计算唯一视图（UV）。

HLL 使用的存储空间由哈希值中的非重复值确定。存储空间因三个条件而异：

- HLL 为空。HLL 中不插入任何值，存储成本最低，为 80 字节。
- HLL 中的非重复哈希值数小于或等于 160。最高存储成本为 1360 字节（80 + 160 * 8 = 1360）。
- HLL 中的非重复哈希值数大于 160。存储成本固定为 16,464 字节（80 + 16 * 1024 = 16464）。

在实际业务场景中，数据量和数据分布会影响查询的内存使用率和近似结果的准确性。您需要考虑以下两个因素：

- 数据量：HLL 返回近似值。数据量越大，结果越准确。数据量越小，偏差越大。
- 数据分布：在 GROUP BY 数据量大、基数维数高的情况下，数据计算会占用更多的内存。在这种情况下，不建议使用 HLL。当您对低基数维度列执行 no-group-by count distinct 或 GROUP BY 时，建议这样做。
- 查询粒度：如果查询数据粒度较大，建议使用聚合表或物化视图对数据进行预聚合，以减少数据量。

## 相关功能

- [HLL_UNION_AGG（hll）：](../../sql-functions/aggregate-functions/hll_union_agg.md)该函数是一个聚合函数，用于估计满足条件的所有数据的基数。这也可用于分析函数。它只支持默认窗口，不支持窗口子句。

- [HLL_RAW_AGG（hll）：](../../sql-functions/aggregate-functions/hll_raw_agg.md)该函数是一个聚合函数，用于聚合 hll 类型的字段，并以 hll 类型返回。

- HLL_CARDINALITY（hll）：此函数用于估计单个 hll 列的基数。

- [HLL_HASH（column_name）](../../sql-functions/aggregate-functions/hll_hash.md)：生成 HLL 列类型，用于插入或导入。请参阅有关使用导入的说明。

- [HLL_EMPTY](../../sql-functions/aggregate-functions/hll_empty.md)：这将生成空的 HLL 列，用于在插入或导入期间填充默认值。请参阅有关使用导入的说明。

## 例子

1. 创建一个包含 HLL 列 `set1` 和 `set2` 的表。

    ```sql
    create table test(
    dt date,
    id int,
    name char(10),
    province char(10),
    os char(1),
    set1 hll hll_union,
    set2 hll hll_union)
    distributed by hash(id);
    ```

2. 使用 [Stream Load](../../../loading/StreamLoad.md) 加载数据。

    ```plain text
    a. 使用表列生成 HLL 列。
    curl --location-trusted -uname:password -T data -H "label:load_1" \
        -H "columns:dt, id, name, province, os, set1=hll_hash(id), set2=hll_hash(name)"
    http://host/api/test_db/test/_stream_load

    b. 使用数据列生成 HLL 列。
    curl --location-trusted -uname:password -T data -H "label:load_1" \
        -H "columns:dt, id, name, province, sex, cuid, os, set1=hll_hash(cuid), set2=hll_hash(os)"
    http://host/api/test_db/test/_stream_load
    ```

3. 通过以下三种方式聚合数据：（如果不进行聚合，对基表的直接查询可能与使用 approx_count_distinct 一样慢）

    ```sql
    -- a. 创建一个 rollup 来聚合 HLL 列。
    alter table test add rollup test_rollup(dt, set1);

    -- b. 创建另一个表来计算 UV 并将数据插入其中。

    create table test_uv(
    dt date,
    id int
    uv_set hll hll_union)
    distributed by hash(id);

    insert into test_uv select dt, id, set1 from test;

    -- c. 创建另一个表来计算 UV。通过对其他列使用 hll_hash 插入数据并生成 HLL 列。

    create table test_uv(
    dt date,
    id int,
    id_set hll hll_union)
    distributed by hash(id);

    insert into test_uv select dt, id, hll_hash(id) from test;
    ```

4. 查询数据。HLL 列不支持直接查询其原始值。可以通过匹配函数进行查询。

    ```plain text
    a. 计算总 UV。
    select HLL_UNION_AGG(uv_set) from test_uv;

    b. 计算每天的 UV。
    select dt, HLL_CARDINALITY(uv_set) from test_uv;

    c. 计算 test 表中 set1 的聚合值。
    select dt, HLL_CARDINALITY(uv) from (select dt, HLL_RAW_AGG(set1) as uv from test group by dt) tmp;
    select dt, HLL_UNION_AGG(set1) as uv from test group by dt;
    ```
