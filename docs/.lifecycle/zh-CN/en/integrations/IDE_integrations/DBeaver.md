---
displayed_sidebar: "Chinese"
---

# DBeaver

DBeaver是一个SQL客户端软件应用和数据库管理工具，它提供了一个有用的助手，可以指导您完成连接到数据库的过程。

## 先决条件

确保您已安装DBeaver。

您可以在 [https://dbeaver.io](https://dbeaver.io/) 下载DBeaver社区版，或在 [https://dbeaver.com](https://dbeaver.com/) 下载DBeaver专业版。

## 集成

请按照以下步骤连接到数据库：

1. 启动DBeaver。

2. 单击DBeaver窗口左上角的加号（**+**）图标，或选择**Database** > **New Database Connection**菜单，以访问助手。

   ![DBeaver - 访问助手](../../assets/IDE_dbeaver_1.png)

   ![DBeaver - 访问助手](../../assets/IDE_dbeaver_2.png)

3. 选择MySQL驱动程序。

   在**选择您的数据库**步骤中，您将看到一个可用驱动程序的列表。单击左侧窗格中的**解析**，快速定位MySQL驱动程序。然后，双击**MySQL**图标。

   ![DBeaver - 选择您的数据库](../../assets/IDE_dbeaver_3.png)

4. 配置数据库连接。

   在**连接设置**步骤中，转到**Main**选项卡，并配置以下必要的连接设置：

   - **服务器主机**：您StarRocks集群的FE主机IP地址。
   - **端口**：您StarRocks集群的FE查询端口，例如，`9030`。
   - **数据库**：您StarRocks集群中的目标数据库。支持内部数据库和外部数据库，但外部数据库的功能可能不完整。
   - **用户名**：用于登录到您StarRocks集群的用户名，例如，`admin`。
   - **密码**：用于登录到您StarRocks集群的密码。

   ![DBeaver - 连接设置 - Main 选项卡](../../assets/IDE_dbeaver_4.png)

   如果需要，您还可以在**Driver属性**选项卡上查看和编辑MySQL驱动程序的属性。要编辑特定属性，请单击该属性的**值**列中的行。

   ![DBeaver - 连接设置 - Driver属性选项卡](../../assets/IDE_dbeaver_5.png)

5. 测试数据库连接。

   单击**Test Connection**以验证连接设置的准确性。将显示一个显示MySQL驱动程序信息的对话框。单击对话框中的**确定**以确认信息。成功配置连接设置后，单击**完成**完成该过程。

   ![DBeaver - 测试连接](../../assets/IDE_dbeaver_6.png)

6. 连接数据库。

   连接建立后，您可以在左侧数据库连接树中查看，并且DBeaver可以有效连接到数据库。

   ![DBeaver - 连接数据库](../../assets/IDE_dbeaver_7.png)