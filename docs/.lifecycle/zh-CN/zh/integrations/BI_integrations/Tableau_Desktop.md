---
displayed_sidebar: "中文"
---

# Tableau 桌面版

Tableau 桌面版支持对 StarRocks 的内部数据和外部数据进行查询和可视化处理。

在 Tableau 桌面版中创建数据库：

![Tableau 桌面版](../../assets/BI_tableau_1.png)

注意以下几点：

- 选择 **其他数据库(****JDBC****)** 作为数据源。
- 在 **方言** 里选择 **MySQL**。
- 在 **URL** 里，按如下 MySQL URI 格式输入 URL：

  ```SQL
  jdbc:mysql://<Host>:<Port>/<Catalog>.<Databases>
  ```

  URL 参数说明如下：

  - `Host`：StarRocks 集群的 FE 主机 IP 地址。
  - `Port`：StarRocks 集群的 FE 查询端口，例如 `9030`。
  - `Catalog`：StarRocks 集群中的目标 Catalog。内部 Catalog 和外部 Catalog 均支持。
  - `Database`：StarRocks 集群中的目标数据库。内部数据库和外部数据库均支持。

- 在 **用户名** 和 **密码** 里输入用户名和密码。
  - **用户名**：用于登录 StarRocks 集群的用户名，例如 `admin`。
  - **密码**：用于登录 StarRocks 集群的用户密码。