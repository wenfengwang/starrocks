---
displayed_sidebar: English
---

# 数据夹改为DataGrip

DataGrip 支持查询 StarRocks 中的内部数据和外部数据。

在 DataGrip 中创建数据源时，请注意必须选择 MySQL 作为数据源类型。

![DataGrip - 1](../../assets/BI_datagrip_1.png)

![DataGrip - 2](../../assets/BI_datagrip_2.png)

您需要配置的参数如下所述：

- **主机**：指的是您的 StarRocks 集群的 FE（前端）主机 IP 地址。
- **端口**：指的是您的 StarRocks 集群的 FE 查询端口，例如，`9030`。
- **认证方式**：您希望使用的认证方法。请选择**用户名 & 密码**。
- **用户**：用来登录您的 StarRocks 集群的用户名，例如 `admin`。
- **密码**：用来登录您的 StarRocks 集群的密码。
- **数据库**：您希望在StarRocks集群中访问的数据源。此参数的值应该按照`\u003ccatalog_name\u003e.\u003cdatabase_name\u003e`的格式填写。
  - catalog_name：您的 StarRocks 集群中目标目录的名称，支持内部目录和外部目录。
  - database_name：您的 StarRocks 集群中目标数据库的名称，支持内部数据库和外部数据库。
