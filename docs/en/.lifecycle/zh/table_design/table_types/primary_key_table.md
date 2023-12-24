---
displayed_sidebar: English
---

# 主键表

在创建表时，可以单独定义主键和排序键。当数据加载到主键表中时，StarRocks 会根据排序键对数据进行排序，然后再存储数据。查询返回具有相同主键的一组记录中的最新记录。与唯一键表不同，主键表在查询过程中不需要聚合操作，支持谓词和索引的下推。因此，尽管数据更新是实时且频繁的，但主键表仍可提供高查询性能。

> **注意**
>
> - 在 v3.0 之前的版本中，主键表不支持主键和排序键的分离。
> - 从 3.1 版本开始，StarRocks 的共享数据模式支持主键表。从 3.1.4 版本开始，StarRocks 共享数据集群中创建的主键表进一步支持索引持久化到本地磁盘上。

## 场景

- 主键表适用于以下需要频繁实时更新数据的场景：
  - **将数据从事务处理系统实时流式传输到 StarRocks。** 在正常情况下，事务处理系统除了涉及插入操作外，还涉及大量的更新和删除操作。如果您需要将事务处理系统中的数据同步到 StarRocks，建议您创建一个使用主键表的表。然后，您可以使用 CDC Connectors for Apache Flink® 等工具将事务处理系统的二进制日志同步到 StarRocks。StarRocks 使用二进制日志对表中的数据进行实时增删改换。这简化了数据同步，并提供比使用唯一键表的读取时合并 （MoR） 表高 3 到 10 倍的查询性能。例如，您可以使用 flink-connector-starrocks 来加载数据。更多信息，请参见[使用 flink-connector-starrocks 加载数据](../../loading/Flink-connector-starrocks.md)。

  - **通过对单个列执行更新操作来联接多个流**。在用户画像等业务场景中，最好使用平面表来提升多维度分析性能，简化数据分析师使用的分析模型。这些场景中的上游数据可能来自各种应用，例如购物应用、外卖应用和银行应用，也可能来自系统，例如执行计算以获取用户不同标签和属性的机器学习系统。主键表非常适合这些方案，因为它支持对单个列的更新。每个应用或系统只能更新在其自己的服务范围内保存数据的列，同时在高查询性能下受益于实时数据添加、删除和更新。

- 主键表适用于主键占用内存可控的场景。

  StarRocks 的存储引擎会为使用主键表的每个表的主键创建索引。此外，当您将数据加载到表中时，StarRocks 会将主键索引加载到内存中。因此，主键表需要比其他三种表类型更大的内存容量。 **StarRocks 将主键的字段总长度限制为 127 字节。**

  如果表具有以下特征，请考虑使用主键表：

  - 该表包含快速变化的数据和缓慢变化的数据。快速变化的数据在最近几天经常更新，而缓慢变化的数据很少更新。假设您需要将 MySQL 订单表实时同步到 StarRocks 进行分析和查询。在此示例中，表的数据按天分区，并且大多数更新都是对最近几天创建的订单执行的。历史订单在完成后不再更新。运行数据加载作业时，主键索引不会加载到内存中，只有最近更新的订单的索引条目才会加载到内存中。

    如下图所示，表中的数据是按天分区的，最近两个分区的数据是频繁更新的。

    ![主要索引 -1](../../assets/3.2-1.png)

  - 该表是由数百或数千列组成的平面表。主键仅包含表数据的一小部分，并且仅占用少量内存。例如，用户状态或配置文件表由大量列组成，但只有数千万到数亿个用户。在这种情况下，主键消耗的内存量是可控的。

    如下图所示，该表仅包含几行，该表的主键仅包含该表的一小部分。

    ![主要索引 -2](../../assets/3.2.4-2.png)

### 原则

主键表是基于 StarRocks 提供的全新存储引擎设计的。主键表中的元数据结构和读写机制与重复键表中的元数据结构和读写机制不同。因此，主键表不需要聚合操作，并支持谓词和索引的下推。这些显著提高了查询性能。

重复键表采用 MoR 策略。MoR 简化了数据写入，但需要在线聚合多个数据版本。此外，Merge 运算符不支持谓词和索引的下推。因此，查询性能会下降。

主键表采用删除+插入策略，确保每条记录都有唯一的主键。这样，主键表就不需要合并操作。详情如下：

- 当 StarRocks 收到对某条记录进行更新操作的请求时，它会通过搜索主键索引来定位该记录，将该记录标记为已删除，并插入一条新记录。换言之，StarRocks 将更新操作转换为删除操作和插入操作。

- 当 StarRocks 收到对记录的删除操作时，会通过搜索主键索引来定位该记录，并将该记录标记为已删除。

