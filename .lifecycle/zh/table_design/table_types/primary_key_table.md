---
displayed_sidebar: English
---

# 主键表

创建表时，您可以分别定义主键和排序键。当数据被加载进主键表时，StarRocks会根据排序键对数据进行排序后再存储。查询将返回具有相同主键的记录组中最新的记录。与唯一键表不同，主键表在查询时不需要进行聚合操作，并且支持谓词下推和索引下推。因此，即使在数据实时且频繁更新的情况下，主键表也能提供高性能的查询。

> **注意**
- 在v3.0之前的版本中，主键表不支持主键和排序键的分离。
- 从3.1版本开始，StarRocks的共享数据模式开始支持主键表。从3.1.4版本开始，在StarRocks共享数据集群中创建的主键表进一步支持将索引持久化到本地磁盘。

## 适用场景

- 主键表适合于以下需要频繁实时更新数据的场景：
-   **实时将交易处理系统中的数据流入StarRocks。**通常，交易处理系统除了插入操作外，还包含大量的更新和删除操作。如果您需要将交易处理系统中的数据同步到StarRocks，我们建议您创建一个使用主键表的表。然后，您可以使用如Apache Flink®的CDC Connectors等工具，将交易处理系统的二进制日志同步到StarRocks。StarRocks利用这些二进制日志实时地添加、删除和更新表中的数据。这样不仅简化了数据同步过程，而且比使用唯一键表的合并读取(MoR)表的查询性能提高了3到10倍。例如，您可以使用flink-connector-starrocks来加载数据。更多信息，请参见[使用flink-connector-starrocks加载数据](../../loading/Flink-connector-starrocks.md)。

-   **通过对单个列执行更新操作来合并多个数据流**。在用户画像等商业场景中，通常使用扁平表来提升多维分析性能并简化数据分析模型。这些场景中的上游数据可能来自各种应用，如购物、配送和银行应用程序，或来自执行计算以获取用户独特标签和属性的机器学习系统。**主键表在这些场景中表现良好**，因为它支持对单个列进行更新。每个应用或系统只需更新其服务范围内的数据列，同时享受实时数据的添加、删除和更新带来的高查询性能。

- 主键表适用于主键所占内存可控的场景。

  StarRocks的存储引擎会为每个使用主键表的表创建一个主键索引。此外，在您向表中加载数据时，StarRocks会将主键索引加载到内存中。因此，主键表比其他三种表类型需要更大的内存容量。**StarRocks将编码后的主键字段总长度限制为127字节**。

  如果表具有以下特点，请考虑使用主键表：

-   表包含快速变化和缓慢变化的数据。快速变化的数据在最近的几天内频繁更新，而缓慢变化的数据很少更新。假设您需要实时同步MySQL订单表到StarRocks进行分析和查询。在这个例子中，表的数据按天分区，而且大多数更新都针对最近几天内创建的订单。一旦订单完成，历史订单就不再更新。当您执行数据加载作业时，只有最近更新的订单的主键索引会被加载到内存中。

    如下图所示，表数据按天分区，最近两个分区中的数据频繁更新。

    ![主索引-1](../../assets/3.2-1.png)

-   表是一个由数百或数千列组成的扁平表。主键只占表数据的一小部分，并且仅占用少量内存。例如，用户状态或配置文件表可能包含大量列，但只有数千万到数亿用户。在这种情况下，主键占用的内存量是可控的。

    如下图所示，表只包含少数行，且表的主键只占一小部分。

    ![主索引-2](../../assets/3.2.4-2.png)

### 原理

主键表基于StarRocks提供的一种新的存储引擎设计。主键表的元数据结构和读写机制与重复键表不同。因此，主键表不需要聚合操作，并支持谓词和索引的下推，这显著提高了查询性能。

重复键表采用合并读取(MoR)策略。MoR简化了数据写入，但需要在线合并多个数据版本。此外，合并运算符不支持谓词和索引的下推，导致查询性能下降。

主键表采用删除+插入策略，确保每条记录都具有唯一的主键，从而不需要合并操作。其详细情况如下：

- 当StarRocks接收到对记录的更新操作请求时，它通过搜索主键索引定位该记录，将记录标记为已删除，然后插入新记录。换言之，StarRocks将更新操作转换为删除操作加上插入操作。

