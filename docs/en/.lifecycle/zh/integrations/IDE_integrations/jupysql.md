---
displayed_sidebar: English
---

# Jupyter

本指南描述了如何将您的 StarRocks 集群与[Jupyter](https://jupyter.org/)集成，这是最新的基于 Web 的交互式开发环境，用于笔记本、代码和数据。

所有这些都是通过[JupySQL](https://jupysql.ploomber.io/)实现的，它允许您通过 %sql、%%sql 和 %sqlplot 魔术在 Jupyter 中运行 SQL 并绘制大型数据集。

您可以在 Jupyter 上使用 JupySQL 来运行 StarRocks 上的查询。

将数据加载到集群后，您可以通过 SQL 绘图对其进行查询和可视化。

## 先决条件

开始之前，您必须在本地安装以下软件：

- [JupySQL](https://jupysql.ploomber.io/en/latest/quick-start.html)：`pip install jupysql`
- Jupyterlab：`pip install jupyterlab`
- [SKlearn Evaluation](https://github.com/ploomber/sklearn-evaluation)：`pip install sklearn-evaluation`
- Python
- PyMySQL：`pip install pymysql`

> **注意**
>
> 满足上述要求后，只需调用 `jupyterlab` 即可打开 Jupyter lab - 这将打开笔记本界面。
> 如果 Jupyter lab 已在笔记本中运行，只需运行下面的单元格即可获取依赖项。

```python
# 安装所需的软件包。
%pip install --quiet jupysql sklearn-evaluation pymysql
```

> **注意**
>
> 您可能需要重新启动内核才能使用更新的软件包。

```python
import pandas as pd
from sklearn_evaluation import plot

# 导入 JupySQL Jupyter 扩展以创建 SQL 单元格。
%load_ext sql
%config SqlMagic.autocommit=False
```

**您需要确保您的 StarRocks 实例已启动并且可以在下一阶段访问。**

> **注意**
>
> 您需要根据您尝试连接的实例类型（url、user 和 password）调整连接字符串。下面的示例使用本地实例。

## 通过 JupySQL 连接 StarRocks

在此示例中，使用了 docker 实例，并且这在连接字符串中有所体现。

使用 `root` 用户连接到本地 StarRocks 实例，创建数据库，并检查数据是否真的可以读写到表中。

```python
%sql mysql+pymysql://root:@localhost:9030
```

创建并使用该 JupySQL 数据库：

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

现在，在创建数据库后，您可以将一些示例数据写入其中并进行查询。

JupySQL 允许您将查询分解为多个单元，从而简化构建大型查询的过程。

您可以编写复杂的查询，保存它们，并在需要时执行它们，其方式类似于 SQL 中的 CTE。

```python
# 这等待下一个 JupySQL 发布。
%%sql --save initialize-table --no-execute
CREATE TABLE tbl(c1 int, c2 int) distributed by hash(c1) properties ("replication_num" = "1");
INSERT INTO tbl VALUES (1, 1), (2, 2), (3, 3);
SELECT * FROM tbl;
```

> **注意**
>
> `--save` 存储查询，而不是数据。

请注意，我们正在使用 `--with;`，这将检索以前保存的查询，并在它们前面添加它们（使用 CTE）。然后，我们将查询保存在 `track_fav`。

## 直接在 StarRocks 上绘图

JupySQL 默认带有一些绘图，允许您直接在 SQL 中可视化数据。

您可以使用条形图来可视化新创建的表中的数据：

```python
top_artist = %sql SELECT * FROM tbl
top_artist.bar()
```

现在，您有了一个新的条形图，而无需任何额外的代码。您可以通过 JupySQL（由 ploomber 提供）直接从笔记本运行 SQL。这为数据科学家和工程师在 StarRocks 上提供了许多可能性。如果您遇到困难或需要任何支持，请通过 Slack 联系我们。
