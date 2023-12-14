---
displayed_sidebar: "Chinese"
---

# 创建表

本快速入门教程将引导您完成在StarRocks中创建表所需的步骤，并介绍StarRocks的一些基本特性。

在部署StarRocks实例之后（详细信息请参阅[使用Docker部署StarRocks](../quick_start/deploy_with_docker.md)），您需要创建一个数据库和一张表来[加载和查询数据](../quick_start/Import_and_query.md)。创建数据库和表需要相应的[用户权限](../administration/User_privilege.md)。在本快速入门教程中，您可以使用默认的`root`用户来执行以下步骤，该用户在StarRocks实例上拥有最高权限。

> **注意**
>
> 您可以通过使用现有的StarRocks实例、数据库、表和用户权限来完成本教程。但是，为简单起见，我们建议您使用教程提供的架构和数据。

## 步骤1：登录到StarRocks

通过您的MySQL客户端登录到StarRocks。您可以使用默认用户`root`登录，默认情况下密码为空。

```Plain
mysql -h <fe_ip> -P<fe_query_port> -uroot
```

> **注意**
>
> - 如果您指定了不同的FE MySQL服务器端口（`query_port`，默认值：`9030`），请相应地更改`-P`的值。
> - 如果您在FE配置文件中指定了配置项`priority_networks`，请相应地更改`-h`的值。

## 步骤2：创建数据库

通过参考[CREATE DATABASE](../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)创建名为`sr_hub`的数据库。

```SQL
CREATE DATABASE IF NOT EXISTS sr_hub;
```

您可以通过执行[SHOW DATABASES](../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) SQL来查看此StarRocks实例中的所有数据库。

## 步骤3：创建表

运行`USE sr_hub`以切换到`sr_hub`数据库，并通过参考[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)创建名为`sr_member`的表。

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
>
> - 从v3.1开始，在创建表时您无需在DISTRIBUTED BY子句中指定桶键。StarRocks支持随机分桶，会将数据随机分布到所有桶中。有关更多信息，请参阅[随机分桶](../table_design/Data_distribution.md#random-bucketing-since-v31)。
> - 由于您部署的StarRocks实例仅有一个BE节点，在创建表时您需要将数据复制个数表示为`1`。
> - 如果没有指定[table type](../table_design/table_types/table_types.md)，则默认创建一个Duplicate Key表。请参阅[Duplicate Key表](../table_design/table_types/duplicate_key_table.md)。
> - 表的列正好对应于您将在教程中加载到StarRocks中的数据字段。为了保证**生产环境**中的高性能，我们强烈建议您使用`PARTITION BY`子句来制定表的数据分区方案。有关更多指示，请参阅[设计分区和分桶规则](../table_design/Data_distribution.md#design-partitioning-and-bucketing-rules)。


表创建完成后，您可以使用DESC语句检查表的详细信息，并通过执行[SHOW TABLES](../sql-reference/sql-statements/data-manipulation/SHOW_TABLES.md)查看数据库中的所有表。StarRocks支持模式更改。您可以查看[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)获取更多信息。

## 接下来做什么

要了解更多关于StarRocks表的概念细节，请参阅[StarRocks表设计](../table_design/StarRocks_table_design.md)。

除了本教程展示的特性外，StarRocks还支持：

- 各种[数据类型](../sql-reference/sql-statements/data-types/BIGINT.md)
- 多种[表类型](../table_design/table_types/table_types.md)
- 灵活的[分区策略](../table_design/Data_distribution.md#dynamic-partition-management)
- 经典的数据库查询索引，包括[位图索引](../using_starrocks/Bitmap_index.md)和[布隆过滤器索引](../using_starrocks/Bloomfilter_index.md)
- [物化视图](../using_starrocks/Materialized_view.md)