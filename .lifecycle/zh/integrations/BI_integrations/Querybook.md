---
displayed_sidebar: English
---

# 查询手册

Querybook 支持在 StarRocks 中查询和可视化内部及外部数据。

## 先决条件

请确保您已完成以下准备工作：

1. 克隆并下载 Querybook 仓库。

   ```SQL
   git clone git@github.com:pinterest/querybook.git
   cd querybook
   ```

2. 在项目根目录下的 requirements 文件夹中创建一个名为 local.txt 的文件。

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

访问 [https:///admin/query_engine/](https://localhost:10001/admin/query_engine/) 并添加一个新的查询引擎：

![Querybook](../../assets/BI_querybook_1.png)

请注意以下几点：

- 对于**语言**，选择**Starrocks**。
- 对于**Executor**，选择**sqlalchemy**。
- 对于**连接字符串**，按照下面的 StarRocks SQLAlchemy URI 格式输入 URI：

  ```SQL
  starrocks://<User>:<Password>@<Host>:<Port>/<Catalog>.<Database>
  ```
  URI 中的参数描述如下：

  - 用户：用于登录 StarRocks 集群的用户名，例如 admin。
  - 密码：用于登录 StarRocks 集群的密码。
  - 主机：StarRocks 集群的 FE 主机 IP 地址。
  - 端口：StarRocks 集群的 FE 查询端口，例如 9030。
  - 目录：您在 StarRocks 集群中的目标目录。支持内部和外部目录。
  - 数据库：您在 StarRocks 集群中的目标数据库。支持内部和外部数据库。
