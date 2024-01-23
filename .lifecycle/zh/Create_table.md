---
displayed_sidebar: English
---

# 创建一个表格

本快速入门教程将引导您完成在 StarRocks 中创建表的必要步骤，并介绍 StarRocks 的一些基本特性。

在部署 StarRocks 实例之后（详见[部署 StarRocks](../quick_start/deploy_with_docker.md)了解详情），您需要创建数据库和表来[加载和查询数据](../quick_start/Import_and_query.md)。创建数据库和表需要相应的[用户权限](../administration/User_privilege.md)。在这个快速入门教程中，您可以使用默认的`root`用户来执行以下步骤，该用户在 StarRocks 实例上拥有最高权限。

> **注意**
> 您可以使用现有的 StarRocks 实例、数据库、表和用户权限来完成本教程。然而，为了简化操作，我们推荐您使用教程提供的模式和数据。

## 第一步：登录 StarRocks

通过您的 MySQL 客户端登录 StarRocks。您可以使用默认用户 `root` 登录，且默认密码为空。

```Plain
mysql -h <fe_ip> -P<fe_query_port> -uroot
```

> **注意**
-  如果您分配了不同的 FE MySQL 服务器端口，请相应地更改 `-P` 值（`query_port`，默认值：`9030`）。
- 如果您在 FE 配置文件中指定了配置项`priority_networks`，请相应地更改`-h`值。

## 步骤 2：创建一个数据库

通过参考[创建数据库](../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)来创建一个名为`sr_hub`的数据库。

```SQL
CREATE DATABASE IF NOT EXISTS sr_hub;
```

您可以通过执行 [SHOW DATABASES](../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) SQL 来查看该 StarRocks 实例下的所有数据库。

## 步骤 3：创建一个表格

运行`USE sr_hub`来切换到`sr_hub`数据库，并参考[创建表](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)来创建名为`sr_member`的表。

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
- 从 v3.1 开始，创建表时无需在 DISTRIBUTED BY 子句中指定分桶键。StarRocks 支持随机分桶，可以将数据随机分布到所有桶中。更多信息，请参见[随机分桶](../table_design/Data_distribution.md#random-bucketing-since-v31)。
- 您需要将表属性 `replication_num`（代表数据副本数量）指定为 `1`，因为您部署的 StarRocks 实例只有一个 BE 节点。
- 如果没有指定[表类型](../table_design/table_types/table_types.md)，系统默认会创建一个[重复键表](../table_design/table_types/duplicate_key_table.md)。参见[重复键表](../table_design/table_types/duplicate_key_table.md)
- 该表的列与您在[加载和查询数据教程](../quick_start/Import_and_query.md)中将要加载到 StarRocks 的数据字段完全对应。
- 为了保证**生产环境中**的高性能，我们强烈建议您使用`PARTITION BY`子句来制定表的数据分区计划。有关更多说明，请参阅[设计分区和桶规则](../table_design/Data_distribution.md#design-partitioning-and-bucketing-rules)。

表创建完成后，可以使用 DESC 语句查看表的详细信息，并通过执行 [SHOW TABLES](../sql-reference/sql-statements/data-manipulation/SHOW_TABLES.md) 查看数据库中的所有表。StarRocks 中的表支持更改 Schema。有关更多信息，请参阅 [ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)。

## 接下来做什么

要了解更多关于 StarRocks 表的概念性细节，请参阅 [StarRocks 表设计](../table_design/StarRocks_table_design.md)。

除了本教程演示的功能外，StarRocks 还支持：

- 多种 [数据类型](../sql-reference/sql-statements/data-types/BIGINT.md)
- 多种[表格类型](../table_design/table_types/table_types.md)
- 灵活的 [分区策略](../table_design/Data_distribution.md#dynamic-partition-management)
- 经典数据库查询索引，包括[位图索引](../using_starrocks/Bitmap_index.md)和[布隆过滤器索引](../using_starrocks/Bloomfilter_index.md)
- [物化视图](../using_starrocks/Materialized_view.md)
