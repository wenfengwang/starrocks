---
displayed_sidebar: "Chinese"
---

# 十六进制

Hex支持查询和可视化StarRocks中的内部数据和外部数据。

在Hex中添加数据连接。请注意，您必须选择MySQL作为连接类型。

![Hex](../../assets/BI_hex_1.png)

您需要配置的参数如下所述：

- **名称**：数据连接的名称。
- **主机和端口**：StarRocks集群的前端主机IP地址和FE查询端口。示例查询端口为`9030`。
- **数据库**：您要在StarRocks集群中访问的数据源。此参数的值采用`<catalog_name>.<database_name>`格式。
  - `catalog_name`：StarRocks集群中目标目录的名称。支持内部目录和外部目录。
  - `database_name`：StarRocks集群中目标数据库的名称。支持内部数据库和外部数据库。
- **类型**：您希望使用的身份验证方式。选择**密码**。
- **用户**：用于登录到StarRocks集群的用户名，例如`admin`。
- **密码**：用于登录到StarRocks集群的密码。