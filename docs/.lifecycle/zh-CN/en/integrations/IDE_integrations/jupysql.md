---
displayed_sidebar: "Chinese"
---

# Jupyter

本指南描述了如何将你的StarRocks集群与[Jupyter](https://jupyter.org/)集成，这是最新的基于web的交互式开发环境，用于笔记本、代码和数据。

所有这些都是通过[JupySQL](https://jupysql.ploomber.io/)实现的，它允许你通过%sql、%%sql和%sqlplot魔术在Jupyter中运行SQL并绘制大型数据集。

你可以在Jupyter上使用JupySQL来在StarRocks上运行查询。

一旦数据加载到集群中，你可以通过SQL绘图查询和可视化数据。

## 先决条件

开始之前，你必须在本地安装以下软件：

- [JupySQL](https://jupysql.ploomber.io/en/latest/quick-start.html): `pip install jupysql`
- Jupyterlab: `pip install jupyterlab`
- [SKlearn Evaluation](https://github.com/ploomber/sklearn-evaluation): `pip install sklearn-evaluation`
- Python
- pymysql: `pip install pymysql`

> **注意**
>
> 一旦满足上述要求，你可以通过调用`jupyterlab`来简单地打开Jupyter lab - 这将打开笔记本界面。
> 如果Jupyter lab已在笔记本中运行，你可以简单地运行下面的单元格来获取依赖项。

```python
# 安装所需软件包。
%pip install --quiet jupysql sklearn-evaluation pymysql
```

> **注意**
>
> 你可能需要重新启动内核以使用更新的软件包。

```python
import pandas as pd
from sklearn_evaluation import plot

# 导入JupySQL Jupyter扩展以创建SQL单元格。
%load_ext sql
%config SqlMagic.autocommit=False
```

**你需要确保StarRocks实例处于上线状态并且可以访问到下一阶段。**

> **注意**
>
> 你需要根据你尝试连接的实例类型（url、用户和密码）调整连接字符串。以下示例使用本地实例。

## 通过JupySQL连接到StarRocks

在这个例子中，使用了一个docker实例，并且这反映在连接字符串中的数据。

使用`root`用户连接到本地StarRocks实例，创建一个数据库，并检查是否可以从表中读取和写入数据。

```python
%sql mysql+pymysql://root:@localhost:9030
```

创建并使用该JupySQL数据库：

```python
%sql CREATE DATABASE jupysql;
```

```python
%sql USE jupysql;
```

创建表：

```python
%%sql
CREATE TABLE tbl(c1 int, c2 int) distributed by hash(c1) properties ("replication_num" = "1");
INSERT INTO tbl VALUES (1, 10), (2, 20), (3, 30);
SELECT * FROM tbl;
```

## 保存和加载查询

现在在你创建数据库之后，你可以将一些示例数据写入其中并查询它。

JupySQL允许你将查询拆分为多个单元格，简化了构建大型查询的过程。

你可以编写复杂的查询，保存它们，并在需要时执行它们，类似于SQL中的CTEs。

```python
# 这在下一个JupySQL发布中待定。
%%sql --save initialize-table --no-execute
CREATE TABLE tbl(c1 int, c2 int) distributed by hash(c1) properties ("replication_num" = "1");
INSERT INTO tbl VALUES (1, 1), (2, 2), (3, 3);
SELECT * FROM tbl;
```

> **注意**
>
> `--save`保存的是查询而不是数据。

注意我们正在使用`--with`，这将检索先前保存的查询，并在前面加上它们（使用CTEs）。然后，我们将查询保存在`track_fav`中。

## 直接在StarRocks上绘图

JupySQL默认提供了一些绘图选项，允许你直接在SQL中可视化数据。

你可以使用条形图来可视化在你新建的表中的数据：

```python
top_artist = %sql SELECT * FROM tbl
top_artist.bar()
```

现在你有了一个新的条形图，无需额外的代码。你可以通过JupySQL（由ploomber提供）直接从你的笔记本中运行SQL。这为数据科学家和工程师提供了许多围绕StarRocks的可能性。如果你遇到困难或需要任何支持，请通过Slack联系我们。