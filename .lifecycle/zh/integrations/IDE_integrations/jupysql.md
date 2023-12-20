---
displayed_sidebar: English
---

# 朱庇特

本指南描述了如何将StarRocks集群与[Jupyter](https://jupyter.org/)集成，Jupyter是最新的基于Web的交互式开发环境，适用于笔记本、代码和数据。

这一切都得益于 [JupySQL](https://jupysql.ploomber.io/)，它允许您通过 %sql、%%sql 和 %sqlplot 魔法命令在 Jupyter 中运行 SQL 并绘制大型数据集。

您可以在 Jupyter 上使用 JupySQL 对 StarRocks 进行查询操作。

数据加载到集群后，您可以通过 SQL 绘图来查询和可视化数据。

## 先决条件

在开始之前，您必须在本地安装以下软件：

- [JupySQL](https://jupysql.ploomber.io/en/latest/quick-start.html)：`pip install jupysql`
- JupyterLab：pip install jupyterlab
- [SKlearn Evaluation](https://github.com/ploomber/sklearn-evaluation)：`pip install sklearn-evaluation`
- Python
- PyMySQL：pip install pymysql

> **注意**
> 一旦满足上述要求，您可以通过运行 `jupyterlab` 命令来打开 Jupyter Lab，这将打开笔记本界面。如果 Jupyter Lab 已在运行中的笔记本中，则可以直接运行下面的单元格来安装依赖项。

```python
# Install required packages.
%pip install --quiet jupysql sklearn-evaluation pymysql
```

> **注意**
> 您可能需要**重启内核**来使用更新后的包。

```python
import pandas as pd
from sklearn_evaluation import plot

# Import JupySQL Jupyter extension to create SQL cells.
%load_ext sql
%config SqlMagic.autocommit=False
```

**您需要确保 StarRocks 实例已启动，并且在接下来的步骤中可以访问。**

> **注意**
> 您需要根据您尝试连接的实例类型调整连接字符串（包括 url、用户名和密码）。下面的示例使用的是本地实例。

## 通过 JupySQL 连接到 StarRocks

在这个示例中，使用的是 docker 实例，这反映在连接字符串中。

使用 root 用户连接到本地 StarRocks 实例，创建数据库，并验证确实可以从表中读取和写入数据。

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

创建数据库后，您可以向其中写入一些样本数据并查询。

JupySQL 允许您将查询分割到多个单元格中，简化了构建大型查询的过程。

您可以编写复杂的查询，保存并在需要时执行它们，类似于 SQL 中的公用表表达式（CTE）。

```python
# This is pending for the next JupySQL release.
%%sql --save initialize-table --no-execute
CREATE TABLE tbl(c1 int, c2 int) distributed by hash(c1) properties ("replication_num" = "1");
INSERT INTO tbl VALUES (1, 1), (2, 2), (3, 3);
SELECT * FROM tbl;
```

> **注意**
> `--save` 存储的是查询，而不是数据。

请注意，我们使用了 --with;，这将检索并预先插入之前保存的查询（使用 CTE）。然后，我们将查询保存在 track_fav 中。

## 直接在 StarRocks 上进行绘图

JupySQL 默认提供了一些图表，让您可以直接在 SQL 中将数据可视化。

您可以使用条形图来展示新创建的表中的数据：

```python
top_artist = %sql SELECT * FROM tbl
top_artist.bar()
```

现在，您有了一个新的条形图，而无需编写任何额外的代码。您可以通过 JupySQL（由 Ploomber 提供）直接在笔记本中运行 SQL。这为数据科学家和工程师提供了围绕 StarRocks 进行数据探索的更多可能性。如果您遇到任何困难或需要支持，请通过 Slack 联系我们。
