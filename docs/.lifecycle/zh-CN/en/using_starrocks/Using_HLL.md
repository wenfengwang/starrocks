---
displayed_sidebar: "Chinese"
---

# 使用HLL进行近似去重计数

## 背景

在现实世界的场景中，随着数据量的增加，去重数据的压力也在增加。当数据的大小达到一定水平时，准确去重的成本相对较高。在这种情况下，用户通常使用近似算法来减少计算压力。 HyperLogLog（HLL）是本节将介绍的一种近似去重算法，它具有出色的空间复杂度O（mloglogn）和时间复杂度O（n）。此外，根据数据集的大小和使用的哈希函数，计算结果的误差率可以控制在约1%-10%。

## 什么是HyperLogLog

HyperLogLog是一种占用非常少存储空间的近似去重算法。**HLL类型**用于实现HyperLogLog算法。它保存HyperLogLog计算的中间结果，并且只能作为数据表的指标列类型使用。

由于HLL算法涉及许多数学知识，我们将用一个实际示例来说明。假设我们设计一个随机化实验A，即进行独立的硬币翻转重复试验，直到出现第一个正面朝上的情况；将第一个正面朝上时的翻转次数记录为随机变量X，那么：

* X=1，P（X=1）=1/2
* X=2，P（X=2）=1/4
* ...
* X=n，P（X=n）=（1/2）<sup>n</sup>

我们使用实验A来构造随机化试验B，即对试验A进行N次独立重复，生成N个相互独立的随机变量X<sub>1</sub>、X<sub>2</sub>、X<sub>3</sub>、…、X<sub>N</sub>。取这些随机变量的最大值作为X<sub>max</sub>。利用极大似然估计，N的估计值是2<sup>X<sub>max</sub></sup>。
<br/>

现在，我们使用给定数据集上的哈希函数模拟上述实验：

* 试验A：计算数据集元素的哈希值，并将哈希值转换为二进制表示。记录从二进制的最低位开始出现的位=1的次数。
* 试验B：为试验B的数据集元素重复进行试验A的过程。更新每个试验中第一个出现的位1的最大位置“m”；
* 估计数据集中不重复元素的数量为m<sup>2</sup>。

事实上，HLL算法将元素根据元素哈希的较低k位分成K=2<sup>k</sup>个桶。计算从第k+1位开始的第一个位1的最大位置m<sub>1</sub>、m<sub>2</sub>、…、m<sub>k</sub>，并估计每个桶中不重复元素的数量为2<sup>m<sub>1</sub></sup>、2<sup>m<sub>2</sub></sup>、…、2<sup>m<sub>k</sub></sup>。数据集中不重复元素的数量是桶的数量乘以桶中不重复元素的数量的总平均值：N = K(K/(2<sup>\-m<sub>1</sub></sup>+2<sup>\-m<sub>2</sub></sup>,..., 2<sup>\-m<sub>K</sub></sup>)。
<br/>

HLL将修正因子与估计结果相乘，使结果更加准确。

有关使用StarRocks SQL语句实现HLL去重算法的文章，请参考[https://gist.github.com/avibryant/8275649](https://gist.github.com/avibryant/8275649)。

~~~sql
SELECT floor((0.721 * 1024 * 1024) / (sum(pow(2, m * -1)) + 1024 - count(*))) AS estimate
FROM(select(murmur_hash3_32(c2) & 1023) AS bucket,
     max((31 - CAST(log2(murmur_hash3_32(c2) & 2147483647) AS INT))) AS m
     FROM db0.table0
     GROUP BY bucket) bucket_values
~~~

该算法对db0.table0的col2进行去重。

* 使用哈希函数`murmur_hash3_32`计算col2的哈希值为32位有符号整数。
* 使用1024个桶，修正因子为0.721，并将哈希值的低10位作为桶的下标。
* 忽略哈希值的符号位，从次高位到低位确定第一个位1的位置。
* 将计算得到的哈希值按桶分组，并使用`MAX`聚合找到桶中第一个位1的最大位置。
* 聚合结果用作子查询，并将所有桶估计的总平均值乘以桶的数量和修正因子。
* 注意空桶的数量为1。

当数据量很大时，上述算法具有非常低的误差率。

这就是HLL算法的核心思想。如果您感兴趣，请参阅[HyperLogLog论文](http://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf)。

### 如何使用HyperLogLog

1. 若要使用HyperLogLog去重，您需要在表创建语句中将目标指标列类型设置为`HLL`，并将聚合函数设置为`HLL_UNION`。
2. 目前，只有聚合模型支持将HLL作为指标列类型。
3. 在HLL类型的列上使用`count distinct`时，StarRocks将自动转换为`HLL_UNION_AGG`计算。

#### 示例

首先，创建一个包含**HLL**列的表，其中uv是一个聚合列，列类型为`HLL`，聚合函数为[HLL_UNION](../sql-reference/sql-functions/aggregate-functions/hll_union.md)。

~~~sql
CREATE TABLE test(
        dt DATE,
        id INT,
        uv HLL HLL_UNION
)
DISTRIBUTED BY HASH(ID);
~~~

> * 注意：当数据量很大时，最好为高频HLL查询创建相应的滚动表

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

经纪人加载模式：

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

* HLL列不允许直接查询其原始值，使用函数[HLL_UNION_AGG](../sql-reference/sql-functions/aggregate-functions/hll_union_agg.md)进行查询。
* 要找出总uv，

`SELECT HLL_UNION_AGG(uv) FROM test;`

该语句等同于

`SELECT COUNT(DISTINCT uv) FROM test;`

* 查询每天的uv

`SELECT COUNT(DISTINCT uv) FROM test GROUP BY ID;`

### 注意事项

在Bitmap和HLL之间应该如何选择？如果数据集的基数在百万级或千万级，并且您有几十台机器，请使用`count distinct`。如果基数在亿级，并且需要准确去重，则使用`Bitmap`；如果可以接受近似去重，则使用`HLL`类型。

Bitmap only supports TINYINT, SMALLINT, INT, and BIGINT. Note that LARGEINT is not supported. For other types of data sets to be de-duplicated, a dictionary needs to be built to map the original type to an integer type.  Building a dictionary  is complex, and requires a trade-off between data volume, update frequency, query efficiency, storage, and other issues. HLL does not need a dictionary, but it needs the corresponding data type to support the hash function. Even in an analytical system that does not support HLL internally, it is still possible to use the hash function and SQL to implement HLL de-duplication.

For common columns, users can use the NDV function for approximate de-duplication. This function returns an approximate aggregation of COUNT(DISTINCT col) results, and the underlying implementation converts the data storage type to the HyperLogLog type for calculation. The NDV function consumes a lot of resources  when calculating and is therefore  not well suited for high concurrency scenarios.

If you want to perform user behavior analysis, you may consider IntersectCount or custom UDAF.