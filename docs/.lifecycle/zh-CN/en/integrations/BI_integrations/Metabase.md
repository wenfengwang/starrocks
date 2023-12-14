---
displayed_sidebar: "English"
---

# Metabase

Metabase支持查询和可视化StarRocks中的内部数据和外部数据。

启动Metabase，并执行以下操作：

1. 在Metabase首页的右上角，单击**设置**图标，然后选择**管理设置**。

   ![Metabase - 管理设置](../../assets/Metabase/Metabase_1.png)

2. 在顶部菜单栏中选择**数据库**。

3. 在**数据库**页面上，单击**添加数据库**。

   ![Metabase - 添加数据库](../../assets/Metabase/Metabase_2.png)

4. 在出现的页面上，配置数据库参数，然后单击**保存**。

   - **数据库类型**：选择**MySQL**。
   - **主机**和**端口**：输入适用于您的情况的主机和端口信息。
   - **数据库名称**：以`<catalog_name>.<database_name>`格式输入数据库名称。在StarRocks v3.2之前的版本中，您只能将StarRocks集群的内部目录与Metabase集成。从StarRocks v3.2开始，您可以将StarRocks集群的内部目录和外部目录都与Metabase集成。
   - **用户名**和**密码**：输入StarRocks集群用户的用户名和密码。

   其他参数不涉及StarRocks。根据您的业务需求进行配置。

   ![Metabase - 配置数据库](../../assets/Metabase/Metabase_3.png)