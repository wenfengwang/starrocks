---
displayed_sidebar: English
---

# 阿帕奇Superset

Apache Superset 支持对 StarRocks 中的内部和外部数据进行查询和可视化处理。

## 准备工作

请确保您已经完成了以下安装步骤：

1. 在您的 Apache Superset 服务器上安装 StarRocks 的 Python 客户端。

   ```SQL
   pip install starrocks
   ```

2. 安装最新版本的 Apache Superset。更多安装信息，请参考[从零开始安装Superset](https://superset.apache.org/docs/installation/installing-superset-from-scratch/)。

## 集成

在 Apache Superset 中创建一个数据库：

![Apache Superset - 1](../../assets/BI_superset_1.png)

![Apache Superset - 2](../../assets/BI_superset_2.png)

注意以下几点：

- 在**支持的数据库**中，选择**StarRocks**作为数据源。
- 对于**SQLALCHEMY URI**，请按照如下 StarRocks SQLAlchemy URI 格式输入 URI：

  ```SQL
  starrocks://<User>:<Password>@<Host>:<Port>/<Catalog>.<Database>
  ```
  URI 参数的描述如下：

  - 用户：登录 StarRocks 集群所用的用户名，例如 admin。
  - 密码：登录 StarRocks 集群所用的密码。
  - 主机：StarRocks 集群的 FE（前端）主机 IP 地址。
  - 端口：StarRocks 集群的 FE 查询端口，例如 9030。
  - 目录：您的 StarRocks 集群中的目标目录。支持内部和外部目录。
  - 数据库：您的 StarRocks 集群中的目标数据库。支持内部和外部数据库。
