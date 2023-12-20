---
displayed_sidebar: English
---

# HLL（HyperLogLog）

## 描述

HLL 用于[近似计数去重](../../../using_starrocks/Using_HLL.md)。

HLL 支持基于 HyperLogLog 算法的程序开发。它用于存储 HyperLogLog 计算过程中的中间结果。它只能用作表的值列类型。HLL 通过聚合减少数据量以加速查询过程。估计结果可能存在 1% 的偏差。

HLL 列是基于导入数据或其他列的数据生成的。导入数据时，[hll_hash](../../sql-functions/aggregate-functions/hll_hash.md) 函数指定将使用哪一列来生成 HLL 列。HLL 常用于替代 COUNT DISTINCT，并通过 rollup 快速计算唯一访问量（UV）。

HLL 使用的存储空间取决于哈希值中的不同值数量。存储空间根据三种情况变化：

- HLL 为空。HLL 中未插入任何值，存储成本最低，为 80 字节。
- HLL 中不同哈希值的数量小于或等于 160。最高存储成本为 1360 字节（80 + 160 * 8 = 1360）。
- HLL 中不同哈希值的数量大于 160。存储成本固定为 16,464 字节（80 + 16 * 1024 = 16464）。

在实际业务场景中，数据量和数据分布会影响查询的内存使用和近似结果的准确性。您需要考虑这两个因素：

- 数据量：HLL 返回一个近似值。数据量越大，结果越准确。数据量越小，偏差越大。
- 数据分布：在数据量大且 GROUP BY 的维度列基数高的情况下，数据计算会占用更多内存。在这种情况下不建议使用 HLL。当您在低基数维度列上执行无 GROUP BY 的 count distinct 或 GROUP BY 时，推荐使用 HLL。
- 查询粒度：如果您在较大的查询粒度下查询数据，建议您使用聚合表或物化视图预先聚合数据，以减少数据量。

## 相关函数

- [HLL_UNION_AGG(hll)](../../sql-functions/aggregate-functions/hll_union_agg.md)：这是一个聚合函数，用于估计满足条件的所有数据的基数。它也可以用于分析函数。它只支持默认窗口，不支持窗口子句。

- [HLL_RAW_AGG(hll)](../../sql-functions/aggregate-functions/hll_raw_agg.md)：这是一个聚合函数，用于聚合 hll 类型的字段，并返回 hll 类型。

- HLL_CARDINALITY(hll)：这个函数用于估计单个 hll 列的基数。

- [HLL_HASH(column_name)](../../sql-functions/aggregate-functions/hll_hash.md)：这生成 HLL 列类型，用于插入或导入。参见导入使用说明。

- [HLL_EMPTY](../../sql-functions/aggregate-functions/hll_empty.md)：这生成空的 HLL 列，并用于在插入或导入时填充默认值。参见导入使用说明。

## 示例

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

   ```plain
   a. 使用表列生成 HLL 列。
   curl --location-trusted -u name:password -T data -H "label:load_1" \
       -H "columns:dt, id, name, province, os, set1=hll_hash(id), set2=hll_hash(name)"
   http://host/api/test_db/test/_stream_load
   
   b. 使用数据列生成 HLL 列。
   curl --location-trusted -u name:password -T data -H "label:load_1" \
       -H "columns:dt, id, name, province, sex, cuid, os, set1=hll_hash(cuid), set2=hll_hash(os)"
   http://host/api/test_db/test/_stream_load
   ```

3. 以下三种方式聚合数据：（不聚合，直接在基表上查询可能和使用 approx_count_distinct 一样慢）

   ```sql
   -- a. 创建 rollup 聚合 HLL 列。
   alter table test add rollup test_rollup(dt, set1);
   
   -- b. 创建另一个表来计算 UV 并插入数据
   
   create table test_uv(
   dt date,
   id int,
   uv_set hll hll_union)
   distributed by hash(id);
   
   insert into test_uv select dt, id, set1 from test;
   
   -- c. 创建另一个表来计算 UV。插入数据并通过 hll_hash 测试其他列生成 HLL 列。
   
   create table test_uv(
   dt date,
   id int,
   id_set hll hll_union)
   distributed by hash(id);
   
   insert into test_uv select dt, id, hll_hash(id) from test;
   ```

4. 查询数据。HLL 列不支持直接查询其原始值。可以通过匹配函数进行查询。

   ```plain
   a. 计算总 UV。
   select HLL_UNION_AGG(uv_set) from test_uv;
   
   b. 计算每天的 UV。
   select dt, HLL_CARDINALITY(uv_set) from test_uv;
   
   c. 计算 test 表中 set1 的聚合值。
   select dt, HLL_CARDINALITY(uv) from (select dt, HLL_RAW_AGG(set1) as uv from test group by dt) tmp;
   select dt, HLL_UNION_AGG(set1) as uv from test group by dt;
   ```