---
displayed_sidebar: English
---

# Apache Superset 支持

[Apache Superset](https://superset.apache.org) 是一个现代化的数据探索和可视化平台。它利用 [SQLAlchemy](https://github.com/StarRocks/starrocks/tree/main/contrib/starrocks-python-client/starrocks) 来查询数据。

尽管可以使用 [MySQL 方言](https://superset.apache.org/docs/databases/mysql)，但它不支持 `largeint`。因此，我们开发了[StarRocks 方言](https://github.com/StarRocks/starrocks/tree/main/contrib/starrocks-python-client/starrocks/sqlalchemy)。

## 环境配置

- Python 3.x
- mysqlclient（通过 pip 安装 mysqlclient）
- [Apache Superset](https://superset.apache.org)

注意：若未安装 mysqlclient，将会抛出异常：

```plain
No module named 'MySQLdb'
```

## 安装说明

由于该方言并未直接集成到 SQLAlchemy 中，需要从源码进行安装。

如果你是通过 Docker 安装 Superset，需要以 root 用户身份安装 sqlalchemy-starrocks。

从[源码](https://github.com/StarRocks/starrocks/tree/main/contrib/starrocks-python-client/starrocks)安装

```shell
pip install .
```

卸载方法

```shell
pip uninstall sqlalchemy-starrocks
```

## 使用指南

要通过 SQLAlchemy 连接 StarRocks，可以使用以下的 URL 模式：

```shell
starrocks://<username>:<password>@<host>:<port>/<database>[?charset=utf8]
```

## 基础示例

### Sqlalchemy 示例

建议使用 Python 3.x 版本连接 StarRocks 数据库，例如：

```python
from sqlalchemy import create_engine
import pandas as pd
conn = create_engine('starrocks://root:@x.x.x.x:9030/superset_db?charset=utf8')
sql = """select * from xxx"""
df = pd.read_sql(sql, conn)
```

### Superset 使用示例

在 Superset 中，选择“其他数据库”，并设置 URL 如下：

```shell
starrocks://root:@x.x.x.x:9030/superset_db?charset=utf8
```
