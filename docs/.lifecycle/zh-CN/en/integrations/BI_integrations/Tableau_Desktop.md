---
displayed_sidebar: "Chinese"
---

# Tableau Desktop

Tableau Desktop支持查询和可视化内部数据和StarRocks中的外部数据。

在Tableau Desktop中创建数据库：

![Tableau Desktop](../../assets/BI_tableau_1.png)

请注意以下几点：

- 选择**其他数据库（**JDBC**）** 作为数据源。
- 对于**方言**，选择**MySQL**。
- 对于**URL**，请按照以下MySQL URI格式输入URL：

  ```SQL
  jdbc:mysql://<Host>:<Port>/<Catalog>.<Databases>
  ```

  URL中的参数描述如下：

  - `主机`：StarRocks集群的FE主机IP地址。
  - `端口`：StarRocks集群的FE查询端口，例如`9030`。
  - `目录`：StarRocks集群中的目标目录。支持内部和外部目录。
  - `数据库`：StarRocks集群中的目标数据库。支持内部和外部数据库。
- 配置**用户名**和**密码**。
  - **用户名**：用于登录到StarRocks集群的用户名，例如`admin`。
  - **密码**：用于登录到StarRocks集群的密码。