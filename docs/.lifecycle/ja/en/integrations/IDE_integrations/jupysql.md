---
displayed_sidebar: English
---

# Jupyter

このガイドでは、StarRocks クラスターをノートブック、コード、およびデータ用の最新の Web ベースの対話型開発環境である [Jupyter](https://jupyter.org/) と統合する方法について説明します。

これはすべて、%sql、%%sql、および %sqlplot マジックを介して Jupyter で SQL を実行し、大規模なデータセットをプロットできる [JupySQL](https://jupysql.ploomber.io/) を通じて可能になります。

Jupyter 上で JupySQL を使用して、StarRocks 上でクエリを実行することができます。

データがクラスターにロードされたら、SQL プロットを使用してクエリを実行し、視覚化することができます。

## 前提条件

開始する前に、以下のソフトウェアをローカルにインストールしておく必要があります：

- [JupySQL](https://jupysql.ploomber.io/en/latest/quick-start.html)：`pip install jupysql`
- JupyterLab：`pip install jupyterlab`
- [SKlearn Evaluation](https://github.com/ploomber/sklearn-evaluation)：`pip install sklearn-evaluation`
- Python
- PyMySQL：`pip install pymysql`

> **注記**
>
> 上記の要件が満たされていれば、`jupyterlab` を実行するだけで Jupyter Lab を開くことができます。これにより、ノートブックインターフェイスが開きます。
> Jupyter Lab がノートブックで既に実行されている場合は、下のセルを実行するだけで依存関係を取得できます。

```python
# 必要なパッケージをインストールします。
%pip install --quiet jupysql sklearn-evaluation pymysql
```

> **注記**
>
> 更新されたパッケージを使用するためには、カーネルを再起動する必要があるかもしれません。

```python
import pandas as pd
from sklearn_evaluation import plot

# SQL セルを作成するための JupySQL Jupyter 拡張機能をインポートします。
%load_ext sql
%config SqlMagic.autocommit=False
```

**StarRocks インスタンスが稼働しており、次のステージに到達可能であることを確認する必要があります。**

> **注記**
>
> 接続しようとしているインスタンスタイプ（URL、ユーザー、パスワード）に応じて接続文字列を調整する必要があります。以下の例では、ローカルインスタンスを使用しています。

## JupySQL 経由で StarRocks に接続する

この例では、Docker インスタンスが使用され、そのデータが接続文字列に反映されています。

`root` ユーザーを使用してローカルの StarRocks インスタンスに接続し、データベースを作成し、実際にテーブルからデータを読み書きできることを確認します。

```python
%sql mysql+pymysql://root:@localhost:9030
```

JupySQL データベースを作成して使用します：

```python
%sql CREATE DATABASE jupysql;
```

```python
%sql USE jupysql;
```

テーブルを作成します：

```python
%%sql
CREATE TABLE tbl(c1 int, c2 int) distributed by hash(c1) properties ("replication_num" = "1");
INSERT INTO tbl VALUES (1, 10), (2, 20), (3, 30);
SELECT * FROM tbl;
```

## クエリの保存と読み込み

データベースを作成した後、サンプルデータを書き込んでクエリを実行できます。

JupySQL を使用すると、クエリを複数のセルに分割して、大規模なクエリの構築プロセスを簡素化できます。

複雑なクエリを記述して保存し、必要に応じて実行することができます。これは SQL の CTE と同様の方法です。

```python
# これは次の JupySQL リリースまで保留されています。
%%sql --save initialize-table --no-execute
CREATE TABLE tbl(c1 int, c2 int) distributed by hash(c1) properties ("replication_num" = "1");
INSERT INTO tbl VALUES (1, 1), (2, 2), (3, 3);
SELECT * FROM tbl;
```

> **注記**
>
> `--save` はデータではなくクエリを保存します。

`--with;` を使用すると、以前に保存されたクエリを取得し、それらを先頭に追加します（CTE を使用して）。その後、クエリを `track_fav` に保存します。

## StarRocks で直接プロットする

JupySQL にはデフォルトでいくつかのプロットが付属しており、SQL でデータを直接視覚化できます。

新しく作成したテーブルのデータを視覚化するために棒グラフを使用できます：

```python
top_artist = %sql SELECT * FROM tbl
top_artist.bar()
```

これで、追加コードなしで新しい棒グラフができました。JupySQL（ploomber を使用）を介してノートブックから直接 SQL を実行することができます。これにより、データサイエンティストやエンジニアにとって StarRocks に関する多くの可能性が広がります。行き詰まった場合やサポートが必要な場合は、Slack 経由でお問い合わせください。
