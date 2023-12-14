---
displayed_sidebar: "Chinese"
---

# DataGrip

DataGrip支持查询StarRocks中的内部数据和外部数据。

在DataGrip中创建数据源。请注意，您必须将数据源选为MySQL。

![DataGrip - 1](../../assets/BI_datagrip_1.png)

![DataGrip - 2](../../assets/BI_datagrip_2.png)

您需要配置的参数如下所述：

- **主机**：您的StarRocks集群的FE主机IP地址。
- **端口**：您的StarRocks集群的FE查询端口，例如`9030`。
- **认证**：您想要使用的认证方法。请选择**用户名和密码**。
- **用户**：用于登录到您的StarRocks集群的用户名，例如`admin`。
- **密码**：用于登录到您的StarRocks集群的密码。
- **数据库**：您想要在StarRocks集群中访问的数据源。此参数的值采用`<catalog_name>.<database_name>`的格式。
  - `catalog_name`：您StarRocks集群中目标目录的名称。支持内部目录和外部目录。
  - `database_name`：您StarRocks集群中目标数据库的名称。支持内部数据库和外部数据库。