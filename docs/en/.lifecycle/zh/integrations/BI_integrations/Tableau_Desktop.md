---
displayed_sidebar: English
---

# Tableau Desktop

Tableau Desktop支持查询和可视化StarRocks中的内部数据和外部数据。

在Tableau Desktop中创建数据库：

![Tableau Desktop](../../assets/BI_tableau_1.png)

请注意以下几点：

- 选择**“其他数据库（JDBC）”**作为数据源。
- 对于**方言**，请选择**MySQL**。
- 对于**URL**，请按照MySQL URI格式输入URL，如下所示：

  ```SQL
  jdbc:mysql://<Host>:<Port>/<Catalog>.<Databases>
  ```

  URL中的参数说明如下：

  - `Host`：StarRocks集群的FE主机IP地址。
  - `Port`：StarRocks集群的FE查询端口，例如，`9030`。
  - `Catalog`：StarRocks集群中的目标目录。支持内部和外部目录。
  - `Database`：StarRocks集群中的目标数据库。支持内部和外部数据库。
- 配置**用户名**和**密码**。
  - **用户名**：用于登录StarRocks集群的用户名，例如`admin`。
  - **密码**：用于登录StarRocks集群的密码。
