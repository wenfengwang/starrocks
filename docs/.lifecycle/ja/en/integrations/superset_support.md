---
displayed_sidebar: English
---

# Superset サポート

[Apache Superset](https://superset.apache.org) は、最新のデータ探索および可視化プラットフォームです。[SQLAlchemy](https://github.com/StarRocks/starrocks/tree/main/contrib/starrocks-python-client/starrocks) を使用してデータをクエリします。

[Mysql Dialect](https://superset.apache.org/docs/databases/mysql) も使用できますが、`largeint` はサポートされていません。そこで、[StarRocks Dialect](https://github.com/StarRocks/starrocks/tree/main/contrib/starrocks-python-client/starrocks/sqlalchemy) を開発しました。

## 環境

- Python 3.x
- mysqlclient (`pip install mysqlclient` でインストール)
- [Apache Superset](https://superset.apache.org)

注意: `mysqlclient` がインストールされていない場合、例外がスローされます：

```plain text
No module named 'MySQLdb'
```

## インストール

`dialect` は `SQLAlchemy` には含まれていないため、ソースコードからインストールする必要があります。

Docker を使用して `superset` をインストールする場合は、`root` 権限で `sqlalchemy-starrocks` をインストールしてください。

ソースコードからのインストール方法は以下の通りです：

```shell
pip install .
```

アンインストール方法：

```shell
pip uninstall sqlalchemy-starrocks
```

## 使用方法

SQLAlchemy を使用して StarRocks に接続するには、以下の URL パターンを使用できます：

```shell
starrocks://<username>:<password>@<host>:<port>/<database>[?charset=utf8]
```

## 基本的な例

### SQLAlchemy の例

StarRocks データベースに接続するには、Python 3.x の使用を推奨します。例えば：

```python
from sqlalchemy import create_engine
import pandas as pd
conn = create_engine('starrocks://root:@x.x.x.x:9030/superset_db?charset=utf8')
sql = """select * from xxx"""
df = pd.read_sql(sql, conn)
```

### Superset の例

Superset では、`Other` データベースを使用し、URL を以下のように設定します：

```shell
starrocks://root:@x.x.x.x:9030/superset_db?charset=utf8
```
