---
displayed_sidebar: English
---

# 使用 HLL 进行近似计数非重复

## 背景

在实际场景中，随着数据量的增加，删除重复数据的压力也在增加。当数据量达到一定水平时，精确的重复数据消除成本相对较高。在这种情况下，用户通常使用近似算法来减少计算压力。本节将介绍的 HyperLogLog（HLL）是一种近似重复数据消除算法，具有出色的空间复杂度O(mloglogn)和时间复杂度O(n)。更重要的是，计算结果的错误率可以控制在1%-10%左右，这取决于数据集的大小和使用的哈希函数。

## 什么是 HyperLogLog

HyperLogLog是一种近似的重复数据消除算法，它占用的存储空间非常少。用于实现HyperLogLog算法的是HLL类型。它保存了HyperLogLog计算的中间结果，只能用作数据表的指示符列类型。

由于HLL算法涉及大量的数学知识，我们将通过一个实际的例子来说明它。假设我们设计了一个随机实验A，即独立重复掷硬币，直到出现第一个正面;将第一个正面出现时的掷硬币次数记录为随机变量X，那么：

* X=1，P(X=1)=1/2
* X=2，P(X=2)=1/4
* ...
* X=n，P(X=n)=(1/2)<sup>n</sup>

我们使用测试A来构造随机测试B，即对测试A进行N次独立重复，生成N个独立同分布的随机变量X<sub>1</sub>, X<sub>2</sub>, X<sub>3</sub>, ..., X<sub>N</sub>。取随机变量的最大值为X<sub>max</sub>。利用大似然估计，N的估计值最大为2<sup>X<sub>max</sub></sup>。
<br/>

现在，我们使用哈希函数在给定数据集上模拟上述实验：

* 测试A：计算数据集元素的哈希值，并将哈希值转换为二进制表示。记录二进制中从最低位开始的bit=1的出现次数。
* 测试B：对测试B的数据集元素重复测试A的过程，更新每个测试的第一个bit=1出现的最大位置"m"；
* 将数据集中非重复元素的数量估计为m<sup>2</sup>。

事实上，HLL算法根据元素哈希的较低k位将元素划分为K=2<sup>k</sup>个桶。将第k+1位以后的第一个bit=1的最大位置计为m<sub>1</sub>, m<sub>2</sub>,..., m<sub>k</sub>，并将桶中非重复元素的数量估计为2<sup>m<sub>1</sub></sup>, 2<sup>m<sub>2</sub></sup>,..., 2<sup>m<sub>k</sub></sup>。数据集中非重复元素的数量是桶数乘以桶中非重复元素数的总平均值：N = K(K/(2<sup>\-m<sub>1</sub></sup>+2<sup>\-m<sub>2</sub></sup>,..., 2<sup>\-m<sub>K</sub></sup>)）。
<br/>

HLL将校正因子与估计结果相乘，使结果更加准确。

参考文章[https://gist.github.com/avibryant/8275649](https://gist.github.com/avibryant/8275649)，了解如何使用StarRocks SQL语句实现HLL重复数据删除算法：

~~~sql
SELECT floor((0.721 * 1024 * 1024) / (sum(pow(2, m * -1)) + 1024 - count(*))) AS estimate
FROM(select(murmur_hash3_32(c2) & 1023) AS bucket,
     max((31 - CAST(log2(murmur_hash3_32(c2) & 2147483647) AS INT))) AS m
     FROM db0.table0
     GROUP BY bucket) bucket_values
~~~

此算法用于对db0.table0的col2进行重复数据删除。

* 使用哈希函数`murmur_hash3_32`计算col2的哈希值为32位有符号整数。
* 使用1024个桶，校正因子为0.721，取哈希值的低10位作为桶的下标。
* 忽略哈希值的符号位，从次高位到最低位，并确定第一个bit=1出现的位置。
* 按照桶对计算出的哈希值进行分组，并使用`MAX`聚合查找桶中第一个bit=1出现的最大位置。
* 将聚合结果作为子查询，将所有桶估计值的总平均值乘以桶的数量和校正因子。
* 请注意，空桶计数为1。

上述算法在数据量较大时错误率非常低。

这就是HLL算法的核心思想。如果您感兴趣，请参阅[HyperLogLog论文](http://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf)。

### 如何使用 HyperLogLog

1. 要使用HyperLogLog进行重复数据消除，需要在表创建语句中将目标指标列类型设置为`HLL`，聚合函数设置为`HLL_UNION`。
2. 目前，只有聚合模型支持HLL作为指标列类型。
3. 在HLL类型的列上使用`count distinct`时，StarRocks会自动将其转换为`HLL_UNION_AGG`计算。

#### 例

首先，创建一个包含**HLL**列的表，其中uv是聚合列，列类型为`HLL`，聚合函数为[HLL_UNION](../sql-reference/sql-functions/aggregate-functions/hll_union.md)。

~~~sql
CREATE TABLE test(
        dt DATE,
        id INT,
        uv HLL HLL_UNION
)
DISTRIBUTED BY HASH(ID);
~~~

> * 注意：当数据量较大时，最好为高频HLL查询创建对应的汇总表

使用[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)加载数据：

~~~bash
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
~~~

Broker Load模式：

~~~sql
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
~~~

查询数据

* HLL列不允许直接查询其原始值，请使用函数[HLL_UNION_AGG](../sql-reference/sql-functions/aggregate-functions/hll_union_agg.md)进行查询。
* 要查找总uv，

`SELECT HLL_UNION_AGG(uv) FROM test;`

此语句等同于

`SELECT COUNT(DISTINCT uv) FROM test;`

* 查询每天的uv

`SELECT COUNT(DISTINCT uv) FROM test GROUP BY ID;`

### 注意事项

如何在位图和HLL之间进行选择？如果数据集的基数在数百万或数千万级别，并且您有几十台机器，请使用`count distinct`。如果基数在数亿级别，并且需要精确消除重复数据，请使用`Bitmap`；如果可以接受近似消除重复数据，请使用`HLL`类型。

位图仅支持TINYINT、SMALLINT、INT和BIGINT。请注意，不支持LARGEINT。对于其他类型的数据集，需要构建字典将原始类型映射到整数类型以进行重复数据消除。构建字典很复杂，需要在数据量、更新频率、查询效率、存储等问题之间进行权衡。HLL不需要字典，但需要相应的数据类型来支持哈希函数。即使在内部不支持HLL的分析系统中，仍然可以使用哈希函数和SQL来实现HLL重复数据消除。

对于常见列，用户可以使用NDV函数进行近似重复数据消除。此函数返回COUNT(DISTINCT col)结果的近似聚合，底层实现将数据存储类型转换为HyperLogLog类型进行计算。NDV函数在计算时会消耗大量资源，因此不太适合高并发场景。

如果要进行用户行为分析，可以考虑使用IntersectCount或自定义UDAF。
