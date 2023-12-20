---
displayed_sidebar: English
---

# 使用 HLL 进行大致去重计数

## 背景

在现实场景中，随着数据量的增加，去重数据的压力也随之增大。当数据量达到一定水平时，精确去重的成本相对较高。在这种情况下，用户通常会采用近似算法来降低计算压力。本节将介绍的 HyperLogLog（HLL）是一种近似去重算法，它具有出色的空间复杂度 O(mloglogn) 和时间复杂度 O(n)。更重要的是，计算结果的误差率可以控制在大约 1%-10%，这取决于数据集的大小和所使用的哈希函数。

## 什么是 HyperLogLog

HyperLogLog 是一种消耗极少存储空间的近似去重算法。**HLL 类型**用于实现 HyperLogLog 算法。它存储 HyperLogLog 计算的中间结果，并且只能作为数据表中的指标列类型使用。

由于 HLL 算法涉及较多数学知识，我们将通过一个实际示例来进行说明。假设我们设计了一个随机实验 A，即独立重复抛硬币直至出现正面；将第一次出现正面的硬币翻转次数记录为随机变量 X，那么：

* X=1，P(X=1)=1/2
* X=2，P(X=2)=1/4
* ...
* X=n，P(X=n)=(1/2)^n

我们使用实验 A 来构建随机实验 B，即进行 N 次实验 A 的独立重复，生成 N 个独立同分布的随机变量 X1，X2，X3，...，XN。取这些随机变量的最大值作为 Xmax。利用极大似然估计，得到 N 的估计值为 2^Xmax。

现在，我们用哈希函数对给定数据集进行上述实验的模拟：

* 实验 A：计算数据集元素的哈希值并将其转换为二进制表示。记录从二进制的最低位开始第一个 bit=1 出现的情况。
* 实验 B：对实验 B 的数据集元素重复实验 A 的过程。更新每次实验中第一个 bit=1 出现的最大位置“m”；
* 估计数据集中不重复元素的数量为 2^m。

实际上，HLL 算法根据元素哈希的低 k 位将元素划分为 K=2^k 个桶。统计从第 k+1 位开始第一个 bit=1 出现的最大值 m1、m2、...、mk，并估计每个桶中不重复元素的数量为 2^m1、2^m2、...、2^mk。数据集中不重复元素的总数是桶的数量乘以各桶中不重复元素数量的平均值：N = K * (K / (2^-m1 + 2^-m2 + ... + 2^-mK))。

HLL 通过与估计结果相乘的修正因子来使结果更精确。

