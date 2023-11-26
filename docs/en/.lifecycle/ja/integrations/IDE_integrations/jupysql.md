---
displayed_sidebar: "Japanese"
---

# Jupyter

このガイドでは、[Jupyter](https://jupyter.org/)という最新のウェブベースの対話型開発環境を使用して、StarRocksクラスタを統合する方法について説明します。Jupyterを使用することで、%sql、%%sql、および%sqlplotのマジックを介して、SQLを実行し、大規模なデータセットをJupyterでプロットすることができます。

Jupyterの上にJupySQLを使用して、StarRocks上でクエリを実行することができます。

データがクラスタにロードされると、SQLプロットを介してクエリを実行し、可視化することができます。

## 前提条件

始める前に、次のソフトウェアがローカルにインストールされている必要があります。

- [JupySQL](https://jupysql.ploomber.io/en/latest/quick-start.html): `pip install jupysql`
- Jupyterlab: `pip install jupyterlab`
- [SKlearn Evaluation](https://github.com/ploomber/sklearn-evaluation): `pip install sklearn-evaluation`
- Python
- pymysql: `pip install pymysql`

> **注意**
>
> 上記の要件を満たしている場合、`jupyterlab`を呼び出すだけでJupyter labを開くことができます。これにより、ノートブックインターフェースが開きます。
> もしJupyter labが既にノートブックで実行されている場合、以下のセルを実行することで依存関係を取得することができます。

```python
# 必要なパッケージをインストールします。
%pip install --quiet jupysql sklearn-evaluation pymysql
```

> **注意**
>
> 更新されたパッケージを使用するためには、カーネルを再起動する必要がある場合があります。

```python
import pandas as pd
from sklearn_evaluation import plot

# SQLセルを作成するためにJupySQL Jupyter拡張機能をインポートします。
%load_ext sql
%config SqlMagic.autocommit=False
```

**次のステージに進む前に、StarRocksインスタンスが起動していてアクセス可能であることを確認する必要があります。**

> **注意**
>
> 接続文字列は、接続しようとしているインスタンスのタイプ（URL、ユーザー、パスワード）に応じて調整する必要があります。以下の例では、ローカルインスタンスを使用しています。

## JupySQLを介してStarRocksに接続する

この例では、Dockerインスタンスが使用され、それが接続文字列に反映されています。

`root`ユーザーを使用して、ローカルのStarRocksインスタンスに接続し、データが実際にテーブルから読み取りおよび書き込みできることを確認します。

```python
%sql mysql+pymysql://root:@localhost:9030
```

JupySQLデータベースを作成して使用します。

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

データベースを作成した後、いくつかのサンプルデータを書き込んでクエリを実行することができます。

JupySQLを使用すると、クエリを複数のセルに分割して、大規模なクエリの作成プロセスを簡素化することができます。

CTE（SQLの共通テーブル式）と同様の方法で、複雑なクエリを書き込んで保存し、必要な時に実行することができます。

```python
# これは次のJupySQLリリースまで保留されています。
%%sql --save initialize-table --no-execute
CREATE TABLE tbl(c1 int, c2 int) distributed by hash(c1) properties ("replication_num" = "1");
INSERT INTO tbl VALUES (1, 1), (2, 2), (3, 3);
SELECT * FROM tbl;
```

> **注意**
>
> `--save`はクエリを保存しますが、データは保存しません。

`--with;`を使用して、以前保存したクエリを取得し、それらを（CTEを使用して）前置します。そして、クエリを`track_fav`に保存します。

## StarRocks上で直接プロットする

JupySQLには、デフォルトでいくつかのプロットが付属しており、データをSQLで直接可視化することができます。

新しく作成したテーブルのデータをバープロットで可視化することができます。

```python
top_artist = %sql SELECT * FROM tbl
top_artist.bar()
```

これで、追加のコードなしで新しいバープロットができました。JupySQL（by ploomber）を介してノートブックから直接SQLを実行することができます。これにより、データサイエンティストやエンジニアにとって、StarRocksの周りで多くの可能性が生まれます。もし困ったことがあったり、サポートが必要な場合は、Slackでお問い合わせください。
