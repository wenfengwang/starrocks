---
displayed_sidebar: English
---

# Hex

Hex 支持在 StarRocks 中查询和可视化内部数据与外部数据。

在 Hex 中添加数据连接。请注意，您必须选择 MySQL 作为连接类型。

![Hex](../../assets/BI_hex_1.png)

您需要配置的参数如下所述：

- **名称**：数据连接的名称。
- **主机 & 端口**：您的 StarRocks 集群的 FE 主机 IP 地址和 FE 查询端口。例如查询端口是 `9030`。
- **数据库**：您想要在 StarRocks 集群中访问的数据源。此参数的值格式为 `<catalog_name>.<database_name>`。
  - `catalog_name`：您的 StarRocks 集群中目标目录的名称。支持内部和外部目录。
  - `database_name`：您的 StarRocks 集群中目标数据库的名称。支持内部和外部数据库。
- **类型**：您想要使用的认证方法。请选择 **Password**。
- **用户**：用于登录您的 StarRocks 集群的用户名，例如 `admin`。
- **密码**：用于登录您的 StarRocks 集群的密码。