---
displayed_sidebar: "中文"
---

# DataGrip

DataGrip 支持查询 StarRocks 中的内部数据和外部数据。

在 DataGrip 中创建数据源时，请注意需要选择 **MySQL** 作为数据源 (**数据源**)。

![DataGrip - 1](../../assets/BI_datagrip_1.png)

![DataGrip - 2](../../assets/BI_datagrip_2.png)

需要设置的参数说明如下：

- **Host**：StarRocks 集群的 FE 主机 IP 地址。
- **Port**：StarRocks 集群的 FE 查询端口，例如 `9030`。
- **Authentication**：认证方式。选择 **用户名 & 密码**。
- **User**：用于登录 StarRocks 集群的用户名，例如 `admin`。
- **Password**：用于登录 StarRocks 集群的用户密码。
- **Database**：StarRocks 集群中要访问的数据源。格式为 `<catalog_name>.<database_name>`。
  - `catalog_name`：StarRocks 集群中目标 Catalog 的名称。内部 Catalog 和外部 Catalog 均支持。
  - `database_name`：StarRocks 集群中目标数据库的名称。内部数据库和外部数据库均支持。