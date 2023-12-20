---
displayed_sidebar: English
---

# Querybook

Querybook 支持在 StarRocks 中查询和可视化内部数据与外部数据。

## 先决条件

确保您已完成以下准备工作：

1. 克隆并下载 Querybook 仓库。

   ```SQL
   git clone git@github.com:pinterest/querybook.git
   cd querybook
   ```

2. 在项目根目录的 `requirements` 文件夹下创建一个名为 `local.txt` 的文件。

   ```SQL
   touch requirements/local.txt
   ```

3. 添加所需的包。

   ```SQL
   echo -e "starrocks\nmysqlclient" > requirements/local.txt 
   ```

4. 启动容器。

   ```SQL
   make
   ```

## 集成

访问 [https://localhost:10001/admin/query_engine/](https://localhost:10001/admin/query_engine/) 并添加一个新的查询引擎：

![Querybook](../../assets/BI_querybook_1.png)

请注意以下几点：

- 对于**Language**，选择 **Starrocks**。
- 对于**Executor**，选择 **sqlalchemy**。
- 对于 **Connection_string**，输入一个 URI，格式应符合 StarRocks SQLAlchemy URI，如下所示：

  ```SQL
  starrocks://<User>:<Password>@<Host>:<Port>/<Catalog>.<Database>
  ```
  URI 中的参数说明如下：

  - `User`：用于登录 StarRocks 集群的用户名，例如 `admin`。
  - `Password`：用于登录 StarRocks 集群的密码。
  - `Host`：StarRocks 集群的 FE 主机 IP 地址。
  - `Port`：您的 StarRocks 集群的 FE 查询端口，例如 `9030`。
  - `Catalog`：StarRocks 集群中的目标目录。支持内部和外部目录。
  - `Database`：StarRocks 集群中的目标数据库。支持内部和外部数据库。