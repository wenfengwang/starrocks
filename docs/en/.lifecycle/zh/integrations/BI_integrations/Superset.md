---
displayed_sidebar: English
---

# Apache Superset

Apache Superset 支持在 StarRocks 中查询和可视化内部数据和外部数据。

## 先决条件

确保您已完成以下安装：

1. 在 Apache Superset 服务器上安装 StarRocks 的 Python 客户端。

   ```SQL
   pip install starrocks
   ```

2. 安装最新版本的 Apache Superset。有关更多信息，请参阅[从头开始安装 Superset](https://superset.apache.org/docs/installation/installing-superset-from-scratch/)。

## 集成

在 Apache Superset 中创建数据库：

![Apache Superset - 1](../../assets/BI_superset_1.png)

![Apache Superset - 2](../../assets/BI_superset_2.png)

请注意以下几点：

- 对于**支持的数据库**，选择**StarRocks**，它将用作数据源。
- 对于 **SQLALCHEMY_URI**，请输入符合 StarRocks SQLAlchemy URI 格式的 URI，如下所示：

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