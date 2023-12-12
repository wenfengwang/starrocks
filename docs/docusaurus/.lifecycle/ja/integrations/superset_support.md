---
displayed_sidebar: "Japanese"
---

# Superset サポート

[Apache Superset](https://superset.apache.org) は、最新のデータ探索および可視化プラットフォームです。[SQLAlchemy](https://github.com/StarRocks/starrocks/tree/main/contrib/starrocks-python-client/starrocks) を使用してデータをクエリします。

[Mysql ディレクトリ](https://superset.apache.org/docs/databases/mysql)を使用できますが、`largeint`はサポートされていません。そのため、[StarRocks ディレクトリ](https://github.com/StarRocks/starrocks/tree/main/contrib/starrocks-python-client/starrocks/sqlalchemy)を開発しました。

## 環境

- Python 3.x
- mysqlclient (pip install mysqlclient)
- [Apache Superset](https://superset.apache.org)

注意: `mysqlclient`がインストールされていない場合、次のように例外が発生します:

```plain text
No module named 'MySQLdb'
```

## インストール

`dialect`は`SQLAlchemy`に貢献しないため、ソースコードからインストールする必要があります。

Dockerで`superset`をインストールする場合は、`root`で`sqlalchemy-starrocks`をインストールしてください。

[ソースコードからインストール](https://github.com/StarRocks/starrocks/tree/main/contrib/starrocks-python-client/starrocks)

```shell
pip install .
```

アンインストール

```shell
pip uninstall sqlalchemy-starrocks
```

## 使用法

StarRocksにSQLAlchemyで接続するには、次のURLパターンを使用できます:

```shell
starrocks://<username>:<password>@<host>:<port>/<database>[?charset=utf8]
```

## 基本例

### Sqlalchemy の例

StarRocksデータベースに接続するには、python 3.xを使用することをお勧めします。例:

```python
from sqlalchemy import create_engine
import pandas as pd
conn = create_engine('starrocks://root:@x.x.x.x:9030/superset_db?charset=utf8')
sql = """select * from xxx"""
df = pd.read_sql(sql, conn)
```

### Superset の例

Supersetでは、`Other`データベースを使用し、次のようにURLを設定します:

```shell
starrocks://root:@x.x.x.x:9030/superset_db?charset=utf8
```