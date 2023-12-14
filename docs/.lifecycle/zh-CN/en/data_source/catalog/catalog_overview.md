---
displayed_sidebar: "Chinese"
---

# 概述

本主题描述了目录是什么，以及如何使用目录管理和查询内部数据和外部数据。

StarRocks 从 v2.3 版本开始支持目录功能。目录使您能够在一个系统中管理内部和外部数据，并为您提供了一种灵活的方式，轻松查询和分析存储在各种外部系统中的数据。

## 基本概念

- **内部数据**：指存储在 StarRocks 中的数据。
- **外部数据**：指存储在外部数据源中的数据，例如 Apache Hive™、Apache Iceberg、Apache Hudi、Delta Lake 和 JDBC。

## 目录

目前，StarRocks 提供两种类型的目录：内部目录和外部目录。

![图1](../../assets/3.8.1.png)

- **内部目录** 管理 StarRocks 的内部数据。例如，如果执行 CREATE DATABASE 或 CREATE TABLE 语句创建数据库或表，则该数据库或表存储在内部目录中。每个 StarRocks 集群只有一个名为 [default_catalog](../catalog/default_catalog.md) 的内部目录。

- **外部目录** 类似于外部管理的元数据存储的链接，使 StarRocks 直接访问外部数据源。您可以直接查询外部数据，无需进行数据加载或迁移。目前，StarRocks 支持以下类型的外部目录：
  - [Hive 目录](../catalog/hive_catalog.md)：用于从 Hive 查询数据。
  - [Iceberg 目录](../catalog/iceberg_catalog.md)：用于从 Iceberg 查询数据。
  - [Hudi 目录](../catalog/hudi_catalog.md)：用于从 Hudi 查询数据。
  - [Delta Lake 目录](../catalog/deltalake_catalog.md)：用于从 Delta Lake 查询数据。
  - [JDBC 目录](../catalog/jdbc_catalog.md)：用于从兼容 JDBC 的数据源查询数据。

  在查询外部数据时，当您查询外部数据时，StarRocks 与外部数据源的以下两个组件进行交互：

  - **元数据服务**：由 FEs 用于访问外部数据源的元数据。 FEs 基于元数据生成查询执行计划。
  - **数据存储系统**：用于存储外部数据。分布式文件系统和对象存储系统都可以用作数据存储系统，以在各种格式中存储数据文件。 FEs 将查询执行计划分发给所有 BEs 后，所有 BEs 并行扫描目标外部数据，执行计算，然后返回查询结果。


## 访问目录

可以使用 [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) 语句在当前会话中切换到指定的目录。然后，您可以使用该目录查询数据。

## 查询数据

### 查询内部数据

要查询 StarRocks 中的数据，请参见[默认目录](../catalog/default_catalog.md)。

### 查询外部数据

要查询来自外部数据源的数据，请参见[查询外部数据](../catalog/query_external_data.md)。

### 跨目录查询

要执行当前目录中的跨目录联邦查询，请以 `catalog_name.database_name` 或 `catalog_name.database_name.table_name` 格式指定要查询的数据。

- 当当前会话为 `default_catalog.olap_db` 时，查询 `hive_db` 中的 `hive_table`。

    ```SQL
    SELECT * FROM hive_catalog.hive_db.hive_table;
    ```

- 当当前会话为 `hive_catalog.hive_db` 时，在 `default_catalog` 中查询 `olap_db` 中的 `olap_table`。

   ```SQL
    SELECT * FROM default_catalog.olap_db.olap_table;
    ```

- 当当前会话为 `hive_catalog.hive_db` 时，在 `hive_catalog` 中的 `hive_table` 和 `default_catalog` 中的 `olap_table` 进行 JOIN 查询。

    ```SQL
    SELECT * FROM hive_table h JOIN default_catalog.olap_db.olap_table o WHERE h.id = o.id;
    ```

- 当前会话为另一个目录时，可以使用 JOIN 子句在 `hive_catalog` 中的 `hive_table` 和 `default_catalog` 中的 `olap_table` 上执行 JOIN 查询。

    ```SQL
    SELECT * FROM hive_catalog.hive_db.hive_table h JOIN default_catalog.olap_db.olap_table o WHERE h.id = o.id;
    ```