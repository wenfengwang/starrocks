---
displayed_sidebar: English
---

# Hex

Hex 支持在 StarRocks 中查询和可视化内部及外部数据。

在 Hex 中添加数据连接。请注意，您必须选择 MySQL 作为连接类型。

![Hex](../../assets/BI_hex_1.png)

您需要配置的参数如下所述：

- **名称**：数据连接的名称。
- **主机与端口**：您的 StarRocks 集群的 FE 主机 IP 地址及 FE 查询端口。一个示例查询端口是 `9030`。
- **数据库**：您希望在 StarRocks 集群中访问的数据源。此参数的值应使用 `
<catalog_name>
.
<database_name>` 的格式。
  - catalog_name：您的 StarRocks 集群中目标目录的名称，支持内部和外部目录。
  - database_name：您的 StarRocks 集群中目标数据库的名称，支持内部和外部数据库。
- **类型**：您希望使用的认证方式。请选择**密码**验证。
- **用户**：用来登录您的 StarRocks 集群的用户名，例如 `admin`。
- **密码**：用来登录您的 StarRocks 集群的密码。
