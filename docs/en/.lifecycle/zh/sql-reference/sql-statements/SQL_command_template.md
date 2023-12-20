---
displayed_sidebar: English
---

# SQL语句模板

> 本模板以`ADMIN SET REPLICA STATUS`为例，说明编写SQL命令主题的要求。

    > 运行文本中的命令和关键字需大写。例如，“SELECT语句用于查询满足特定条件的记录。”，“你可以使用GROUP BY对该列数据进行分组。”，“LIMIT关键字指定了可以返回的记录的最大数量。”

    > 如果需要在运行文本中引用参数或参数值，请将其用两个反引号(``)括起来，例如，`cachesize`。

## ADMIN SET REPLICA STATUS

> 主题标题。使用英文命令名作为主题标题。命令名中所有字母需大写。确保拼写正确。

### 描述

指定tablet的副本状态。此命令用于手动设置tablet的副本状态为`bad`或`ok`。

> 此命令的作用。你可以添加相关描述或使用说明。

### 语法

```SQL
ADMIN SET REPLICA STATUS

PROPERTIES ("key" = "value", ...);
```

> 此命令的语法。将语法放在代码块中。确保语法符合编码规范。

    > 使用适当的换行和缩进。

    > 代码中不要使用中文字符，如中文分号或逗号。

    > 在SQL命令中关键字需大写。例如：

```SQL
SELECT ta.x, count(ta.y) AS y, sum(tb.z) AS z

FROM (

    SELECT a AS x, b AS y

    FROM t) ta

    JOIN tb

        ON ta.x = tb.x

WHERE tb.a > 10

GROUP BY ta.x

ORDER BY ta.x, z

LIMIT 10
```

### 参数

`PROPERTIES`：每个属性必须是键值对。支持的属性包括：

- `tablet_id`：tablet的ID。此参数是必需的。
- `backend_id`：tablet的BE ID。此参数是必需的。
- `status`：副本的状态。此参数是必需的。有效值为`bad`和`ok`。`ok`表示系统将自动修复tablet的副本。如果副本状态被设置为`bad`，副本可能会立即被删除。执行此操作时请谨慎。如果指定的tablet不存在或副本状态为`bad`，系统将忽略这些副本。

> 命令中参数的说明。

    > 优选的参数描述应包括参数的含义、值格式、值范围、是否必需以及必要时的其他备注。

    > 你可以使用无序列表来组织参数描述。如果描述较复杂，可以使用表格来组织信息。表格可以包含以下列：参数名、值类型（可选）、示例值（可选）、参数描述。

### 使用说明（可选）

> 你可以添加一些使用该命令的注释或注意事项。

### 示例

示例1：将BE 10001上的tablet 10003的副本状态设置为`bad`。

```SQL
ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");
```

示例2：将BE 10001上的tablet 10003的副本状态设置为`ok`。

```SQL
ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "ok");
```

    > 提供使用此命令的示例，并解释每个示例的目的。

    > 你可以提供多个示例。

    > 如果需要在一个示例中描述多个场景，请在代码片段中为每个场景添加注释，以帮助用户快速区分它们。