- 当StarRocks接收到记录的删除操作时，它通过搜索主键索引来定位记录，并将其标记为已删除。

## 创建表

示例1：假设您需要每日分析订单数据。在这个例子中，创建一个名为orders的表，将dt和order_id定义为主键，其他列定义为度量列。

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
- 创建表时，必须使用 `DISTRIBUTED BY HASH` 子句指定分桶列。更多详细信息，请参阅[bucketing](../Data_distribution.md#design-partitioning-and-bucketing-rules)。
- 从v2.5.7版本开始，StarRocks在创建表或添加分区时可以自动设置存储桶（BUCKETS）的数量，无需手动设置。更多详细信息，请参阅[determine the number of buckets](../Data_distribution.md#determine-the-number-of-buckets)。

示例2：假设您需要实时从用户地址和最后活动时间等维度分析用户行为。创建表时，您可以将user_id列定义为主键，并将address和last_active列的组合定义为排序键。

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

## 使用说明

- 在设计表的主键时，请注意以下几点：
-   主键使用PRIMARY KEY关键字定义。

-   主键必须建立在有唯一性约束的列上，且主键列的名称不可更改。

-   主键列可以是以下数据类型：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、STRING、VARCHAR、DATE和DATETIME。但是，主键列不能定义为NULL。

-   分区列和桶列必须包含在主键中。

-   为了节省内存，需要合理设计主键列的数量和总长度。我们建议您选择数据类型占用内存较小的列作为主键，例如INT和BIGINT。我们建议不要让VARCHAR数据类型的列参与主键。

-   在创建表之前，建议您根据主键列的数据类型和表中的行数估算主键索引占用的内存，以避免表内存不足。以下示例演示了如何计算主键索引占用的内存：
-     假设dt列是DATE数据类型，占用4字节，id列是BIGINT数据类型，占用8字节，它们被定义为主键。在这种情况下，主键长度为12字节。

-     假设表中有10,000,000行热数据，并存储在三个副本中。

-     根据上述信息，根据以下公式计算出的主键索引占用的内存为945MB：

      (12 + 9) x 10,000,000 x 3 x 1.5 = 945 (MB)

      在上述公式中，9是每行固定的开销，1.5是哈希表平均每项额外的开销。

- enable_persistent_index：主键索引可以持久化到磁盘并存储在内存中，以减少其占用的内存量。通常情况下，主键索引只占用原来内存的1/10。在创建表时，您可以在PROPERTIES中设置此属性。有效值为true或false，默认值为false。

  - 如果您想在表创建后修改此参数，请参见[ALTER TABLE](../../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)中的修改表属性部分。
  - 如果磁盘是SSD，建议将此属性设置为true。
  - 从2.3.0版本开始，StarRocks支持设置此属性。
  - 从3.1版本开始，StarRocks的共享数据模式支持主键表。从3.1.4版本开始，StarRocks共享数据集群中创建的主键表进一步支持索引持久化到本地磁盘。

- 您可以使用ORDER BY关键字指定任意列的排列组合作为排序键。

    > **注意**
    > 如果指定了排序键，则根据排序键构建前缀索引；如果未指定排序键，则根据主键构建前缀索引。

- ALTER TABLE可用于更改表结构，但存在以下限制：
  - 不支持修改主键。
  - 支持使用ALTER TABLE ... ORDER BY...重新指定排序键。不支持删除排序键。不支持修改排序键中列的数据类型。
  - 不支持调整列的顺序。

- 从2.3.0版本开始，除主键列之外的列现在支持BITMAP和HLL数据类型。

- 在创建表时，您可以为非主键列创建BITMAP索引或布隆过滤器(Bloom Filter)索引。

- 从2.4.0版本开始，您可以基于主键表创建异步物化视图。

## 下一步

创建表后，您可以执行**加载作业**，将数据加载到**主键表**中。有关支持的加载方法，请参阅[数据加载概述](../../loading/Loading_intro.md)。

如果您需要更新主键表中的数据，可以 [执行加载作业](../../loading/Load_to_Primary_Key_tables.md) 或执行 DML 语句（[UPDATE](../../sql-reference/sql-statements/data-manipulation/UPDATE.md) 或 [DELETE](../../sql-reference/sql-statements/data-manipulation/DELETE.md)）。也就是说，这些更新操作保证了原子性。
