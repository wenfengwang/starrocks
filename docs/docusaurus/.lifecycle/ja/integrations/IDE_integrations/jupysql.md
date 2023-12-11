---
displayed_sidebar: "Japanese"
---

# Jupyter

このガイドでは、[Jupyter](https://jupyter.org/)、最新のWebベースのインタラクティブなノートブック、コード、およびデータ開発環境と、StarRocksクラスターを統合する方法について説明します。

これはすべて、Jupyterを介して大規模データセットのSQLクエリとプロットを実行できるようにする[JupySQL](https://jupysql.ploomber.io/)によって可能になります。これは、%sql、%%sql、および%sqlplotのマジックを使用してJupyter上でSQLを実行し、データをプロットすることができます。

Jupyter上でJupySQLを使用してStarRocks上でクエリを実行できます。

データがクラスターにロードされると、SQLプロットを使ってクエリを実行し、視覚化することができます。

## 前提条件

開始する前に、ローカルに以下のソフトウェアをインストールしている必要があります:

- [JupySQL](https://jupysql.ploomber.io/en/latest/quick-start.html): `pip install jupysql`
- Jupyterlab: `pip install jupyterlab`
- [SKlearn Evaluation](https://github.com/ploomber/sklearn-evaluation): `pip install sklearn-evaluation`
- Python
- pymysql: `pip install pymysql`

> **注意**
>
> 上記の要件を満たしている場合、`jupyterlab`を呼び出すだけでJupyter labを開くことができます。これにより、ノートブックインターフェースが開きます。 Jupyter labがすでにノートブックで実行されている場合は、単にセルを実行するだけで依存関係を取得できます。

```python
# 必要なパッケージをインストールします。
%pip install --quiet jupysql sklearn-evaluation pymysql
```

> **注意**
>
> 更新されたパッケージを使用するには、カーネルを再起動する必要がある場合があります。

```python
import pandas as pd
from sklearn_evaluation import plot

# JupySQL Jupyter拡張機能をインポートしてSQLセルを作成します。
%load_ext sql
%config SqlMagic.autocommit=False
```

**次の段階でStarRocksインスタンスが起動し、アクセスできることを確認する必要があります。**

> **注意**
>
> 接続文字列を、接続しようとしているインスタンスタイプ（url、ユーザー、パスワード）に応じて調整する必要があります。以下の例では、ローカルインスタンスが使用されています。

## JupySQLを介したStarRocksへの接続

この例では、dockerインスタンスが使用され、それが接続文字列のデータを反映しています。

`root`ユーザーを使用してローカルのStarRocksインスタンスに接続し、データを読み取りおよび書き込みできるかどうかを確認します。

```python
%sql mysql+pymysql://root:@localhost:9030
```

次に、JupySQLデータベースを作成して使用します。

```python
%sql CREATE DATABASE jupysql;
```

```python
%sql USE jupysql;
```

テーブルを作成します。

```python
%%sql
CREATE TABLE tbl(c1 int, c2 int) distributed by hash(c1) properties ("replication_num" = "1");
INSERT INTO tbl VALUES (1, 10), (2, 20), (3, 30);
SELECT * FROM tbl;
```

## クエリの保存と読み込み

データベースを作成した後、サンプルデータを書き込み、クエリを実行できます。

JupySQLを使用すると、クエリを複数のセルに分割し、大規模なクエリを構築するプロセスを簡素化できます。

複雑なクエリを作成し、保存し、必要に応じて実行できます。これはSQLのCTEと同様の方法です。

```python
# これは次のJupySQLリリースを待っています。
%%sql --save initialize-table --no-execute
CREATE TABLE tbl(c1 int, c2 int) distributed by hash(c1) properties ("replication_num" = "1");
INSERT INTO tbl VALUES (1, 1), (2, 2), (3, 3);
SELECT * FROM tbl;
```

> **注意**
>
> `--save`はクエリを保存しますが、データは保存しません。

前に保存したクエリを取得し、それらを先頭に追加し（CTEを使用）、クエリを`track_fav`に保存します。

## StarRocks上で直接プロット

JupySQLには、いくつかのデフォルトのプロットが付属しており、SQLでデータを直接視覚化できます。

新しく作成したテーブルのデータを棒グラフで視覚化するために、次のコードを使用できます。

```python
top_artist = %sql SELECT * FROM tbl
top_artist.bar()
```

これで余分なコードを追加することなく新しい棒グラフが得られます。データサイエンティストやエンジニアには、JupySQL（by ploomber）を介してノートブックから直接SQLを実行する可能性がたくさん提供されます。行き詰まった場合やサポートが必要な場合は、Slackを介してお問い合わせください。