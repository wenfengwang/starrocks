---
displayed_sidebar: English
---

# 概述

本主题介绍了目录是什么，以及如何利用目录来管理和查询内部数据与外部数据。

StarRocks 从版本2.3起支持目录功能。目录允许您在同一系统中管理内部和外部数据，并提供了一种灵活的方式，便于您轻松查询和分析存储在各种外部系统中的数据。

## 基本概念

- **内部数据**：指的是存储在 StarRocks 中的数据。
- **外部数据**：指的是存储在外部数据源的数据，例如 Apache Hive™、Apache Iceberg、Apache Hudi、Delta Lake 和 JDBC。

## 目录

目前，StarRocks 提供了两种类型的目录：内部目录和外部目录。

![figure1](../../assets/3.8.1.png)

- **内部目录** 管理 StarRocks 的内部数据。例如，如果您执行 CREATE DATABASE 或 CREATE TABLE 语句来创建数据库或表，这些数据库或表会被存储在内部目录中。每个 StarRocks 集群只有一个名为[default_catalog](../catalog/default_catalog.md)的内部目录。

- **外部目录** 相当于外部管理的元数据存储的链接，使 StarRocks 能够直接访问外部数据源。您可以直接查询外部数据，无需进行数据加载或迁移。目前，StarRocks 支持以下类型的外部目录：
  - [Hive 目录](../catalog/hive_catalog.md)：用于查询 Hive 中的数据。
  - [Iceberg 目录](../catalog/iceberg_catalog.md)：用于查询数据从 Iceberg 中。
  - [Hudi 目录](../catalog/hudi_catalog.md)：用于查询 Hudi 中的数据。
  - [Delta Lake 目录](../catalog/deltalake_catalog.md)：用于查询数据从 Delta Lake。
  - [JDBC 目录](../catalog/jdbc_catalog.md)：用于查询兼容 JDBC 的数据源中的数据。

  当您查询外部数据时，StarRocks 会与外部数据源的以下两个组件互动：

  - **Metastore 服务**：前端（FE）使用它来访问外部数据源的元数据。FE 根据元数据生成查询执行计划。
  - **数据存储系统**：用于存储外部数据。无论是分布式文件系统还是对象存储系统，都可以作为数据存储系统来存储各种格式的数据文件。FE 将查询执行计划分发给所有后端（BE），所有 BE 并行扫描目标外部数据，执行计算，然后返回查询结果。

## 访问目录

您可以使用[SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)语句在当前会话中切换到指定的目录。然后，您可以通过该目录查询数据。

## 查询数据

### 查询内部数据

要查询 StarRocks 中的数据，请参见[默认目录](../catalog/default_catalog.md)。

### 查询外部数据

要查询外部数据源中的数据，请参见[Query external data](../catalog/query_external_data.md)。

### 跨目录查询

要进行当前目录下的跨目录联合查询，请使用 catalog_name.database_name 或 catalog_name.database_name.table_name 格式指定您想要查询的数据。

- 当当前会话是 default_catalog.olap_db 时，查询 hive_db 中的 hive_table。

  ```SQL
  SELECT * FROM hive_catalog.hive_db.hive_table;
  ```

- 当当前会话是 hive_catalog.hive_db 时，查询 default_catalog 中的 olap_table。

  ```SQL
   SELECT * FROM default_catalog.olap_db.olap_table;
  ```

- 当当前会话是 hive_catalog.hive_db 时，对 hive_catalog 中的 hive_table 和 default_catalog 中的 olap_table 进行 JOIN 查询。

  ```SQL
  SELECT * FROM hive_table h JOIN default_catalog.olap_db.olap_table o WHERE h.id = o.id;
  ```

- 当当前会话为其他目录时，使用 JOIN 子句对 hive_catalog 中的 hive_table 和 default_catalog 中的 olap_table 进行 JOIN 查询。

  ```SQL
  SELECT * FROM hive_catalog.hive_db.hive_table h JOIN default_catalog.olap_db.olap_table o WHERE h.id = o.id;
  ```
