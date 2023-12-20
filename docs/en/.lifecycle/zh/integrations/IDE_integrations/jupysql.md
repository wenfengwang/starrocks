---
displayed_sidebar: English
---

# Jupyter

本指南描述了如何将您的 StarRocks 集群与 [Jupyter](https://jupyter.org/) 集成，Jupyter 是最新的基于 Web 的交互式开发环境，用于笔记本、代码和数据。

所有这一切都是通过 [JupySQL](https://jupysql.ploomber.io/) 实现的，它允许您在 Jupyter 中通过 %sql、%%sql 和 %sqlplot 魔法命令运行 SQL 并绘制大型数据集。

您可以在 Jupyter 之上使用 JupySQL 在 StarRocks 上运行查询。

一旦数据被加载到集群中，您就可以通过 SQL 绘图来进行查询和可视化。

## 先决条件

在开始之前，您必须在本地安装以下软件：

- [JupySQL](https://jupysql.ploomber.io/en/latest/quick-start.html)：`pip install jupysql`
- JupyterLab：`pip install jupyterlab`
- [SKlearn Evaluation](https://github.com/ploomber/sklearn-evaluation)：`pip install sklearn-evaluation`
- Python
- PyMySQL：`pip install pymysql`

> **注意**
> 一旦满足上述要求，您可以通过调用 `jupyter lab` 来打开 JupyterLab - 这将打开笔记本界面。
如果 JupyterLab 已在笔记本中运行，您可以直接运行下面的单元格来获取依赖项。

```python
# Install required packages.
%pip install --quiet jupysql sklearn-evaluation pymysql
```

> **注意**
> 您可能需要重新启动内核以使用更新的包。

```python
import pandas as pd
from sklearn_evaluation import plot

# Import JupySQL Jupyter extension to create SQL cells.
%load_ext sql
%config SqlMagic.autocommit=False
```

**您需要确保您的 StarRocks 实例已启动并且可以在下一阶段访问。**

> **注意**
> 您需要根据您尝试连接的实例类型（URL、用户和密码）调整连接字符串。下面的示例使用本地实例。

## 通过 JupySQL 连接到 StarRocks

在此示例中，使用了 Docker 实例，这反映在连接字符串中。

使用 `root` 用户连接到本地 StarRocks 实例，创建数据库，并检查确保可以从表中读取和写入数据。

```python
%sql mysql+pymysql://root:@localhost:9030
```

创建并使用 JupySQL 数据库：

```python
%sql CREATE DATABASE jupysql;
```

```python
%sql USE jupysql;
```

创建一个表：

```python
%%sql
CREATE TABLE tbl(c1 int, c2 int) distributed by hash(c1) properties ("replication_num" = "1");
INSERT INTO tbl VALUES (1, 10), (2, 20), (3, 30);
SELECT * FROM tbl;
```

## 保存和加载查询

现在，在创建数据库之后，您可以向其中写入一些示例数据并进行查询。

JupySQL 允许您将查询分解到多个单元格中，简化了构建大型查询的过程。

您可以编写复杂的查询，保存它们，并在需要时执行，这与 SQL 中的 CTEs 类似。

```python
# This is pending for the next JupySQL release.
%%sql --save initialize-table --no-execute
CREATE TABLE tbl(c1 int, c2 int) distributed by hash(c1) properties ("replication_num" = "1");
INSERT INTO tbl VALUES (1, 1), (2, 2), (3, 3);
SELECT * FROM tbl;
```

> **注意**
> `--save` 用于存储查询，而不是数据。

请注意，我们使用了 `--with;`，这将检索之前保存的查询，并将它们作为公共表表达式（CTEs）添加到前面。然后，我们将查询保存在 `track_fav` 中。

## 直接在 StarRocks 上绘图

JupySQL 默认提供了一些图表，允许您直接在 SQL 中可视化数据。

您可以使用条形图来可视化新创建的表中的数据：

```python
top_artist = %sql SELECT * FROM tbl
top_artist.bar()
```

现在您有了一个新的条形图，无需任何额外的代码。您可以通过 JupySQL（由 ploomber 提供）直接从笔记本中运行 SQL。这为数据科学家和工程师围绕 StarRocks 提供了更多的可能性。如果您遇到困难或需要任何支持，请通过 Slack 与我们联系。