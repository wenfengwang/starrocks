---
displayed_sidebar: English
---

# 创建一个表

本快速入门教程将引导您完成在 StarRocks 中创建表的必要步骤，并介绍 StarRocks 的一些基本特性。

在 StarRocks 实例部署完成后（详情请参见[部署 StarRocks](../quick_start/deploy_with_docker.md)），您需要创建数据库和表来[加载和查询数据](../quick_start/Import_and_query.md)。创建数据库和表需要相应的[用户权限](../administration/User_privilege.md)。在本快速入门教程中，您可以使用默认的 `root` 用户执行以下步骤，该用户在 StarRocks 实例上拥有最高权限。

> **注意**
> 您可以使用现有的 StarRocks 实例、数据库、表和用户权限来完成本教程。但是，为了简化操作，我们建议您使用教程提供的架构和数据。

## 第 1 步：登录 StarRocks

通过 MySQL 客户端登录 StarRocks。您可以使用默认用户 `root` 登录，密码默认为空。

```Plain
mysql -h <fe_ip> -P<fe_query_port> -uroot
```

> **注意**
- 如果您分配了不同的 FE MySQL 服务器端口（`query_port`，默认值：`9030`），请相应地更改 `-P` 值。
- 如果在 FE 配置文件中指定了 `priority_networks` 配置项，请相应修改 `-h` 值。

## 第 2 步：创建数据库

通过参考 [CREATE DATABASE](../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md) 创建名为 `sr_hub` 的数据库。

```SQL
CREATE DATABASE IF NOT EXISTS sr_hub;
```

您可以通过执行 [SHOW DATABASES](../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) SQL 查看该 StarRocks 实例中的所有数据库。

## 第 3 步：创建表

执行 `USE sr_hub` 切换到 `sr_hub` 数据库，并参考 [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) 创建名为 `sr_member` 的表。

```SQL
USE sr_hub;
CREATE TABLE IF NOT EXISTS sr_member (
    sr_id            INT,
    name             STRING,
    city_code        INT,
    reg_date         DATE,
    verified         BOOLEAN
)
PARTITION BY RANGE(reg_date)
(
    PARTITION p1 VALUES [('2022-03-13'), ('2022-03-14')),
    PARTITION p2 VALUES [('2022-03-14'), ('2022-03-15')),
    PARTITION p3 VALUES [('2022-03-15'), ('2022-03-16')),
    PARTITION p4 VALUES [('2022-03-16'), ('2022-03-17')),
    PARTITION p5 VALUES [('2022-03-17'), ('2022-03-18'))
)
DISTRIBUTED BY HASH(city_code);
```

> **注意**
- 从 v3.1 开始，创建表时不需要在 `DISTRIBUTED BY` 子句中指定分桶键。StarRocks 支持随机分桶，即在所有存储桶中随机分布数据。有关更多信息，请参阅[随机分桶](../table_design/Data_distribution.md#random-bucketing-since-v31)。
- 由于您部署的 StarRocks 实例只有一个 BE 节点，因此需要将表属性 `replication_num`（代表数据副本数）指定为 `1`。
- 如果未指定 [table type](../table_design/table_types/table_types.md)，则默认创建 Duplicate Key 表。请参阅 [Duplicate Key table](../table_design/table_types/duplicate_key_table.md)。
- 表中的列与您将在[加载和查询数据](../quick_start/Import_and_query.md)教程中加载到 StarRocks 的数据字段完全对应。
- 为了保证**生产环境中**的高性能，我们强烈建议您使用 `PARTITION BY` 子句为表制定数据分区方案。请参阅[设计分区和分桶规则](../table_design/Data_distribution.md#design-partitioning-and-bucketing-rules)以获取更多指导。

表创建完成后，可以使用 DESC 语句查看表的详细信息，并通过执行 [SHOW TABLES](../sql-reference/sql-statements/data-manipulation/SHOW_TABLES.md) 查看数据库中的所有表。StarRocks 中的表支持架构更改。您可以查看 [ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) 了解更多信息。

## 接下来做什么

要了解有关 StarRocks 表的概念细节，请参阅 [StarRocks Table Design](../table_design/StarRocks_table_design.md)。

除了本教程演示的功能之外，StarRocks 还支持：

- 多种[数据类型](../sql-reference/sql-statements/data-types/BIGINT.md)
- 多种 [table types](../table_design/table_types/table_types.md)
- 灵活的[分区策略](../table_design/Data_distribution.md#dynamic-partition-management)
- 经典数据库查询索引，包括 [bitmap index](../using_starrocks/Bitmap_index.md) 和 [bloom filter index](../using_starrocks/Bloomfilter_index.md)
- [物化视图](../using_starrocks/Materialized_view.md)