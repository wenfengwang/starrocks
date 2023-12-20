---
displayed_sidebar: English
---

# 创建表格

本快速入门教程将指导您完成在 StarRocks 中创建表的必要步骤，并向您介绍 StarRocks 的一些基础特性。

在 StarRocks 实例部署完毕后（详情请参阅[部署 StarRocks](../quick_start/deploy_with_docker.md)），您需要创建数据库和表来进行[数据的加载和查询](../quick_start/Import_and_query.md)。创建数据库和表需要相应的[用户权限](../administration/User_privilege.md)。在本快速入门教程中，您可以使用具有最高权限的默认 `root` 用户来执行以下步骤。

> **注意**
> 您可以通过使用现有的StarRocks实例、数据库、表和用户权限来完成本教程。但为简化操作，我们推荐您使用教程提供的架构和数据。

## 第1步：登录 StarRocks

通过 MySQL 客户端登录 StarRocks。您可以使用默认用户 root 登录，密码默认为空。

```Plain
mysql -h <fe_ip> -P<fe_query_port> -uroot
```

> **注意**
- 如果您为 FE MySQL 服务器端口（查询端口，默认：9030）设置了不同的值，请相应地更改 -P 参数。
- 如果您在 FE 配置文件中指定了 priority_networks 配置项，请相应地更改 -h 参数。

## 第2步：创建数据库

参照[CREATE DATABASE](../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)创建一个名为 `sr_hub` 的数据库。

```SQL
CREATE DATABASE IF NOT EXISTS sr_hub;
```

您可以通过执行[SHOW DATABASES](../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) SQL命令查看该 StarRocks 实例中的所有数据库。

## 第3步：创建表

执行 `USE sr_hub` 切换到 `sr_hub` 数据库，并参照 [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) 创建一个名为 `sr_member` 的表。

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
- 从 v3.1 版本开始，创建表时无需在 DISTRIBUTED BY 子句中指定桶键。StarRocks支持随机分桶，它会随机地将数据分布到所有桶中。更多信息请参见[随机分桶](../table_design/Data_distribution.md#random-bucketing-since-v31)。
- 由于您部署的 StarRocks 实例只有一个 BE 节点，您需要将表属性 replication_num（表示数据副本数量）设置为 1。
- 如果没有指定[table type](../table_design/table_types/table_types.md)，默认会创建一个 Duplicate Key 表。参见[Duplicate Key table](../table_design/table_types/duplicate_key_table.md)。
- 表的列应与您在加载和查询数据教程中将要导入到 StarRocks 的数据字段完全对应。[加载和查询数据](../quick_start/Import_and_query.md)教程上
- 为了确保**生产环境**中的高性能，我们强烈推荐您使用`PARTITION BY`子句为表策划数据分区方案。更多指导请参阅[设计分区和分桶规则](../table_design/Data_distribution.md#design-partitioning-and-bucketing-rules)。

在表创建完成后，您可以使用 DESC 语句查看表的详细信息，并通过执行 [SHOW TABLES](../sql-reference/sql-statements/data-manipulation/SHOW_TABLES.md) 命令查看数据库中的所有表。StarRocks 中的表支持架构变更，您可以查阅 [ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) 获取更多信息。

## 接下来要做的

要了解更多关于 StarRocks 表的概念性细节，请参阅[StarRocks Table Design](../table_design/StarRocks_table_design.md)。

除了本教程展示的特性之外，StarRocks 还支持：

- 多种[数据类型](../sql-reference/sql-statements/data-types/BIGINT.md)
- 多种[table types](../table_design/table_types/table_types.md)
- 灵活的[分区策略](../table_design/Data_distribution.md#dynamic-partition-management)
- 经典的数据库查询索引，包括[位图索引](../using_starrocks/Bitmap_index.md)和[布隆过滤器索引](../using_starrocks/Bloomfilter_index.md)
- [物化视图](../using_starrocks/Materialized_view.md)
