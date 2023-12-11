---
displayed_sidebar: "Japanese"
---

# Superset サポート

[Apache Superset](https://superset.apache.org) は、現代のデータ探索および可視化プラットフォームです。[SQLAlchemy](https://github.com/StarRocks/starrocks/tree/main/contrib/starrocks-python-client/starrocks) を使用してデータをクエリします。

[Mysql ディレクトリ](https://superset.apache.org/docs/databases/mysql) を使用できますが、`largeint` はサポートされていません。そのため、[StarRocks ディレクトリ](https://github.com/StarRocks/starrocks/tree/main/contrib/starrocks-python-client/starrocks/sqlalchemy) を開発しました。

## 環境

- Python 3.x
- mysqlclient (pip install mysqlclient)
- [Apache Superset](https://superset.apache.org)

注意：`mysqlclient` がインストールされていない場合は、次のような例外がスローされます：

```plain text
No module named 'MySQLdb'
```

## インストール

`dialect` が `SQLAlchemy` に寄与しないため、ソースコードからインストールする必要があります。

Docker で `superset` をインストールする場合は、`root` 権限で `sqlalchemy-starrocks` をインストールします。

[ソースコードからインストール](https://github.com/StarRocks/starrocks/tree/main/contrib/starrocks-python-client/starrocks)

```shell
pip install .
```

アンインストール

```shell
pip uninstall sqlalchemy-starrocks
```

## 使用方法

SQLAlchemy で StarRocks に接続するためには、次の URL パターンを使用できます：

```shell
starrocks://<username>:<password>@<host>:<port>/<database>[?charset=utf8]
```

## 基本的な例

### Sqlalchemy の例

Python 3.x を使用して StarRocks データベースに接続することを推奨します。例：

```python
from sqlalchemy import create_engine
import pandas as pd
conn = create_engine('starrocks://root:@x.x.x.x:9030/superset_db?charset=utf8')
sql = """select * from xxx"""
df = pd.read_sql(sql, conn)
```

### Superset の例

superset では、`Other` データベースを使用し、url を次のように設定します：

```shell
starrocks://root:@x.x.x.x:9030/superset_db?charset=utf8
```