---
displayed_sidebar: English
---

# DBeaver

DBeaver 是一个 SQL 客户端软件应用和数据库管理工具，它提供了一个有用的助手，引导您完成连接到数据库的过程。

## 先决条件

确保您已安装 DBeaver。

您可以在 [https://dbeaver.io](https://dbeaver.io/) 下载 DBeaver 社区版，或在 [https://dbeaver.com](https://dbeaver.com/) 下载 DBeaver PRO 版。

## 集成

请按照以下步骤连接到数据库：

1. 启动 DBeaver。

2. 单击 DBeaver 窗口左上角的加号 (**+**) 图标或在菜单栏中选择 **数据库** > **新建数据库连接** 以访问助手。

   ![DBeaver - 访问助手](../../assets/IDE_dbeaver_1.png)

   ![DBeaver - 访问助手](../../assets/IDE_dbeaver_2.png)

3. 选择 MySQL 驱动。

   在**选择您的数据库**步骤中，您将看到可用驱动的列表。单击左侧面板中的**分析**快速定位 MySQL 驱动。然后，双击 **MySQL** 图标。

   ![DBeaver - 选择您的数据库](../../assets/IDE_dbeaver_3.png)

4. 配置数据库连接。

   在**连接设置**步骤中，转到**主**标签页并配置以下基本连接设置：

   - **服务器主机**：您 StarRocks 集群的 FE 主机 IP 地址。
   - **端口**：您的 StarRocks 集群的 FE 查询端口，例如 `9030`。
   - **数据库**：您 StarRocks 集群中的目标数据库。支持内部和外部数据库，但外部数据库的功能可能不完整。
   - **用户名**：用于登录到 StarRocks 集群的用户名，例如 `admin`。
   - **密码**：用于登录到您的 StarRocks 集群的密码。

   ![DBeaver - 连接设置 - 主标签页](../../assets/IDE_dbeaver_4.png)

   如有必要，您还可以在**驱动属性**标签页上查看和编辑 MySQL 驱动的属性。要编辑特定属性，请单击该属性的**值**列中的行。

   ![DBeaver - 连接设置 - 驱动属性标签页](../../assets/IDE_dbeaver_5.png)

5. 测试数据库连接。

   单击 **测试连接** 以验证连接设置的准确性。将出现一个显示 MySQL 驱动信息的对话框。在对话框中单击 **确定** 以确认信息。成功配置连接设置后，单击 **完成** 以完成过程。

   ![DBeaver - 测试连接](../../assets/IDE_dbeaver_6.png)

6. 连接到数据库。

   连接建立后，您可以在左侧的数据库连接树中查看它，DBeaver 可以有效地连接到数据库。

   ![DBeaver - 连接数据库](../../assets/IDE_dbeaver_7.png)