---
displayed_sidebar: "Chinese"
---

# 主键表

创建表时，可以分别定义主键和排序键。加载数据到主键表后，StarRocks会根据排序键对数据进行排序后再存储数据。查询会返回具有相同主键的一组记录中的最近记录。与唯一键表不同，主键表在查询期间不需要聚合操作，并支持谓词和索引的下推。因此，主键表尽管需要实时和频繁的数据更新，仍能提供高查询性能。

> **说明**
>
> - 在v3.0之前的版本中，主键表不支持解耦主键和排序键。
> - 自版本3.1以来，StarRocks的共享数据模式支持主键表。自版本3.1.4以来，在StarRocks共享数据集群中创建的主键表进一步支持将索引持久化到本地磁盘。

## 使用场景

- 主键表适用于以下需要实时频繁更新数据的场景：
  - **实时从事务处理系统中的流数据导入StarRocks**。在正常情况下，事务处理系统涉及大量更新和删除操作以及插入操作。如果需要将数据从事务处理系统同步到StarRocks，建议创建使用主键表的表。然后，可以使用诸如Apache Flink®的CDC Connectors等工具将事务处理系统的二进制日志同步到StarRocks。StarRocks使用二进制日志实时添加、删除和更新表中的数据。这简化了数据同步，并提供了比使用唯一键表的Merge on Read (MoR)表查询性能提高3至10倍。例如，您可以使用flink-connector-starrocks加载数据。有关更多信息，请参阅 [使用flink-connector-starrocks加载数据](../../loading/Flink-connector-starrocks.md)。

  - **通过对单个列执行更新操作合并多个流**。在业务场景中，例如用户画像，首选使用扁平表来提高多维分析性能并简化数据分析人员使用的分析模型。在这些场景中，上游数据可能来自各种应用程序，例如购物应用程序、交付应用程序和银行应用程序，或者来自进行计算以获取用户的不同标签和属性的系统，例如机器学习系统。主键表在这些场景中非常适用，因为它支持对单个列的更新。在实现实时数据增加、删除和更新的高查询性能的同时，每个应用程序或系统可以仅更新在其自身服务范围内保存数据的列。

- 主键表适用于主键所占内存可控的场景。

  StarRocks的存储引擎为使用主键表的每个表创建了主键的索引。此外，在将数据加载到表时，StarRocks会将主键索引加载到内存中。因此，主键表需要比其他三种表类型更大的内存容量。**StarRocks将由主键组成的字段的总长度编码后限制为127个字节。**

  如果一个表具有以下特点，考虑使用主键表：

  - 表中包含变化快速和变化缓慢的数据。变化快速的数据在最近几天内频繁更新，而变化缓慢的数据很少更新。假设需要将MySQL订单表实时同步到StarRocks以进行分析和查询。在这个例子中，表的数据按天分区，大多数更新操作是在最近几天内创建的订单上执行的。完成历史订单后，历史订单将不再更新。当运行数据加载作业时，主键索引不会加载到内存中，只有最近更新的订单的索引项会加载到内存中。

    如下图所示，表中的数据按天分区，并且最近两个分区中的数据经常更新。

    ![Primary index -1](../../assets/3.2-1.png)

  - 表是由数百或数千列组成的扁平表。主键只占表数据的一小部分，并且仅消耗少量内存。例如，用户状态或资料表包含大量列，但只有数百万至数亿用户。在这种情况下，主键消耗的内存量是可控的。

    如下图所示，表只包含少量行，表的主键只涵盖表的一小部分。

    ![Primary index -2](../../assets/3.2.4-2.png)

### 原理

主键表是基于StarRocks提供的新存储引擎设计的。主键表的元数据结构和读写机制与唯一键表不同。因此，主键表不需要聚合操作，并支持谓词和索引的下推。这些显著提高了查询性能。

唯一键表采用MoR策略。MoR简化了数据写入，但需要对多个数据版本进行在线聚合。此外，合并运算符不支持谓词和索引的下推。因此，查询性能下降。

主键表采用Delete+Insert策略，以确保每条记录都具有唯一的主键。因此，主键表不需要合并操作。详细信息如下：

- 当StarRocks接收到对记录的更新操作请求时，它会通过搜索主键索引找到记录，将记录标记为删除，并插入新记录。换句话说，StarRocks会将更新操作转换为删除操作加上插入操作。

- 当StarRocks接收到对记录的删除操作时，它会通过搜索主键索引找到记录并将记录标记为删除。

## 创建表

示例1：假设需要按日分析订单。在此示例中，创建一个名为`orders`的表，将`dt`和`order_id`定义为主键，并将其他列定义为指标列。

```SQL
create table orders (
    dt date NOT NULL,
    order_id bigint NOT NULL,
    user_id int NOT NULL,
    merchant_id int NOT NULL,
    good_id int NOT NULL,
    good_name string NOT NULL,
    price int NOT NULL,
    cnt int NOT NULL,
    revenue int NOT NULL,
    state tinyint NOT NULL
) PRIMARY KEY (dt, order_id)
PARTITION BY RANGE(`dt`) (
    PARTITION p20210820 VALUES [('2021-08-20'), ('2021-08-21')),
    PARTITION p20210821 VALUES [('2021-08-21'), ('2021-08-22')),
    ...
    PARTITION p20210929 VALUES [('2021-09-29'), ('2021-09-30')),
    PARTITION p20210930 VALUES [('2021-09-30'), ('2021-10-01'))
) DISTRIBUTED BY HASH(order_id)
PROPERTIES("replication_num" = "3",
"enable_persistent_index" = "true");
```

