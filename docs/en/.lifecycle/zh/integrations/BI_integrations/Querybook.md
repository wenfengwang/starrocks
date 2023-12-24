---
displayed_sidebar: English
---

# 查询手册

Querybook 支持在 StarRocks 中查询和可视化内部数据和外部数据。

## 先决条件

确保您已完成以下准备工作：

1. 克隆并下载 Querybook 存储库。

   ```SQL
   git clone git@github.com:pinterest/querybook.git
   cd querybook
   ```

2. 在项目根目录下的`requirements`文件夹中创建名为`local.txt`的文件。

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

访问 [https:///admin/query_engine/](https://localhost:10001/admin/query_engine/) 并添加一个新的查询引擎：

![查询手册](../../assets/BI_querybook_1.png)

请注意以下几点：

- 对于 **语言**，请选择 **StarRocks**。
- 对于 **执行程序**，请选择 **sqlalchemy**。
- 对于 **Connection_string**，输入以下格式的 StarRocks SQLAlchemy URI：

  ```SQL
  starrocks://<User>:<Password>@<Host>:<Port>/<Catalog>.<Database>
  ```

  URI 中的参数说明如下：

  - `User`：用于登录到 StarRocks 集群的用户名，例如 `admin`。
  - `Password`：用于登录到 StarRocks 集群的密码。
  - `Host`：StarRocks 集群的 FE 主机 IP 地址。
  - `Port`：StarRocks 集群的 FE 查询端口，例如 `9030`。
  - `Catalog`：StarRocks 集群中的目标目录。支持内部和外部目录。
  - `Database`：StarRocks 集群中的目标数据库。支持内部和外部数据库。
