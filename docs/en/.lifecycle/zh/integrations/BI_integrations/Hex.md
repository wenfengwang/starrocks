---
displayed_sidebar: English
---
# 十六进制

Hex 支持在 StarRocks 中查询和可视化内部数据和外部数据。

在 Hex 中添加数据连接。请注意，您必须选择 MySQL 作为连接类型。

![Hex](../../assets/BI_hex_1.png)

您需要配置的参数如下所述：

- **名称**：数据连接的名称。
- **主机和端口**：StarRocks 集群的 FE 主机 IP 地址和 FE 查询端口。查询端口的示例是 `9030`。
- **数据库**：您希望在 StarRocks 集群中访问的数据源。此参数的值采用 `<catalog_name>.<database_name>` 格式。
  - `catalog_name`：StarRocks 集群中目标目录的名称。支持内部和外部目录。
  - `database_name`：StarRocks 集群中目标数据库的名称。支持内部和外部数据库。
- **类型**：您希望使用的身份验证方法。选择 **密码**。
- **用户**：用于登录到 StarRocks 集群的用户名，例如 `admin`。
- **密码**：用于登录到 StarRocks 集群的密码。