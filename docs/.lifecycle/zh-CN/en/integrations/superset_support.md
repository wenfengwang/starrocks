---
displayed_sidebar: "Chinese"
---

# Superset 支持

[Apache Superset](https://superset.apache.org) 是一个现代化的数据探索和可视化平台。它使用[SQLAlchemy](https://github.com/StarRocks/starrocks/tree/main/contrib/starrocks-python-client/starrocks) 来查询数据。

虽然[Mysql Dialect](https://superset.apache.org/docs/databases/mysql) 可以使用，但不支持 `largeint`。因此，我们开发了[StarRocks Dialect](https://github.com/StarRocks/starrocks/tree/main/contrib/starrocks-python-client/starrocks/sqlalchemy)。

## 环境

- Python 3.x
- mysqlclient (pip install mysqlclient)
- [Apache Superset](https://superset.apache.org)

注意: 如果未安装 `mysqlclient`，将抛出异常:

```plain text
No module named 'MySQLdb'
```

## 安装

由于 `dialect` 未贡献给 `SQLAlchemy`，因此需要从源代码安装。

如果您使用 Docker 安装 `superset`，需要以 `root` 用户安装 `sqlalchemy-starrocks`。

从[源代码安装](https://github.com/StarRocks/starrocks/tree/main/contrib/starrocks-python-client/starrocks)

```shell
pip install .
```

卸载

```shell
pip uninstall sqlalchemy-starrocks
```

## 使用

要使用 SQLAlchemy 连接到 StarRocks，可以使用以下 URL 模式:

```shell
starrocks://<username>:<password>@<host>:<port>/<database>[?charset=utf8]
```

## 基本示例

### Sqlalchemy 示例

建议使用 python 3.x 连接到 StarRocks 数据库，例如:

```python
from sqlalchemy import create_engine
import pandas as pd
conn = create_engine('starrocks://root:@x.x.x.x:9030/superset_db?charset=utf8')
sql = """select * from xxx"""
df = pd.read_sql(sql, conn)
```

### Superset 示例

在 superset 中，使用 `Other` 数据库，并将 url 设置为:

```shell
starrocks://root:@x.x.x.x:9030/superset_db?charset=utf8
```