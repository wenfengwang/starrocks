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

2. 安装 Apache Superset 的最新版本。有关更多信息，请参阅 [从头开始安装 Superset](https://superset.apache.org/docs/installation/installing-superset-from-scratch/)。

## 集成

在 Apache Superset 中创建一个数据库：

![Apache Superset - 1](../../assets/BI_superset_1.png)

![Apache Superset - 2](../../assets/BI_superset_2.png)

 请注意以下几点：

- 在**支持的数据库**中，选择**StarRocks**，作为数据源。
- 对于 **SQLALCHEMY URI**，请输入以下 StarRocks SQLAlchemy URI 格式的 URI：

  ```SQL
  starrocks://<User>:<Password>@<Host>:<Port>/<Catalog>.<Database>
  ```

  URI 中的参数说明如下：

  - `User`：用于登录到您的 StarRocks 集群的用户名，例如 `admin`。
  - `Password`：用于登录到您的 StarRocks 集群的密码。
  - `Host`：您的 StarRocks 集群的 FE 主机 IP 地址。
  - `Port`：您的 StarRocks 集群的 FE 查询端口，例如 `9030`。
  - `Catalog`：您的 StarRocks 集群中的目标目录。支持内部和外部目录。
  - `Database`：您的 StarRocks 集群中的目标数据库。支持内部和外部数据库。
  