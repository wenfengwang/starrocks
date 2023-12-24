---
displayed_sidebar: English
---

# Metabase

Metabase 支持在 StarRocks 中查询和可视化内部数据和外部数据。

启动 Metabase 并执行以下操作：

1. 在 Metabase 主页的右上角，单击**设置**图标，然后选择**管理员设置**。

   ![Metabase - 管理设置](../../assets/Metabase/Metabase_1.png)

2. 在顶部菜单栏中选择**数据库**。

3. 在**数据库**页面上，单击**添加数据库**。

   ![Metabase - 添加数据库](../../assets/Metabase/Metabase_2.png)

4. 在弹出的页面中，配置数据库参数，然后单击**保存**。

   - **数据库类型**：选择**MySQL**。
   - **主机和端口**：输入适合您的用例的主机和端口信息。
   - **数据库名称**：按照`<catalog_name>.<database_name>`格式输入数据库名称。在 StarRocks v3.2 之前的版本中，您只能将 StarRocks 集群的内部目录与 Metabase 集成。从 StarRocks v3.2 开始，您可以将 StarRocks 集群的内部目录和外部目录与 Metabase 集成。
   - **用户名**和**密码**：输入您的 StarRocks 集群用户的用户名和密码。

   其他参数与 StarRocks 无关。根据您的业务需求进行配置。

   ![Metabase - 配置数据库](../../assets/Metabase/Metabase_3.png)