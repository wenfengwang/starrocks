---
displayed_sidebar: "中文"
---

# DBeaver

作为一款 SQL 客户端应用软件，DBeaver 提供了强大的数据库管理能力，通过 Assistant 帮助您快速连接数据库。

## 前提条件

确保已安装 DBeaver。

您可以访问 [https://dbeaver.io](https://dbeaver.io/) 下载安装 DBeaver 社区版，也可以访问 [https://dbeaver.com](https://dbeaver.com/) 下载安装 DBeaver PRO 版。

## 集成

按如下步骤连接数据库：

1. 启动 DBeaver。
2. 在 DBeaver 窗口左上角单击加号 (**+**) 图标，或者在菜单栏选择 **Database** > **New Database Connection**，打开 Assistant。

   ![DBeaver - 访问助手](../../assets/IDE_dbeaver_1.png)

   ![DBeaver - 访问助手](../../assets/IDE_dbeaver_2.png)

3. 选择 MySQL 驱动器。

   在 **选择您的数据库** 窗口，您可以看到所有支持的驱动器 (Driver)。在窗口左侧单击 **Analytical** 可以快速找到 MySQL 驱动器，然后双击 **MySQL** 图标。

   ![DBeaver - 选择您的数据库](../../assets/IDE_dbeaver_3.png)

4. 配置数据库连接。

   在 **连接设置** 窗口，进入 **Main** 页签，并配置以下连接信息（以下信息均为必选）：

   - **Server Host**：StarRocks 集群的 FE 主机 IP 地址。
   - **Port**：StarRocks 集群的 FE 查询端口，如 `9030`。
   - **Database**：StarRocks 集群中的目标数据库。内部数据库和外部数据库均支持，但是外部数据库可能功能不完备。
   - **Username**：用于登录 StarRocks 集群的用户名，如 `admin`。
   - **Password**：用于登录 StarRocks 集群的用户密码。

   ![DBeaver - 连接设置 - 主要 选项卡](../../assets/IDE_dbeaver_4.png)

   在 **驱动程序属性** 选项卡，您还可以查看 MySQL 驱动程序的各个属性，并且可以单击某个属性所在行的 **Value** 列、然后对该属性进行编辑。

   ![DBeaver - 连接设置 - 驱动程序属性 选项卡](../../assets/IDE_dbeaver_5.png)

5. 测试数据库连接。

   单击 **测试连接** 验证数据库连接信息的准确性。系统返回如下对话框，提醒您确认配置信息。单击 **确定** 即确认配置信息准确无误。然后，单击 **完成** 完成连接配置。

   ![DBeaver - 测试连接](../../assets/IDE_dbeaver_6.png)

6. 连接数据库。

   数据库连接建立以后，您可以在左侧的数据库连接导航树里看到该连接，并且可以通过 DBeaver 快速连接到您的数据库。

   ![DBeaver - 连接数据库](../../assets/IDE_dbeaver_7.png)