> **注意**
>
> - 创建表时，必须使用`DISTRIBUTED BY HASH`子句指定桶列。有关详细信息，请参见 [桶分配](../Data_distribution.md#design-partitioning-and-bucketing-rules)。
> - 自v2.5.7起，StarRocks在创建表或添加分区时可以自动设置桶的数量（BUCKETS）。您不再需要手动设置桶的数量。有关详细信息，请参见 [确定桶的数量](../Data_distribution.md#determine-the-number-of-buckets)。

示例2：假设需要根据用户的地址和最后活跃时间等维度实时分析用户行为。创建表时，可以将`user_id`列定义为主键，并将`address`和`last_active`列的组合定义为排序键。

```SQL
create table users (
    user_id bigint NOT NULL,
    name string NOT NULL,
    email string NULL,
    address string NULL,
    age tinyint NULL,
    sex tinyint NULL,
    last_active datetime,
    property0 tinyint NOT NULL,
    property1 tinyint NOT NULL,
    property2 tinyint NOT NULL,
    property3 tinyint NOT NULL,
    ....
) PRIMARY KEY (user_id)
DISTRIBUTED BY HASH(user_id)
ORDER BY(`address`,`last_active`)
PROPERTIES("replication_num" = "3",
"enable_persistent_index" = "true");
```

## 使用须知

- 关于表的主键，注意以下几点：
  - 主键使用`PRIMARY KEY`关键字来定义。

  - 主键必须在强制唯一约束的列上创建，并且主键列的名称不能更改。

  - 主键列可以是以下任何数据类型：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、STRING、VARCHAR、DATE和DATETIME。但是，主键列不能定义为`NULL`。

  - 分区列和桶列必须参与主键。

- 主键列的数量和总长度必须经过合理设计以节约内存。我们建议您识别数据类型占用较少内存的列，并将这些列定义为主键。此类数据类型包括 INT 和 BIGINT。我们建议不要让 VARCHAR 数据类型的列参与主键。

- 在创建表之前，我们建议您根据主键列的数据类型和表中行的数量来估算主键索引占用的内存。这样，您就可以避免表耗尽内存。以下示例说明了如何计算主键索引占用的内存：
  - 假设`dt`列的数据类型为占用 4 字节的 DATE 类型，`id`列的数据类型为占用 8 字节的 BIGINT 类型，则主键的长度为 12 字节。

  - 假设表包含 10,000,000 行热数据，并存储在三个副本中。

  - 根据上述信息，根据以下公式，主键索引占用的内存为 945 MB：

    (12 + 9) x 10,000,000 x 3 x 1.5 = 945 (MB)

    在上述公式中，`9`是每行的不变开销，`1.5` 是每个哈希表的平均额外开销。

- `enable_persistent_index`: 可以将主键索引持久化到磁盘并存储在内存中，以避免占用过多内存。一般情况下，主键索引占用的内存只能是原来的 1/10。在创建表时，您可以在 `PROPERTIES` 中设置此属性。有效值为 true 或 false。默认值为 false。

  > - 如果您想在创建表后修改此参数，请参见 [ALTER TABLE](../../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) 中表属性修改部分。
  > - 建议在磁盘为 SSD 时将此属性设置为 true。
  > - 从版本 2.3.0 开始，StarRocks 支持设置此属性。
  > - 自版本 3.1 开始，StarRocks 的共享数据模式支持主键表。自版本 3.1.4 开始，StarRocks 共享数据集群中创建的主键表进一步支持将索引持久化到本地磁盘。

- 您可以使用 `ORDER BY` 关键字将排序键指定为任何列的排列和组合。

  > **注意**
  >
  > 如果指定了排序键，则会根据排序键构建前缀索引；如果未指定排序键，则会根据主键构建前缀索引。

- ALTER TABLE 可用于更改表模式，但存在以下限制：
  - 不支持修改主键。
  - 支持使用 ALTER TABLE ... ORDER BY ... 重新分配排序键。不支持删除排序键。不支持修改排序键中列的数据类型。
  - 不支持调整列顺序。

- 从 2.3.0 版开始，除了主键列，现在还支持 BITMAP 和 HLL 数据类型的列。

- 创建表时，您可以在除主键列以外的列上创建 BITMAP 索引或 Bloom Filter 索引。

- 从 2.4.0 版开始，您可以基于主键表创建异步物化视图。

## 下一步操作

在创建表后，您可以运行装载任务将数据加载到主键表中。有关支持的加载方法的更多信息，请参见 [数据加载概述](../../loading/Loading_intro.md)。

如果需要更新主键表中的数据，您可以 [运行装载任务](../../loading/Load_to_Primary_Key_tables.md) 或执行 DML 语句（[UPDATE](../../sql-reference/sql-statements/data-manipulation/UPDATE.md) 或 [DELETE](../../sql-reference/sql-statements/data-manipulation/DELETE.md)）。这些更新操作也保证了原子性。