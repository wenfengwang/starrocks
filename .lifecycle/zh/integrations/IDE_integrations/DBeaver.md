---
displayed_sidebar: English
---

# 数据库海狸（DBeaver）

DBeaver是一款SQL客户端软件应用和数据库管理工具，它提供了一个实用的向导，帮助您完成连接数据库的整个过程。

## 先决条件

请确保您已安装了DBeaver。

您可以在 [https://dbeaver.io](https://dbeaver.io/) 下载DBeaver社区版，或者在 [https://dbeaver.com](https://dbeaver.com/) 下载DBeaver专业版。

## 集成

按照以下步骤来连接数据库：

1. 1. 打开DBeaver。

2. 点击DBeaver窗口左上角的加号（**+**）图标，或者在菜单栏选择**数据库** \"> **新建数据库连接**来打开向导。

   ![DBeaver - 打开向导](../../assets/IDE_dbeaver_1.png)

   ![DBeaver - 打开向导](../../assets/IDE_dbeaver_2.png)

3. 3. 选择MySQL驱动。

   在**选择您的数据库**步骤中，您会看到一个可用驱动的列表。点击**分析**可以快速定位到MySQL驱动。然后，双击**MySQL**图标。

   ![DBeaver - 选择您的数据库](../../assets/IDE_dbeaver_3.png)

4. 4. 配置数据库连接。

   在**连接设置**步骤中，切换到**主**标签页，并配置以下必要的连接设置：

   - **服务器主机**：您的StarRocks集群的FE主机IP地址。
   - **端口**：您的StarRocks集群的FE查询端口，例如`9030`。
   - **数据库**：您的StarRocks集群中的目标数据库。支持内部和外部数据库，但外部数据库的功能可能不齐全。
   - **用户名**：用来登录您的StarRocks集群的用户名，例如 `admin`。
   - **密码**：用来登录您的StarRocks集群的密码。

   ![DBeaver - 连接设置 - 主标签页](../../assets/IDE_dbeaver_4.png)

   如果需要，您也可以在“**驱动程序属性**”标签页查看和编辑MySQL驱动的属性。要编辑特定属性，点击该属性的“**值**”列中的行。

   ![DBeaver - 连接设置 - 驱动程序属性标签页](../../assets/IDE_dbeaver_5.png)

5. 5. 测试数据库连接。

   点击**测试连接**来确认连接设置是否准确。会弹出一个显示MySQL驱动信息的对话框。在对话框中点击**确定**以确认信息。在成功配置连接设置之后，点击**完成**来完成设置过程。

   ![DBeaver - 测试连接](../../assets/IDE_dbeaver_6.png)

6. 6. 连接数据库。

   一旦连接建立，您可以在左侧的数据库连接树中看到它，此时DBeaver可以有效地连接到数据库。

   ![DBeaver - 连接数据库](../../assets/IDE_dbeaver_7.png)
