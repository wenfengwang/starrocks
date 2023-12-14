---
displayed_sidebar: "Chinese"
---

# Apache Superset

Apache Superset支持在StarRocks中查询和可视化内部数据和外部数据。

## 先决条件

确保您已完成以下安装：

1. 在您的Apache Superset服务器上安装StarRocks的Python客户端。

   ```SQL
   pip install starrocks
   ```

2. 安装最新版本的Apache Superset。有关更多信息，请参阅[从头开始安装Superset](https://superset.apache.org/docs/installation/installing-superset-from-scratch/)。

## 集成

在Apache Superset中创建一个数据库：

![Apache Superset - 1](../../assets/BI_superset_1.png)

![Apache Superset - 2](../../assets/BI_superset_2.png)

 请注意以下要点：

- 对于**支持的数据库**，请选择**StarRocks**，这将被用作数据源。
- 对于**SQLALCHEMY** **URI**，输入如下StarRocks SQLAlchemy URI格式的URI：

  ```SQL
  starrocks://<User>:<Password>@<Host>:<Port>/<Catalog>.<Database>
  ```

  URI中的参数说明如下：

  - `User`：用于登录到您的StarRocks集群的用户名，例如`admin`。
  - `Password`：用于登录到您的StarRocks集群的密码。
  - `Host`：您的StarRocks集群的FE主机IP地址。
  - `Port`：您的StarRocks集群的FE查询端口，例如`9030`。
  - `Catalog`：StarRocks集群中的目标catalog。支持内部和外部catalog。
  - `Database`：StarRocks集群中的目标数据库。支持内部和外部数据库。