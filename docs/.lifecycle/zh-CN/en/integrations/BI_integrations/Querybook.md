---
displayed_sidebar: "Chinese"
---

# Querybook

Querybook支持查询和可视化StarRocks中的内部数据和外部数据。

## 先决条件

确保您已完成以下准备工作：

1. 克隆并下载Querybook存储库。

   ```SQL
   git clone git@github.com:pinterest/querybook.git
   cd querybook
   ```

2. 在项目的根目录下的`requirements`文件夹中创建名为`local.txt`的文件。

   ```SQL
   touch requirements/local.txt
   ```

3. 添加所需的软件包。

   ```SQL
   echo -e "starrocks\nmysqlclient" > requirements/local.txt 
   ```

4. 启动容器。

   ```SQL
   make
   ```

## 集成

访问[https:///admin/query_engine/](https://localhost:10001/admin/query_engine/)并添加新的查询引擎：

![Querybook](../../assets/BI_querybook_1.png)

注意以下几点：

- 对于**语言**，请选择**Starrocks**。
- 对于**执行器**，请选择**sqlalchemy**。
- 对于**Connection_string**，输入StarRocks SQLAlchemy URI格式的URI，如下所示：

  ```SQL
  starrocks://<User>:<Password>@<Host>:<Port>/<Catalog>.<Database>
  ```

  URI中的参数描述如下：

  - `User`：用于登录到StarRocks集群的用户名，例如，`admin`。
  - `Password`：用于登录到StarRocks集群的密码。
  - `Host`：您的StarRocks集群的FE主机IP地址。
  - `Port`：您的StarRocks集群的FE查询端口，例如，`9030`。
  - `Catalog`：StarRocks集群中的目标目录。支持内部目录和外部目录。
  - `Database`：StarRocks集群中的目标数据库。支持内部数据库和外部数据库。