## 创建表

示例 1：假设您需要每天分析订单。在此示例中，创建一个名为 `orders` 的表，将 `dt` 和 `order_id` 定义为主键，并将其他列定义为度量列。

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
> - 创建表时，必须使用 `DISTRIBUTED BY HASH` 子句来指定分桶列。有关详细信息，请参见 [分桶](../Data_distribution.md#design-partitioning-and-bucketing-rules)。
> - 从 v2.5.7 版本开始，StarRocks 可以在创建表或添加分区时自动设置存储桶（BUCKETS）的数量。您不再需要手动设置存储桶的数量。有关详细信息，请参见 [确定存储桶的数量](../Data_distribution.md#determine-the-number-of-buckets)。

例如2：假设您需要实时分析用户行为，从用户地址和最后活跃时间等维度。在创建表时，您可以将 `user_id` 列定义为主键，并将 `address` 和 `last_active` 列的组合定义为排序键。

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

- 请注意以下关于表主键的要点：
  - 主键是使用 `PRIMARY KEY` 关键字定义的。

  - 主键必须在强制实施唯一约束的列上创建，且主键列的名称不能更改。

  - 主键列可以是以下数据类型之一：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、STRING、VARCHAR、DATE 和 DATETIME。但是，主键列不能定义为 `NULL`。

  - 分区列和存储桶列必须参与主键。

  - 主键列的数量和总长度必须合理设计，以节省内存。我们建议您确定数据类型占用较少内存的列，并将这些列定义为主键。此类数据类型包括 INT 和 BIGINT。我们建议不要让 VARCHAR 数据类型的列参与主键。

  - 在创建表之前，我们建议您根据主键列的数据类型和表中的行数来估算主键索引占用的内存。这样，您可以避免表内存不足。以下示例说明如何计算主键索引占用的内存：
    - 假设 `dt` 列是 DATE 数据类型，占用 4 个字节，`id` 列是 BIGINT 数据类型，占用 8 个字节，定义为主键。在这种情况下，主键长度为 12 个字节。

    - 假设表中包含 10,000,000 行热数据，并存储在三个副本中。

    - 根据上述信息，根据以下公式，主键索引占用的内存为 945 MB：

      (12 + 9) x 10,000,000 x 3 x 1.5 = 945 (MB)

      在上述公式中，`9` 是每行的不可变开销，`1.5` 是每个哈希表的平均额外开销。

- `enable_persistent_index`：主键索引可以持久化到磁盘并存储在内存中，以避免占用过多内存。通常情况下，主键索引只会占用之前的 1/10 内存。您可以在创建表时的 `PROPERTIES` 中设置此属性。有效值为 true 或 false。默认值为 false。

  > - 如果要在创建表后修改此参数，请参见 [ALTER TABLE](../../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) 中修改表属性部分。
  > - 如果磁盘是 SSD，建议将此属性设置为 true。
  > - 从 2.3.0 版本开始，StarRocks 支持设置此属性。
  > - 从 3.1 版本开始，StarRocks 的共享数据模式支持主键表。从 3.1.4 版本开始，在 StarRocks 共享数据集群中创建的主键表进一步支持将索引持久化到本地磁盘上。

- 您可以使用 `ORDER BY` 关键字将排序键指定为任何列的排列和组合。

  > **注意**
  >
  > 如果指定了排序键，则根据排序键构建前缀索引；如果未指定排序键，则根据主键构建前缀索引。

- ALTER TABLE 可用于更改表架构，但存在以下限制：
  - 不支持修改主键。
  - 支持使用 ALTER TABLE ... ORDER BY ... 重新分配排序键。不支持删除排序键。不支持修改排序键中列的数据类型。
  - 不支持调整列顺序。

- 从 2.3.0 版本开始，除主键列之外的列现在支持 BITMAP 和 HLL 数据类型。

- 创建表时，除主键列之外的列可以创建 BITMAP 索引或 Bloom Filter 索引。

- 从 2.4.0 版本开始，您可以基于主键表创建异步物化视图。

## 下一步操作

创建表后，您可以运行加载作业将数据加载到主键表中。有关支持的加载方法的详细信息，请参见 [数据加载概述](../../loading/Loading_intro.md)。
如果需要更新主键表中的数据，可以[运行加载作业](../../loading/Load_to_Primary_Key_tables.md)或执行 DML 语句（[UPDATE](../../sql-reference/sql-statements/data-manipulation/UPDATE.md)或[DELETE](../../sql-reference/sql-statements/data-manipulation/DELETE.md)）。另外，这些更新操作保证了原子性。