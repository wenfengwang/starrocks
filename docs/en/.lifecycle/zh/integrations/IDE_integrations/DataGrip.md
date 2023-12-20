---
displayed_sidebar: English
---

# DataGrip

DataGrip 支持查询 StarRocks 中的内部数据和外部数据。

在 DataGrip 中创建数据源。注意，必须选择 MySQL 作为数据源。

![DataGrip - 1](../../assets/BI_datagrip_1.png)

![DataGrip - 2](../../assets/BI_datagrip_2.png)

您需要配置的参数如下所述：

- **Host**：您的 StarRocks 集群的 FE 主机 IP 地址。
- **Port**：您的 StarRocks 集群的 FE 查询端口，例如 `9030`。
- **Authentication**：您想要使用的认证方法。请选择 **Username & Password**。
- **User**：用于登录您的 StarRocks 集群的用户名，例如 `admin`。
- **Password**：用于登录您的 StarRocks 集群的密码。
- **Database**：您想要在您的 StarRocks 集群中访问的数据源。此参数的值应该是 `<catalog_name>.<database_name>` 格式。
  - `catalog_name`：您的 StarRocks 集群中目标目录的名称。支持内部和外部目录。
  - `database_name`：您的 StarRocks 集群中目标数据库的名称。支持内部和外部数据库。