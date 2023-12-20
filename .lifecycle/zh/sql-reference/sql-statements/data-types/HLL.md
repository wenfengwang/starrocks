---
displayed_sidebar: English
---

# HLL（HyperLogLog 超对数）

## 描述

HLL 用于[近似计数不同值](../../../using_starrocks/Using_HLL.md)。

HLL 支持开发基于 HyperLogLog 算法的程序。它用于存储 HyperLogLog 计算过程中的中间结果。它只能作为表的值列类型使用。HLL 通过聚合减少数据量，以加速查询过程。估计结果可能存在 1% 的偏差。

HLL 列是基于导入数据或其他列数据生成的。在导入数据时，[hll_hash](../../sql-functions/aggregate-functions/hll_hash.md) 函数指定哪一列将用于生成 HLL 列。HLL 经常用来替代 COUNT DISTINCT，并通过 rollup 快速计算独立访客数（UV）。

HLL 使用的存储空间取决于哈希值中的不同值数量。存储空间根据以下三种情况变化：

- 1. HLL 为空。未向 HLL 中插入任何值时，存储成本最低，为 80 字节。
- 2. HLL 中的不同哈希值数量小于或等于 160 时，最高存储成本为 1360 字节（80 + 160 * 8 = 1360）。
- 3. HLL 中的不同哈希值数量大于 160 时，存储成本固定为 16,464 字节（80 + 16 * 1024 = 16464）。

在实际业务场景中，数据量和数据分布会影响查询的内存使用和近似结果的准确性。您需要考虑这两个因素：

- 数据量：HLL 返回的是一个近似值。数据量越大，结果越准确。数据量越小，偏差越大。
- 数据分布：在数据量大且 GROUP BY 使用高基数维度列的情况下，数据计算将占用更多内存。在这种情况下不建议使用 HLL。当您在低基数维度列上执行无分组的 count distinct 或 GROUP BY 时，推荐使用 HLL。
- 查询粒度：如果您以较大的查询粒度查询数据，我们建议您使用聚合表或物化视图预先聚合数据，以减少数据量。

## 相关函数

- [HLL_UNION_AGG(hll)](../../sql-functions/aggregate-functions/hll_union_agg.md)：这是一个聚合函数，用于估计满足条件的所有数据的基数。它也可以用于分析功能。它仅支持默认窗口，不支持窗口子句。

- [HLL_RAW_AGG(hll)](../../sql-functions/aggregate-functions/hll_raw_agg.md)：这是一个聚合函数，用于聚合 hll 类型的字段，并返回 hll 类型。

- HLL_CARDINALITY(hll)：这个函数用于估计单个 hll 列的基数。

- [HLL_HASH(column_name)](../../sql-functions/aggregate-functions/hll_hash.md)：此函数生成 HLL 列类型，用于插入或导入操作。参见导入使用说明。

- [HLL_EMPTY](../../sql-functions/aggregate-functions/hll_empty.md)：此函数生成空的 HLL 列，用于在插入或导入时填充默认值。参见导入使用说明。

## 示例

1. 创建一个包含 HLL 列 set1 和 set2 的表。

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

   ```plain
   a. Use table columns to generate an HLL column.
   curl --location-trusted -uname:password -T data -H "label:load_1" \
       -H "columns:dt, id, name, province, os, set1=hll_hash(id), set2=hll_hash(name)"
   http://host/api/test_db/test/_stream_load
   
   b. Use data columns to generate an HLL column.
   curl --location-trusted -uname:password -T data -H "label:load_1" \
       -H "columns:dt, id, name, province, sex, cuid, os, set1=hll_hash(cuid), set2=hll_hash(os)"
   http://host/api/test_db/test/_stream_load
   ```

3. 以以下三种方式聚合数据：（若不进行聚合，直接在基表上查询可能会像使用 approx_count_distinct 一样慢）

   ```sql
   -- a. Create a rollup to aggregate HLL column.
   alter table test add rollup test_rollup(dt, set1);
   
   -- b. Create another table to calculate uv and insert data into it
   
   create table test_uv(
   dt date,
   id int
   uv_set hll hll_union)
   distributed by hash(id);
   
   insert into test_uv select dt, id, set1 from test;
   
   -- c. Create another table to calculate UV. Insert data and generate HLL column by testing other columns through hll_hash.
   
   create table test_uv(
   dt date,
   id int,
   id_set hll hll_union)
   distributed by hash(id);
   
   insert into test_uv select dt, id, hll_hash(id) from test;
   ```

4. 查询数据。HLL 列不支持直接查询其原始值。它可以通过匹配的函数进行查询。

   ```plain
   a. Calculate the total UV.
   select HLL_UNION_AGG(uv_set) from test_uv;
   
   b. Calculate the UV for each day.
   select dt, HLL_CARDINALITY(uv_set) from test_uv;
   
   c. Calculate the aggregation value of set1 in the test table.
   select dt, HLL_CARDINALITY(uv) from (select dt, HLL_RAW_AGG(set1) as uv from test group by dt) tmp;
   select dt, HLL_UNION_AGG(set1) as uv from test group by dt;
   ```