请参考文章[https://gist.github.com/avibryant/8275649](https://gist.github.com/avibryant/8275649)，其中介绍了如何使用StarRocks SQL语句实现HLL去重算法。

```sql
SELECT floor((0.721 * 1024 * 1024) / (sum(pow(2, m * -1)) + 1024 - count(*))) AS estimate
FROM(select(murmur_hash3_32(c2) & 1023) AS bucket,
     max((31 - CAST(log2(murmur_hash3_32(c2) & 2147483647) AS INT))) AS m
     FROM db0.table0
     GROUP BY bucket) bucket_values
```

该算法用于去重 db0.table0 中的 col2。

* 使用哈希函数 murmur_hash3_32 计算 col2 的哈希值，结果为 32 位有符号整数。
* 使用 1024 个桶，修正系数为 0.721，取哈希值的低 10 位作为桶的索引。
* 忽略哈希值的符号位，从次高位到低位，确定第一个 bit 1 出现的位置。
* 按桶对计算出的哈希值进行分组，并使用 MAX 聚合函数找到每个桶中第一个 bit 1 出现的最大位置。
* 将聚合结果作为子查询，并将所有桶的估计平均值乘以桶的数量和修正系数。
* 注意，空桶的计数为 1。

当数据量较大时，上述算法的误差率非常低。

这是 HLL 算法的核心理念。请参阅[HyperLogLog paper](http://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf)，如果您感兴趣。

### 如何使用 HyperLogLog

1. 要使用 HyperLogLog 进行去重，您需要在创建表的语句中将目标指标列的类型设置为 HLL，并将聚合函数设置为 HLL_UNION。
2. 目前，只有聚合模型支持将 HLL 作为指标列类型。
3. 在 HLL 类型的列上使用 count distinct 时，StarRocks 会自动将其转换为 HLL_UNION_AGG 计算。

#### 示例

首先，创建一个包含 **HLL** 列的表，其中 uv 是一个聚合列，列类型为 `HLL`，聚合函数为 [HLL_UNION](../sql-reference/sql-functions/aggregate-functions/hll_union.md)。

```sql
CREATE TABLE test(
        dt DATE,
        id INT,
        uv HLL HLL_UNION
)
DISTRIBUTED BY HASH(ID);
```

* 注意：当数据量较大时，为了提高 HLL 查询的频率，最好创建相应的 rollup 表。

使用 [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) 加载数据：

```bash
curl --location-trusted -u <username>:<password> -H "label:label_1600997542287" \
    -H "column_separator:," \
    -H "columns:dt,id,user_id, uv=hll_hash(user_id)" -T /root/test.csv http://starrocks_be0:8040/api/db0/test/_stream_load
{
    "TxnId": 2504748,
    "Label": "label_1600997542287",
    "Status": "Success",
    "Message": "OK",
    "NumberTotalRows": 5,
    "NumberLoadedRows": 5,
    "NumberFilteredRows": 0,
    "NumberUnselectedRows": 0,
    "LoadBytes": 120,
    "LoadTimeMs": 46,
    "BeginTxnTimeMs": 0,
    "StreamLoadPutTimeMs": 1,
    "ReadDataTimeMs": 0,
    "WriteDataTimeMs": 29,
    "CommitAndPublishTimeMs": 14
}
```

Broker Load 模式：

```sql
LOAD LABEL test_db.label
 (
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/file")
    INTO TABLE `test`
    COLUMNS TERMINATED BY ","
    (dt, id, user_id)
    SET (
      uv = HLL_HASH(user_id)
    )
 );
```

查询数据

* HLL 列不允许直接查询其原始值，使用函数 [HLL_UNION_AGG](../sql-reference/sql-functions/aggregate-functions/hll_union_agg.md) 进行查询。
* 要找到总 UV，

SELECT HLL_UNION_AGG(uv) FROM test;

这个语句等同于

SELECT COUNT(DISTINCT uv) FROM test;

* 查询每日 UV

SELECT COUNT(DISTINCT uv) FROM test GROUP BY ID;

### 注意事项

在 Bitmap 和 HLL 之间如何选择？如果数据集的基数在百万或千万级别，并且您有几十台机器，请使用 count distinct。如果基数在亿级，并且需要精确去重，请使用 Bitmap；如果可以接受近似去重，请使用 HLL 类型。

Bitmap 仅支持 TINYINT、SMALLINT、INT 和 BIGINT 数据类型。请注意，不支持 LARGEINT。对于需要去重的其他类型数据集，需要构建字典以将原始类型映射到整数类型。构建字典较为复杂，需要在数据量、更新频率、查询效率、存储等因素之间进行权衡。HLL 不需要字典，但它需要支持哈希函数的相应数据类型。即使在不支持 HLL 的分析系统中，也可以使用哈希函数和 SQL 实现 HLL 去重。

对于常规列，用户可以使用 NDV 函数进行近似去重。此函数返回 COUNT(DISTINCT col) 结果的近似聚合值，其底层实现将数据存储类型转换为 HyperLogLog 类型进行计算。NDV 函数在计算时消耗资源较多，因此不适合高并发场景。

如果想进行用户行为分析，可以考虑使用 IntersectCount 或自定义 UDAF。
