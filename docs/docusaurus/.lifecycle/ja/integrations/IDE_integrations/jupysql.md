---
displayed_sidebar: "Japanese"
---

# Jupyter

このガイドでは、最新のWebベースのインタラクティブ開発環境である[Jupyter](https://jupyter.org/)と、StarRocksクラスターを統合する方法について説明します。これにより、ノートブック、コード、データを使用して、SQLを実行し、大規模なデータセットをプロットすることが可能となります。

これはすべて、[JupySQL](https://jupysql.ploomber.io/)によって実現されており、%sql、%%sql、%sqlplotのマジックを介してJupyterでSQLを実行し、大規模なデータセットをプロットすることができます。

Jupyter上でJupySQLを使用することで、StarRocks上でクエリを実行し、可視化することができます。

## 前提条件

開始する前に、以下のソフトウェアをローカルにインストールしている必要があります：

- [JupySQL](https://jupysql.ploomber.io/en/latest/quick-start.html): `pip install jupysql`
- Jupyterlab: `pip install jupyterlab`
- [SKlearn Evaluation](https://github.com/ploomber/sklearn-evaluation): `pip install sklearn-evaluation`
- Python
- pymysql: `pip install pymysql`

> **注意**
>
> 上記の要件を満たしている場合、単純に`jupyterlab`を呼び出すことでJupyter labを開くことができます。これは、ノートブックインターフェースを開きます。Jupyter labが既にノートブックで実行中の場合は、次のセルを実行するだけで依存関係を取得できます。

```python
# 必要なパッケージをインストールします。
%pip install --quiet jupysql sklearn-evaluation pymysql
```

> **注意**
>
> アップデートされたパッケージを使用するには、カーネルを再起動する必要がある場合があります。

```python
import pandas as pd
from sklearn_evaluation import plot

# JupySQL Jupyter拡張機能をインポートして、SQLセルを作成します。
%load_ext sql
%config SqlMagic.autocommit=False
```

**次のステップでStarRocksインスタンスが起動してアクセス可能であることを確認する必要があります。**

> **注意**
>
> 接続文字列を、接続しようとしているインスタンスのタイプ（URL、ユーザー、パスワード）に応じて調整する必要があります。以下の例では、ローカルインスタンスを使用しています。

## JupySQLを使用してStarRocksに接続する

この例では、Dockerインスタンスを使用しており、それが接続文字列に反映されています。

`root`ユーザーを使用してローカルのStarRocksインスタンスに接続し、データベースを作成し、データが実際にテーブルから読み取れるかどうかを確認します。

```python
%sql mysql+pymysql://root:@localhost:9030
```

次に、そのJupySQLデータベースを作成し、使用します。

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

データベースを作成した後、いくつかのサンプルデータを入力し、クエリを実行できます。

JupySQLを使用すると、クエリを複数のセルに分割して、大規模なクエリを構築するプロセスを簡略化することができます。

複雑なクエリを記述し、保存し、必要なときに実行できるようにすることができ、これはSQLのCTEと同様の方法で行います

```python
# これは次のJupySQLリリースまで保留です。
%%sql --save initialize-table --no-execute
CREATE TABLE tbl(c1 int, c2 int) distributed by hash(c1) properties ("replication_num" = "1");
INSERT INTO tbl VALUES (1, 1), (2, 2), (3, 3);
SELECT * FROM tbl;
```

> **注意**
>
> `--save`はクエリを保存し、データを保存するのではありません。

CTEを使用して以前に保存されたクエリを取得し、クエリを`track_fav`に保存します。

## StarRocksで直接プロット

JupySQLにはデフォルトでいくつかのプロットが付属しており、SQLでデータを直接可視化することができます。

新しく作成されたテーブルのデータをバープロットで視覚化するために使用できます。

```python
top_artist = %sql SELECT * FROM tbl
top_artist.bar()
```

こうして、追加のコードを書かずに新しいバープロットが作成されます。PloomberによるJupySQLを使用すると、データサイエンティストやエンジニアにとって、StarRocksのデータを直接ノートブックからSQLで実行できるようになります。もし問題が発生した場合やサポートが必要な場合は、Slackを通じてお問い合わせください。