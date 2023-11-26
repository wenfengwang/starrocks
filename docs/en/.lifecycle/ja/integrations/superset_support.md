---
displayed_sidebar: "Japanese"
---

# Superset サポート

[Apache Superset](https://superset.apache.org)は、モダンなデータ探索および可視化プラットフォームです。[SQLAlchemy](https://github.com/StarRocks/starrocks/tree/main/contrib/starrocks-python-client/starrocks)を使用してデータをクエリします。

[Mysql Dialect](https://superset.apache.org/docs/databases/mysql)を使用することもできますが、`largeint`をサポートしていません。そのため、[StarRocks Dialect](https://github.com/StarRocks/starrocks/tree/main/contrib/starrocks-python-client/starrocks/sqlalchemy)を開発しました。

## 環境

- Python 3.x
- mysqlclient (pip install mysqlclient)
- [Apache Superset](https://superset.apache.org)

注意: `mysqlclient`がインストールされていない場合、次のエラーが発生します:

```plain text
No module named 'MySQLdb'
```

## インストール

`dialect`は`SQLAlchemy`に貢献しないため、ソースコードからインストールする必要があります。

Dockerで`superset`をインストールする場合は、`root`で`sqlalchemy-starrocks`をインストールします。

[ソースコードからインストール](https://github.com/StarRocks/starrocks/tree/main/contrib/starrocks-python-client/starrocks)

```shell
pip install .
```

アンインストール

```shell
pip uninstall sqlalchemy-starrocks
```

## 使用方法

SQLAlchemyを使用してStarRocksに接続するためには、次のURLパターンを使用できます:

```shell
starrocks://<username>:<password>@<host>:<port>/<database>[?charset=utf8]
```

## 基本的な例

### Sqlalchemyの例

StarRocksデータベースに接続するためには、Python 3.xを使用することをお勧めします。例:

```python
from sqlalchemy import create_engine
import pandas as pd
conn = create_engine('starrocks://root:@x.x.x.x:9030/superset_db?charset=utf8')
sql = """select * from xxx"""
df = pd.read_sql(sql, conn)
```

### Supersetの例

Supersetでは、`Other`データベースを使用し、URLを次のように設定します:

```shell
starrocks://root:@x.x.x.x:9030/superset_db?charset=utf8
```
