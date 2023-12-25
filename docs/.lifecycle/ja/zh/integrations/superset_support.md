---
displayed_sidebar: Chinese
---

# Superset のサポート

[Apache Superset](https://superset.apache.org) は、現代のデータ探索および可視化プラットフォームです。[SQLAlchemy](https://github.com/StarRocks/starrocks/tree/main/contrib/starrocks-python-client/starrocks) を使用してデータをクエリします。
[Mysql Dialect](https://superset.apache.org/docs/databases/mysql) を使用することも可能ですが、LARGEINT はサポートされていません。そのため、[StarRocks Dialect](https://github.com/StarRocks/starrocks/tree/main/contrib/starrocks-python-client/starrocks/sqlalchemy) を開発しました。

## 環境準備

- Python 3.x
- mysqlclient (`pip install mysqlclient` でインストール)
- [Apache Superset](https://superset.apache.org)

注意: `mysqlclient` がインストールされていない場合、例外が発生します: No module named 'MySQLdb'。

## インストール

`dialect` はまだ SQLAlchemy コミュニティに貢献されていないため、ソースコードを使用してインストールする必要があります。

Docker を使用して Superset をインストールする場合は、`root` ユーザーで `sqlalchemy-starrocks` をインストールする必要があります。

以下の[ソースコード](https://github.com/StarRocks/starrocks/tree/main/contrib/starrocks-python-client/starrocks)を使用してインストールします。

```shell
pip install .
```

アンインストール

```shell
pip uninstall sqlalchemy-starrocks
```

## 使用方法

SQLAlchemy を介して StarRocks に接続するには、以下の接続文字列を使用します:

```shell
starrocks://<username>:<password>@<host>:<port>/<database>[?charset=utf8]
```

## 例

### Sqlalchemy の例

Python 3.x を使用して StarRocks データベースに接続することをお勧めします。例えば：

```shell
from sqlalchemy import create_engine
import pandas as pd
conn = create_engine('starrocks://root:@x.x.x.x:9030/superset_db?charset=utf8')
sql = """select * from xxx"""
df = pd.read_sql(sql, conn)
```

### Superset の例

Superset を使用する際は、`Other` データベース接続を使用し、URL を以下のように設定します:

```shell
starrocks://root:@x.x.x.x:9030/superset_db?charset=utf8
```
