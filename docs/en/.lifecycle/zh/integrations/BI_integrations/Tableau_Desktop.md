---
displayed_sidebar: English
---

# Tableau Desktop

Tableau Desktop 支持在 StarRocks 中查询和可视化内部数据和外部数据。

在 Tableau Desktop 中创建数据库：

![Tableau Desktop](../../assets/BI_tableau_1.png)

请注意以下几点：

- 选择**其他数据库(JDBC)**作为数据源。
- 对于**方言**，请选择**MySQL**。
- 对于 **URL**，请输入以下格式的 MySQL URI：

  ```SQL
  jdbc:mysql://<Host>:<Port>/<Catalog>.<Database>
  ```
  URL中的参数说明如下：

  - `Host`：StarRocks 集群的 FE 主机 IP 地址。
  - `Port`：StarRocks 集群的 FE 查询端口，例如 `9030`。
  - `Catalog`：StarRocks 集群中的目标目录。支持内部和外部目录。
  - `Database`：StarRocks 集群中的目标数据库。支持内部和外部数据库。
- 配置**用户名**和**密码**。
  - **用户名**：用于登录 StarRocks 集群的用户名，例如 `admin`。
  - **密码**：用于登录 StarRocks 集群的密码。