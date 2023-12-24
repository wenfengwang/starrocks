---
displayed_sidebar: English
---

# DataGrip

DataGrip 支持在 StarRocks 中查询内部数据和外部数据。

在 DataGrip 中创建一个数据源。请注意，您必须选择 MySQL 作为数据源。

![DataGrip - 1](../../assets/BI_datagrip_1.png)

![DataGrip - 2](../../assets/BI_datagrip_2.png)

您需要配置的参数如下：

- **主机**：StarRocks 集群的 FE 主机 IP 地址。
- **端口**：StarRocks 集群的 FE 查询端口，例如 `9030`。
- **身份验证**：您想要使用的身份验证方法。请选择**用户名和密码**。
- **用户**：用于登录到 StarRocks 集群的用户名，例如 `admin`。
- **密码**：用于登录到 StarRocks 集群的密码。
- **数据库**：您想要在 StarRocks 集群中访问的数据源。此参数的值采用 `<catalog_name>.<database_name>` 格式。
  - `catalog_name`：StarRocks 集群中目标目录的名称。支持内部和外部目录。
  - `database_name`：StarRocks 集群中目标数据库的名称。支持内部和外部数据库。
